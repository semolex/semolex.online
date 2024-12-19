+++
date = '2024-12-19T16:45:13+02:00'
draft = false
title = 'Use Lambda Global Cache'
+++


## Introduction

One of the problems with serverless applications is the lack of shared state between invocations. This is especially true for AWS Lambda functions, which are stateless by design. However, there are situations where you might want to share data between invocations, such as caching frequently accessed data or storing configuration settings.
You can use Redis or something similar, but it adds complexity and cost to your architecture. Other options?
<!--more-->

### Global Cache
Technically this is not something you should rely on. But if you have a small amount of data that you want to share between invocations, you can use the global scope of the Lambda function. 
This state will be preserved between invocations as long as the container is not destroyed. 
Imagine you need to query some service for a list of items, and you know that this list will not change often. You can store this list in the global scope of the Lambda function and reuse it between invocations.
Same goes for client connections, configuration settings, or any other data that you want to share between invocations.

### Techniques
Here are a few techniques you can use to store data in the global scope of a Lambda function:

1. **Global/Module Variables**: You can define global variables at the top of your Lambda function code. These variables will be shared between invocations as long as the container is not destroyed.
2. **Class-Level Variables**: You can define class-level variables in a class and reuse them between invocations. These variables will be shared between invocations as long as the container is not destroyed.
3. **Explicit Caching**: You can use some hash table or dictionary (or whatever) to store data explicitly. This way, you can control the data that is stored and retrieved between invocations.
4. **Abstraction via LRUCache**: You can use an abstraction like `functools.lru_cache` in Python to cache function results. This way, you can cache the results of expensive function calls and reuse them between invocations.

### Example
#### Global Variable:

```python
# Global variable
client = None

def my_handler(event, context):
    global client

    if client is None:
        client = boto3.client('s3')

    # Use the client to interact with S3
    response = client.list_buckets()

    return response
```

In this example, we define a global variable `client` at the top of the Lambda function code. We check if the client is `None` and create a new client if it is. This way, we reuse the client between invocations and save time and resources on creating a new client for each invocation.

#### Class-Level Variable:

```python
class MyHandler:
    client = None

    def __init__(self):
        if MyHandler.client is None:
            MyHandler.client = boto3.client('s3')

    def handle_event(self, event):
        # Use the client to interact with S3
        response = MyHandler.client.list_buckets()

        return response
```

In this example, we define a class `MyHandler` with a class-level variable `client`. We check if the client is `None` in the constructor and create a new client if it is. This way, we reuse the client between invocations and save time and resources on creating a new client for each invocation.
It is just nicer way to organize your code instead of using global variables.

#### Explicit Caching:

```python
cache = {}

def my_handler(event, context):
    user = event['user']
    my_param = event['my_param']
    cache_key = f"{user}-{my_param}"
    if cache_key not in cache:
        # Fetch data from some service
        data = fetch_data()
        cache[cache_key] = data

    return cache[cache_key]
```

In this example, we define a dictionary `cache` at the top of the Lambda function code. We check if the data for the given key is in the cache and fetch it if it is not. This way, we reuse the data between invocations and save time and resources on fetching the data for each invocation.

#### Abstraction via LRUCache:

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fetch_data(user, my_param):
    # Fetch data from some service
    # ...
    return data
```

In this example, we define a function `fetch_data` that fetches data from some service. We use the `functools.lru_cache` decorator to cache the results of the function calls. This way, we reuse the data between invocations and save time and resources on fetching the data for each invocation.

#### Considerations
Be careful when using global variables or caching data in the global scope of a Lambda function.
For example, you can run into issues with concurrent invocations, or you might run out of memory if you store too much data in the global scope.
So utilise techniques like cache invalidation, size limits, or other strategies to manage the data that you store.
When using own caching - choose cache key wisely, so you do not expose sensitive data or leak data between invocations.
Do not rely on this technique for critical data or data that changes often. It is more suitable for configuration settings, static data, or other non-critical data that you want to share between invocations.

### Conclusion
While it might sound more like a bad hack - using "scope cache" works. 
Sometimes AWS will spin up a new container for your function even if you expect it to be "hot", and you will lose the data. But in many cases, it is a good way to save time and resources on creating new connections, fetching data, or other expensive operations.