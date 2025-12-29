---
layout: post
title: "Pandas in Practice: A Developer's Cheatsheet"
date: 2025-12-28 10:30:00 +0545
categories: [Python, Data Science]
tags: [pandas, python, data-manipulation, data-analysis, cheatsheet]
---

# Pandas in Practice: A Developer's Cheatsheet

## Introduction

In the world of Python data analysis, the `pandas` library is an indispensable tool. It provides high-performance, easy-to-use data structures and data analysis tools that make working with structured data intuitive and efficient. This post is not about the theory behind pandas; it's a practical, hands-on cheatsheet designed for developers who need to get things done. We'll cover the essentials you'll use 95% of the time.

## Installation

First, ensure you have pandas installed in your environment. If not, a simple pip install will suffice.

```bash
pip install pandas
```

## Core Data Structures: Series and DataFrame

Pandas has two primary data structures: `Series` (1-dimensional) and `DataFrame` (2-dimensional). Think of a `DataFrame` as a spreadsheet or a SQL table, and a `Series` as a single column within it.

### Creating a DataFrame

Most of the time, you'll be creating DataFrames, either by reading a file or from other Python objects like dictionaries or lists.

#### From a Dictionary

This is a common way to create a small, sample DataFrame.

```python
import pandas as pd

# A dictionary where keys are column names and values are lists of column data
data = {
    'Name': ['Alice', 'Bob', 'Charlie', 'David', 'Eva'],
    'Age': [24, 27, 22, 32, 29],
    'City': ['New York', 'Los Angeles', 'Chicago', 'Houston', 'Phoenix'],
    'Salary': [70000, 80000, 65000, 95000, 82000]
}

df = pd.DataFrame(data)
print(df)
```
**Output:**
```
      Name  Age         City  Salary
0    Alice   24     New York   70000
1      Bob   27  Los Angeles   80000
2  Charlie   22      Chicago   65000
3    David   32      Houston   95000
4      Eva   29      Phoenix   82000
```

## Data Inspection

Once you have a DataFrame, the first step is always to understand its structure and content.

### `head()`, `tail()`, and `sample()`
Get the first `n` rows, last `n` rows, or a random sample of `n` rows.

```python
# Get the first 3 rows
print(df.head(3))

# Get the last 2 rows
print(df.tail(2))

# Get a random sample of 2 rows
print(df.sample(2))
```

### `info()`
Provides a concise summary of the DataFrame, including the index dtype and column dtypes, non-null values, and memory usage. This is crucial for spotting missing data.

```python
df.info()
```
**Output:**
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 5 entries, 0 to 4
Data columns (total 4 columns):
 #   Column  Non-Null Count  Dtype
---  ------  --------------  -----
 0   Name    5 non-null      object
 1   Age     5 non-null      int64
 2   City    5 non-null      object
 3   Salary  5 non-null      int64
dtypes: int64(2), object(2)
memory usage: 288.0+ bytes
```

### `describe()`
Generates descriptive statistics for numerical columns (like count, mean, std, min, max, and quartiles).

```python
print(df.describe())
```
**Output:**
```
             Age        Salary
count   5.000000      5.000000
mean   26.800000  78400.000000
std     4.086563  11949.895397
min    22.000000  65000.000000
25%    24.000000  70000.000000
50%    27.000000  80000.000000
75%    29.000000  82000.000000
max    32.000000  95000.000000
```

## Data Selection and Indexing

Selecting the right piece of data is fundamental. Pandas offers powerful, intuitive ways to do this.

### Selecting Columns
You can select a single column (which returns a `Series`) or multiple columns.

```python
# Select a single column
ages = df['Age']
print(ages)

# Select multiple columns
name_and_city = df[['Name', 'City']]
print(name_and_city)
```

### Selecting Rows: `.loc` and `.iloc`

This is a common point of confusion. The distinction is simple:
- `.loc[]`: **Label-based** selection. You use the actual index labels or column names.
- `.iloc[]`: **Integer-position-based** selection. You use the integer index (from 0 to length-1).

```python
# --- Using .loc ---
# Select row with index label 1
print(df.loc[1])

# Select rows with index labels 0 and 2
print(df.loc[[0, 2]])

# Select rows 0 to 2, and columns 'Name' and 'Salary'
print(df.loc[0:2, ['Name', 'Salary']])


# --- Using .iloc ---
# Select row at integer position 1
print(df.iloc[1])

# Select rows at integer positions 0 to 2 (exclusive of 3)
print(df.iloc[0:3])

