+++
date = '2024-11-27T14:50:07+02:00'
draft = false
title = 'Serverless Command Pattern'
tags = ['serverless', 'aws', 'lambda', 'command-pattern', 'design-pattern', 'architecture', 'faas', 'microservices']
+++

## Introduction
There are many ways to organize your serverless functions.
Technically, their name is self-explanatory - they are functions.
Provider-agnostic, they are just pieces of code that do something upon execution and can either have a payload or not.
However, when you start building a slightly bigger system, you might want to organize them in a more structured way.
<!--more-->

### Idea Behind
When I first faced a `Command/Action` pattern in one of my projects, I was a bit confused.
I mean this is a well-known pattern in [software engineering](https://en.wikipedia.org/wiki/Design_Patterns), but I never thought about it in the context of serverless.
Pattern is well known in OOP world, but it can be applied in many other paradigms.
The idea is simple - you have a command that does something.
Some boilerplate code is shared between commands, so for a simple functions it might be an overkill (and overhead).
But as soon as you will have more than a few semantically related (but structurally different) events, you might want to consider this pattern.

### Command Pattern
Take a look at this code:
```python
from abc import ABC, abstractmethod
from typing import Dict

# Command interface
class Command(ABC):
    @abstractmethod
    def execute(self) -> bool:
        pass

# Concrete commands
class CreateUserCommand(Command):
    def __init__(self, event: dict):
        self.user_data = event
        
    def execute(self) -> bool:
        print(f"Creating user with data: {self.user_data}")
        # Some actual user creation logic here
        return True

class DeleteUserCommand(Command):
    def __init__(self, event: dict):
        self.user_id = event["user_id"]
        
    def execute(self) -> bool:
        print(f"Deleting user with ID: {self.user_id}")
        # Some actual user deletion logic here
        return True

# Event handler that uses commands
class EventHandler:
    def __init__(self):
        self.command_map: Dict[str, Command] = {}
    
    def register_command(self, event_type: str, command: Command):
        self.command_map[event_type] = command
    
    def handle_event(self, event_type: str) -> bool:
        if event_type not in self.command_map:
            raise ValueError(f"No command registered for event: {event_type}")
            
        command = self.command_map[event_type]
        return command.execute()

# Usage example
if __name__ == "__main__":
    # Create event handler
    handler = EventHandler()
    
    # Register commands for different events
    handler.register_command("user_created", CreateUserCommand({"name": "John", "email": "john@example.com"}))
    handler.register_command("user_deleted", DeleteUserCommand("user123"))
    
    # Handle events
    try:
        handler.handle_event("user_created")  # This will execute CreateUserCommand
        handler.handle_event("user_deleted")  # This will execute DeleteUserCommand
    except ValueError as e:
        print(f"Error: {e}")
```

Indeed, it looks like a basic OOP example.
It starts shining when you have more than a few commands and when you are using a single entry point to handle all events.

This is especially useful in serverless environments where you might have many functions that are triggered by different events.
Or in microservices orchestration where you want to have a single entry point to handle all events.

Additionally, it allows to abstract the actual logic from the event handling logic.

You can implement `UserCreatedCommand` and `UserDeletedCommand` in a way that is completely independent of the event handling logic.
For example, you can inject a test command in your tests to verify that the event handling logic works correctly.
Yes, there are many other ways to do this, many of them looks similar.

### Serverless Command Pattern

Same or similar approach is really useful when you use a defined interfaces for your services (_think `gRPC` or `Avro` etc._).
Commands can be easily swapped to test different implementations unless `consumer` of the command has not changed or have a strict contract.

It also a good approach in a state machine or workflow engine, where you can define a set of commands that can be executed in a specific order.

The pattern fits naturally because:

- Each state transition can be represented as a command
- Commands can be validated before execution
- Easy to track and monitor state changes
- Supports rollback/compensation actions

Here's a practical example combining both concepts:

```python
import json
from abc import ABC, abstractmethod
from typing import Dict, Any
from datetime import datetime
import boto3
from aws_lambda_powertools import Logger

logger = Logger()

# Base Command
class Command(ABC):
    @abstractmethod
    async def execute(self, context: Dict[str, Any]) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def validate(self, payload: Dict[str, Any]) -> bool:
        pass

# Concrete Commands
class ProcessOrderCommand(Command):
    def __init__(self):
        self.dynamodb = boto3.resource('dynamodb')
        self.sqs = boto3.client('sqs')
        
    def validate(self, payload: Dict[str, Any]) -> bool:
        required_fields = ['orderId', 'userId', 'amount']
        return all(field in payload for field in required_fields)
    
    async def execute(self, context: Dict[str, Any]) -> Dict[str, Any]:
        order_data = context['payload']
        
        # Store order in DynamoDB
        table = self.dynamodb.Table('Orders')
        order_data['status'] = 'PROCESSING'
        order_data['timestamp'] = datetime.utcnow().isoformat()
        
        await table.put_item(Item=order_data)
        
        # Send message to payment processing queue
        await self.sqs.send_message(
            QueueUrl='payment-processing-queue-url',
            MessageBody=json.dumps(order_data)
        )
        
        return {
            'statusCode': 200,
            'body': {'orderId': order_data['orderId'], 'status': 'PROCESSING'}
        }

class RefundOrderCommand(Command):
    def __init__(self):
        self.dynamodb = boto3.resource('dynamodb')
        self.sns = boto3.client('sns')
    
    def validate(self, payload: Dict[str, Any]) -> bool:
        required_fields = ['orderId', 'refundAmount']
        return all(field in payload for field in required_fields)
    
    async def execute(self, context: Dict[str, Any]) -> Dict[str, Any]:
        refund_data = context['payload']
        
        # Update order status
        table = self.dynamodb.Table('Orders')
        await table.update_item(
            Key={'orderId': refund_data['orderId']},
            UpdateExpression='SET status = :status, refundAmount = :amount',
            ExpressionAttributeValues={
                ':status': 'REFUNDED',
                ':amount': refund_data['refundAmount']
            }
        )
        
        # Notify relevant services
        await self.sns.publish(
            TopicArn='refund-notification-topic',
            Message=json.dumps(refund_data)
        )
        
        return {
            'statusCode': 200,
            'body': {'orderId': refund_data['orderId'], 'status': 'REFUNDED'}
        }

# Command Factory
class CommandFactory:
    _commands: Dict[str, Command] = {
        'PROCESS_ORDER': ProcessOrderCommand(),
        'REFUND_ORDER': RefundOrderCommand()
    }
    
    @classmethod
    def get_command(cls, command_type: str) -> Command:
        command = cls._commands.get(command_type)
        if not command:
            raise ValueError(f"Unknown command type: {command_type}")
        return command

# Lambda Handler
@logger.inject_lambda_context
def handler(event, context):
    try:
        # Extract command type and payload from event
        command_type = event['commandType']
        payload = event['payload']
        
        # Get appropriate command
        command = CommandFactory.get_command(command_type)
        
        # Validate payload
        if not command.validate(payload):
            return {
                'statusCode': 400,
                'body': 'Invalid payload for command'
            }
        
        # Execute command
        result = command.execute({'payload': payload, 'context': context})
        return result
    
    except Exception as e:
        logger.exception("Error processing command")
        return {
            'statusCode': 500,
            'body': str(e)
        }
```
I am using `AWS` as an example here, but you can easily adapt it to any other provider or even run it locally.
The rest is on your state machine or workflow engine to handle the commands in a specific order. Each command can provide a response that can be used by the next command in the chain.

That's it! I hope you find this pattern useful.