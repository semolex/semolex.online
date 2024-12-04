+++
date = '2024-12-04T16:33:34+02:00'
draft = false
title = 'Python: Optimize Dynamic Object Creation'
+++

## Introduction

Python is extremely flexible and powerful, but it's not always the fastest language.
And in many cases it really doesn't matter. Because raw speed is not the only thing that matters.
Python ecosystem has a lot of libraries that are more performant than similar libraries in other languages.
But in general, you should know your language and its features to write efficient code.
What is even more important - Python gives you a lot of ways to handle the way you do some things.
You know - there are multiple ways to create objects in Python.
<!--more-->

## Different Ways to Create Objects

I collected some well known and less known ways to create objects in Python.
And by `object` I mean a simple structured object with some attributes. 
Examples:
```python
from collections import namedtuple
from dataclasses import dataclass
import sys
from typing import Any


# Test object definitions
class ClassicClass:
    def __init__(self, x: int, y: int, name: str, value: float):
        self.x = x
        self.y = y
        self.name = name
        self.value = value


class SlotsClass:
    __slots__ = ['x', 'y', 'name', 'value']

    def __init__(self, x: int, y: int, name: str, value: float):
        self.x = x
        self.y = y
        self.name = name
        self.value = value


@dataclass
class DataClass:
    x: int
    y: int
    name: str
    value: float


@dataclass
class SlottedDataClass:
    __slots__ = ['x', 'y', 'name', 'value']
    x: int
    y: int
    name: str
    value: float


NamedTupleClass = namedtuple('NamedTupleClass', ['x', 'y', 'name', 'value'])
```

## Python Object Types Performance Comparison

This benchmark compares different Python object types performance characteristics for creation and attribute access operations.
Instead of setting slots like I did - you can use `@dataclass(slots=True)` decorator to enable slots behavior (Python 3.10+).
I used `pytest-benchmark` to measure performance of different object types.
Let's agree on the following:
- Synthetic test cases are not always the best way to measure performance.
- Different environments and Python versions can give different results.
- There are some common patterns that are represented across different versions and environments.
- I randomly picked one run result to show those common patterns.
- The fewer attributes you have - the less memory overhead you have.

### Results

| Type                  | Access Time (ns) |          | Creation Time (ns) |          |
|----------------------|------------------|----------|-------------------|----------|
|                      | Mean (StdDev)    | OPS      | Mean (StdDev)     | OPS      |
| Named Tuple          | 141.64 (17.51)   | 7.06M/s  | 291.55 (53.55)    | 3.43M/s  |
| Slots Class          | 149.00 (19.81)   | 6.71M/s  | 287.97 (55.78)    | 3.47M/s  |
| Slotted Dataclass    | 149.61 (17.72)   | 6.68M/s  | 356.69 (255.35)   | 2.80M/s  |
| Dataclass            | 166.10 (19.19)   | 6.02M/s  | 419.27 (249.54)   | 2.38M/s  |
| Classic Class        | 166.59 (19.31)   | 6.00M/s  | 440.08 (1604.14)  | 2.27M/s  |

### Performance Analysis

#### Attribute Access
- Named Tuple provides fastest attribute access at 141.64ns
- Slots-based implementations (Slots Class and Slotted Dataclass) are close second
- Regular Dataclass and Classic Class are ~15% slower for access

#### Object Creation
- Slots Class and Named Tuple are fastest and most consistent
- Slotted Dataclass shows moderate performance with some variability
- Classic Class shows highest creation time and extreme variability (StdDev: 1604ns)

### Recommendations

1. **For Attribute-Heavy Operations**
   - Named Tuple if immutability is desired
   - Slots Class if mutability is needed

2. **For Creation-Heavy Operations**
   - Slots Class offers best balance of creation speed and stability
   - Named Tuple is good alternative if immutability is needed

3. **General Use**
   - Slotted Dataclass provides good overall performance with modern Python features
   - Regular Dataclass when flexibility is more important than performance

### Notes
- All measurements in nanoseconds (ns)
- OPS = Operations Per Second in millions
- StdDev indicates measurement variability
- Results may vary based on Python version and system characteristics

### Conclusion
Do not take performance benchmarks as the only factor when choosing object types.
In many cases semantics and readability are more important than raw performance.
But there are some cases where even slight performance improvements can make a difference.

One case is when you need to create a lot of objects in a short time or `duck type` object dynamically, so they have the same attributes as required by some function.
For example, you are using third-party library that requires specific object structure but the actual object is either too complex or requires additional data.
However, the final result is not dependent on the object type, but on the object attributes, etc.

You should also consider an _application_ where you are going to use those objects.
Think about how you're going to serialize/deserialize them, how you're going to store them, how you're going to access them, etc.

I do like slots-based classes but commonly use dataclasses for their simplicity and readability.
And as you can see - they behave like an enhanced version of slots-based classes and more widely used in modern Python code.