# Select rows 0-2 and columns 0-1
print(df.iloc[0:3, 0:2])
```

### Conditional Selection (Boolean Indexing)
This is the most powerful way to select data. You can filter your DataFrame based on one or more conditions.

```python
# Find everyone older than 25
older_than_25 = df[df['Age'] > 25]
print(older_than_25)

# Find everyone who is older than 25 AND lives in Los Angeles
la_resident = df[(df['Age'] > 25) & (df['City'] == 'Los Angeles')]
print(la_resident)
```

## Data Cleaning

Real-world data is messy. Here’s how to handle missing values. Let's first add some `NaN` (Not a Number) values.

```python
import numpy as np

df.loc[2, 'Salary'] = np.nan
df.loc[4, 'Age'] = np.nan
print(df)
```

### Finding Missing Values: `isnull()`
Returns a boolean DataFrame indicating where data is missing.

```python
print(df.isnull().sum()) # Get a count of nulls per column
```

### Dropping Missing Values: `dropna()`
Removes rows or columns with missing data.

```python
# Drop any row with at least one missing value
cleaned_rows = df.dropna()
print(cleaned_rows)

# Drop columns with at least one missing value
cleaned_cols = df.dropna(axis='columns')
print(cleaned_cols)
```

### Filling Missing Values: `fillna()`
Replaces `NaN` values with a specified value or method.

```python
# Fill missing Age with the mean age
mean_age = df['Age'].mean()
df_filled = df.copy() # Work on a copy
df_filled['Age'].fillna(mean_age, inplace=True)

# Fill missing Salary with 0
df_filled['Salary'].fillna(0, inplace=True)
print(df_filled)
```

## Data Manipulation and Operations

### Grouping: `groupby()`
The `groupby()` operation is a cornerstone of data analysis. It involves splitting the data into groups based on some criteria, applying a function to each group independently, and combining the results.

```python
# Let's add a 'Department' column for a better example
df['Department'] = ['HR', 'Engineering', 'HR', 'Engineering', 'Sales']

# Group by department and calculate the average salary
avg_salary_by_dept = df.groupby('Department')['Salary'].mean()
print(avg_salary_by_dept)

# Group by multiple columns and get the size of each group
group_size = df.groupby(['Department', 'City']).size()
print(group_size)
```

### Applying Functions: `apply()`
`apply()` lets you run a custom function on every row or column.

```python
# A function to classify salary into brackets
def salary_bracket(salary):
    if salary > 80000:
        return 'High'
    elif salary > 70000:
        return 'Medium'
    else:
        return 'Low'

# Apply the function to the 'Salary' column
df['Salary Bracket'] = df['Salary'].apply(salary_bracket)
print(df)
```

### Merging and Joining
Pandas provides SQL-like functionality to combine DataFrames.

```python
# Create another DataFrame
departments = pd.DataFrame({
    'Department': ['HR', 'Engineering', 'Sales', 'Marketing'],
    'Manager': ['Carol', 'John', 'Michael', 'Susan']
})

# Merge df with departments
df_merged = pd.merge(df, departments, on='Department', how='left')
print(df_merged)
```

## File I/O

Reading from and writing to files is seamless.

### Reading a CSV
```python
# Assuming you have a file named 'my_data.csv'
# df_from_csv = pd.read_csv('my_data.csv')

# You can also read from a URL
# url = 'https://raw.githubusercontent.com/someuser/some-repo/main/data.csv'
# df_from_url = pd.read_csv(url)
```

### Writing to a CSV
```python
# Write the merged DataFrame to a new CSV file
# The index=False argument prevents pandas from writing the row index
df_merged.to_csv('employee_data_with_managers.csv', index=False)
```
Pandas also supports other formats like `read_excel()`, `to_excel()`, `read_json()`, `to_json()`, `read_sql()`, and more.

## Conclusion

Pandas is a feature-rich library, but these core operations form the bedrock of most data manipulation tasks. By mastering data creation, inspection, selection, cleaning, grouping, and I/O, you have a powerful toolkit for transforming raw data into actionable insights. The key is to practice: create your own DataFrames, experiment with these methods, and gradually explore the more advanced features as you need them.

{% include inarticle-adsense.html %}

## Suggested Reading

- **Official Pandas Documentation:** The best and most comprehensive resource. [pandas.pydata.org/docs/](https://pandas.pydata.org/docs/)
- **"Python for Data Analysis" by Wes McKinney:** Written by the creator of pandas, it's a foundational text.
- **Real Python Pandas Tutorials:** A collection of excellent, practical guides. [realpython.com/pandas-dataframe/](https://realpython.com/pandas-dataframe/)
