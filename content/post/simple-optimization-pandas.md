+++
date = '2025-01-08T16:46:06+02:00'
draft = false
title = 'Simple Pandas Optimizations: A Quick Guide'
tags = ['pandas', 'optimization', 'data-science', 'python']
+++

## Introduction
Pandas is incredibly powerful but there are always some tricks to make it faster. I tried to generalize some simple yet effective optimizations that can help with performance. Most of them are obvious but often ignored.
<!--more-->

## 1. Use Appropriate Data Types

One of the most impactful optimizations is using the right data types:

```python
# Before optimization
df = pd.read_csv('data.csv')

# After optimization
df = pd.read_csv('data.csv', dtype={
    'user_id': 'int32',      # Instead of int64
    'status': 'category',    # Instead of string/object
    'timestamp': 'datetime64' # Instead of string
})
```

Using appropriate data types can reduce memory usage by up to 50% and improve processing speed.
Pick a proper data type though, as it can lead to unexpected results if not chosen correctly.
## 2. Vectorization Over Loops

Always prefer vectorized operations over explicit loops (think of `vectorization` as a way to apply an operation to an entire array or series at once):

```python
# Slow: Using loops
for i in range(len(df)):
    df.loc[i, 'total'] = df.loc[i, 'price'] * df.loc[i, 'quantity']

# Fast: Vectorized operation
df['total'] = df['price'] * df['quantity']
```

## 3. Efficient String Operations

When working with strings, use built-in string methods instead of `apply()`:

```python
# Slower
df['name'] = df['name'].apply(str.lower)

# Faster
df['name'] = df['name'].str.lower()
```

## 4. Smart Filtering

Use efficient filtering techniques:

```python
# Less efficient
df[df['column'].apply(lambda x: x.startswith('prefix'))]

# More efficient
df[df['column'].str.startswith('prefix')]

# Even better: Use boolean indexing when possible
df[df['value'] > 100]
```

## 5. Bulk Operations

Process data in bulk rather than incrementally:

```python
# Inefficient: Appending rows one by one
# However, sometimes it's necessary!
for data in new_data:
    df = df.append(data, ignore_index=True)

# Efficient: Combine all data first, then create DataFrame
df = pd.DataFrame(new_data)
```

## 6. Use Query Method for Complex Conditions

For complex filtering, `query()` can be more readable and sometimes faster:

```python
# Less readable
df[(df['age'] > 25) & (df['salary'] > 50000) & (df['department'] == 'IT')]

# More readable but not potentially faster
df.query('age > 25 and salary > 50000 and department == "IT"')
```

## 7. Batch Processing for Large Datasets

When dealing with large files, process them in chunks:

```python
chunk_size = 10000
for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # Process each chunk
    do_some_processing(chunk)
```

## Memory Impact
These optimizations can lead to significant improvements:

* Appropriate dtypes: 30-50% memory reduction
* Category type for strings: Up to 90% memory reduction for repeated strings
* Vectorization: 10-100x speed improvement over loops
* Batch processing: Enables handling of datasets larger than RAM

Remember to profile your specific use case, as the effectiveness of each optimization can vary depending on your data structure and operations.