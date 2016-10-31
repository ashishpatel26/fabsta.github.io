---
title: "Pandas Cheat Sheet"
description: "Pandas Cheat Sheet"
category: Programming
tags: [programming]
sidebar:
  nav: "new_side"
---


[this cheat sheet is based on the fantastic work by @Mark_Graph]. 

# Table of contents

* TOC
{:toc}


# Preliminaries
```python
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
from pandas import DataFrame, Series
```

(<a href="#top">Back to top</a>)
<hr>

# Input/Output

## Loading: Get data into DataFrame

### Empty DataFrame
Instantiate an empty DataFrame

```python
df = DataFrame()
```

(<a href="#top">Back to top</a>)
<hr>

### CSV
Load a DataFrame from a CSV file

```python
# often works
df = pd.read_csv('file.csv')
df = pd.read_csv('file.csv', header=0, index_col=0, quotechar='"',sep=':', na_values = ['na', '-', '.', ''])
# specifying "." and "NA" as missing values in the Last Name column and "." as missing values in Pre-Test Score column
df = pd.read_csv('../data/example.csv', na_values={'Last Name': ['.', 'NA'], 'Pre-Test Score': ['.']}) 
# skipping the top 3 rows
df = pd.read_csv('../data/example.csv', na_values=sentinels, skiprows=3)
# interpreting "," in strings around numbers as thousands separators
df = pd.read_csv('../data/example.csv', thousands=',') 
```

(<a href="#top">Back to top</a>)
<hr>

### CSV (Inline)
Get data from inline CSV text to a DataFrame

```python
from io import StringIO
data = """, Animal, Cuteness, Desirable
row-1, dog, 8.7, True
row-2, cat, 9.5, True
row-3, bat, 2.6, False"""
df = pd.read_csv(StringIO(data),
	header=0, index_col=0,
	skipinitialspace=True)
```

(<a href="#top">Back to top</a>)
<hr>

### JSON
```python
import json
json_data = open('data-text.json').read()
data = json.loads(json_data)
for item in data:
    print item
```

### XML
```python
from xml.etree import ElementTree as ET
tree = ET.parse('../../data/chp3/data-text.xml')
root = tree.getroot()
print root
data = root.find('Data')
all_data = []
for observation in data:
    record = {}
    for item in observation:
        lookup_key = item.attrib.keys()[0]
        if lookup_key == 'Numeric':
            rec_key = 'NUMERIC'
            rec_value = item.attrib['Numeric']
        else:
            rec_key = item.attrib[lookup_key]
            rec_value = item.attrib['Code']
        record[rec_key] = rec_value
    all_data.append(record)
print all_data
```    


### Excel
Load DataFrames from a Microsoft Excel file

```python
# Each Excel sheet in a Python dictionary
workbook = pd.ExcelFile('file.xlsx')
d = {} # start with an empty dictionary
for sheet_name in workbook.sheet_names:
	df = workbook.parse(sheet_name)
	d[sheet_name] = df
```

(<a href="#top">Back to top</a>)
<hr>


### MySQL
 Load a DataFrame from a MySQL database
 
```python
import pymysql
from sqlalchemy import create_engine
engine = create_engine('mysql+pymysql://'
+'USER:PASSWORD@HOST/DATABASE')
df = pd.read_sql_table('table', engine)
```

(<a href="#top">Back to top</a>)
<hr>


### Combine DataFrame
Data in Series then combine into a DataFrame

```python
# Example 1 ...
s1 = Series(range(6))
s2 = s1 * s1
s2.index = s2.index + 2# misalign indexes
df = pd.concat([s1, s2], axis=1)
# Example 2 ...
s3 = Series({'Tom':1, 'Dick':4, 'Har':9})
s4 = Series({'Tom':3, 'Dick':2, 'Mar':5})
df = pd.concat({'A':s3, 'B':s4 }, axis=1)
```

(<a href="#top">Back to top</a>)
<hr>

### From Dictionary
Get a DataFrame from a Python dictionary

#### default --- assume data is in columns
```python
df = DataFrame({
	'col0' : [1.0, 2.0, 3.0, 4.0],
	'col1' : [100, 200, 300, 400]
})
```

#### use helper method for data in rows
```python
df = DataFrame.from_dict({ # data by row
# rows as python dictionaries
	'row0' : {'col0':0, 'col1':'A'},
	'row1' : {'col0':1, 'col1':'B'}
	}, orient='index')
df = DataFrame.from_dict({ # data by row
# rows as python lists
	'row0' : [1, 1+1j, 'A'],
	'row1' : [2, 2+2j, 'B']
	}, orient='index')
```

(<a href="#top">Back to top</a>)
<hr>

### Testing
Create play/fake data (useful for testing)

#### --- simple - default integer indexes
```python
df = DataFrame(np.random.rand(50,5))
```

#### --- with a time-stamp row index:
```python
df = DataFrame(np.random.rand(500,5))
df.index = pd.date_range('1/1/2005',
periods=len(df), freq='M')
```

#### --- with alphabetic row and col indexes and a "groupable" variable
```python
import string
import random
r = 52 # note: min r is 1; max r is 52
c = 5
df = DataFrame(np.random.randn(r, c),
	columns = ['col'+str(i) for i in range(c)],
	index = list((string. ascii_uppercase+ string.ascii_lowercase)[0:r]))
	df['group'] = list(''.join(random.choice('abcde')
	for _ in range(r)) )
```

(<a href="#top">Back to top</a>)
<hr>


## Saving as DataFrame

### CSV
Saving a DataFrame to a CSV file

```python
df.to_csv('name.csv', encoding='utf-8')
```

(<a href="#top">Back to top</a>)
<hr>

### Excel
Saving DataFrames to an Excel Workbook

```python
from pandas import ExcelWriter
writer = ExcelWriter('filename.xlsx')
df1.to_excel(writer,'Sheet1')
df2.to_excel(writer,'Sheet2')
writer.save()
```

(<a href="#top">Back to top</a>)
<hr>

### MySQL
Saving a DataFrame to MySQL

```python
import pymysql
from sqlalchemy import create_engine
e = create_engine('mysql+pymysql://' +
	'USER:PASSWORD@HOST/DATABASE')
df.to_sql('TABLE',e, if_exists='replace')
```

(<a href="#top">Back to top</a>)
<hr>

### Python object
Saving to Python objects

```python
d = df.to_dict() # to dictionary
str = df.to_string() # to string
m = df.as_matrix() # to numpy matrix
```

(<a href="#top">Back to top</a>)
<hr>

# Summary: Selecting using Index

Using the DataFrame index to select columns

s = df['col_label']  # returns Series
Note: the difference in return type with the first two
<hr>

Using the DataFrame index to select rows

df = df['from':'inc_to']# label slice

<hr>

Using the DataFrame index to select a cross-section

# r and c can be scalar, list, slice

<hr>

Using the DataFrame index to select a cell

# r and c must be label or integer

<hr>

v = df.get_value(r, c) # get by row, col
__Note__: the indexing attributes (.loc, .iloc, .ix, .at .iat) can




<hr>

# --- some Index attributes
```


# Whole DataFrame

## Content/Structure
Peek at the DataFrame contents/structure

df.info() # index & data types

(<a href="#top">Back to top</a>)
<hr>

DataFrame non-indexing attributes

dfT = df.T # transpose rows and cols
<hr>
DataFrame utility methods

df = df.copy() # copy a DataFrame
df = df.sort(['sales'], ascending=[False]) 
<hr>

DataFrame iteration methods

df.iteritems()# (col-index, Series) pairs
```

(<a href="#top">Back to top</a>)
<hr>

## Maths
Maths on the whole DataFrame (not a complete list)

df = df.abs() # absolute values
Note: The methods that return a series default to
<hr>

DataFrame select/filter rows/cols on label values

df = df.filter(items=['a', 'b']) # by col
```

<hr>

# Columns

Each DataFrame column is a pandas Series object

Get column index and labels

idx = df.columns # get col index

<hr>

### Data type conversions
st = df['col'].astype(str)# Series dtype
```

<hr>

### Common column-wide methods/attributes
value = df['col'].dtype # type of data
<hr>

label = df['col1'].idxmin()

<hr>

s = df['col'].isnull()
Note: also rolling_min(), rolling_max(), and many more.
<hr>

### Position of a column index label
Get the integer position of a column index label

j = df.columns.get_loc('col_name')

<hr>

Test if column index values are unique/monotonic

if df.columns.is_unique: pass # ...
```

<hr>


## Selecting

### Columns
s = df['colName'] # select col to Series

<hr>

s = df.a # same as s = df['a']
```
Trap: column names must be valid identifiers.
<hr>

### Selecting columns with .loc, .iloc and .ix
df = df.loc[:, 'col1':'col2'] # inclusive

<hr>

### Conditional selection

```python
df.query('A > C')
df.query('A > 0') 
df.query('A > 0 & A < 1')
df.query('A > B | A > C')
df[df['coverage'] > 50] # all rows where coverage is more than 50
df[(df['deaths'] > 500) | (df['deaths'] < 50)]
df[(df['score'] > 1) & (df['score'] < 5)]
df[~(df['regiment'] == 'Dragoons')] # Select all the regiments not named "Dragoons"
#
df[df['age'].notnull() & df['sex'].notnull()]  # ignore the missing data points
```

(<a href="#top">Back to top</a>)
<hr>

### Is in

```python
df[df.name.isin(value_list)]   # value_list = ['Tina', 'Molly', 'Jason']
df[~df.name.isin(value_list)]
```

### Partial matching

```python
df2[df2.E.str.contains("tw|ou")]
```

### Regex

```python
df['raw'].str.contains('....-..-..', regex=True)  # regex
```

(<a href="#top">Back to top</a>)
<hr>


## Changing

### Rename column labels
df.rename(columns={'old1':'new1','old2':'new2'}, inplace=True)
```
Note: can rename multiple columns at once.
<hr>



#### Column
Adding new columns to a DataFrame

df['new_col'] = range(len(df))
```

<hr>

#### Vectorised arithmetic on columns
df['proportion']=df['count']/df['total']
<hr>

#### Append a column of row sums to a DataFrame
df['Total'] = df.sum(axis=1)
Note: also means, mins, maxs, etc.
<hr>

#### Apply numpy mathematical functions to columns
df['log_data'] = np.log(df['col1'])
Note: Many more numpy mathematical functions.

(<a href="#top">Back to top</a>)
<hr>

#### Set column values set based on criteria
df['b']=df['a'].where(df['a']>0,other=0)
```
<hr>

Swap column contents – change column order

df[['B', 'A']] = df[['A', 'B']]

<hr>

Dropping (deleting) columns (mostly by label)

df = df.drop('col1', axis=1)
#### drop columns with column names where the first three letters of the column names was 'pre'

```python
# drop columns with column names where the first three letters of the column names was 'pre'
cols = [c for c in df.columns if c.lower()[:3] != 'pre']
df=df[cols]
```



<hr>

### Multiply every column in DataFrame by Series
df = df.mul(s, axis=0) # on matched rows
Note: also add, sub, div, etc.
<hr>

# Rows

## Info

### Get Position
Get integer position of rows that meet condition

a = np.where(df['col'] >= 2) #numpy array
<hr>

Test if two DataFrames have same row index

len(a)==len(b) and all(a.index==b.index)     # Get the integer position of a row or col index label
<hr>


Test if the row index values are unique/monotonic

if df.index.is_unique: pass # ...
```

(<a href="#top">Back to top</a>)
<hr>


### Get the row index and labels
idx = df.index      # get row index

<hr>

df.index = idx     # new ad hoc index
note: old index stored as a col in df
df.index = range(len(df)) # set with list

<hr>

## Selecting

### By column values
Boolean row selection by values in a column

df = df[df['col2'] >= 0.0]

<hr>

Selecting rows using isin over multiple columns

## fake up some data

<hr>

Select a slice of rows by integer position

[inclusive-from : exclusive-to [: step]]
Trap: a single integer without a colon is a column label

<hr>


### Slice of rows by label/index
Select a slice of rows by label/index

[inclusive-from : inclusive–to [ : step]]
<hr>


## Manipulating

df = original_df.append(more_rows_in_df)
```
<hr>

### Append a row of column totals to a DataFrame
# Option 1: use dictionary comprehension
<hr>


df = df.drop('row_label')

<hr>

### Drop duplicates in the row index
df['index'] = df.index # 1 create new col
<hr>

for (index, row) in df.iterrows(): # pass
```
<hr>

## Sorting
Sorting DataFrame rows values

df = df.sort(df.columns[0], ascending=False)
<hr>


Sort DataFrame by its row index

df.sort_index(inplace=True) # sort by row
<hr>

Random selection of rows

import random as r
```


df.take(np.random.permutation(len(df))[:3])
```

<hr>


# Cells

## Selecting

### By row and column
Selecting a cell by row and column labels

value = df.at['row', 'col']
Note: .at[] fastest label based scalar lookup
<hr>

### By integer position
Selecting a cell by integer position

value = df.iat[9, 3] # [row, col]

<hr>

### Slice by labels
df = df.loc['row1':'row3', 'col1':'col3']
Note: the "to" on this slice is inclusive.
<hr>



Selecting a range of cells by int position

df = df.iloc[2:4, 2:4] # subset of the df
```
<hr>

### By label and/or Index
.ix for mixed label and integer position indexing

value = df.ix[5, 'col1']
```

(<a href="#top">Back to top</a>)
<hr>



## Manipulating

### Updating

Setting a cell by row and column labels

df.at['row', 'col'] = value

<hr>

df.loc['A':'C', 'col1':'col3'] = np.nan
Remember: inclusive "to" in the slice
<hr>

df.iloc[0, 0] = value # [row, col]

<hr>

df.iloc[0:3, 0:5] = value
```

<hr>


<hr>

# Joining/Combining DataFrames

Three ways to join two DataFrames:
df_new = pd.merge(left=df1, right=df2, how='outer', left_index=True, right_index=True)
How: 'left', 'right', 'outer', 'inner'
<hr>

df_new = pd.merge(left=df1, right=df2, how='left', left_on='col1', right_on='col2')
```
<hr>

df_new = df1.join(other=df2, on='col1',how='outer')
__Note__: DataFrame.join() joins on indexes by default.
<hr>

df=pd.concat([df1,df2],axis=0)#top/bottom
__Trap__: can end up with duplicate rows or cols
<hr>

df = df1.combine_first(other=df2)
Uses the non-null values from df1. The index of the

(<a href="#top">Back to top</a>)
<hr>

# GroupBy: Split-Apply-Combine
The pandas "groupby" mechanism allows us to split the


gb = df.groupby('cat') # by one columns
```


<hr>

for name, group in gb:

<hr>

dfa = df.groupby('cat').get_group('a')

<hr>

# apply to a column ...
```
<hr>

gb = df.groupby('cat')
```
<hr>

# transform to group z-scores, which have
```
<hr>

# select groups with more than 10 members
```

<hr>

## Group by a row index (non-hierarchical index)
df = df.set_index(keys='cat')
```

(<a href="#top">Back to top</a>)
<hr>

# Working with dates, times and their indexes
t = pd.Timestamp('2013-01-01')
```
<hr>

ts = ['2015-04-01 13:17:27', '2014-04-02 13:17:29']
# Series of Timestamps (good)
```
<hr>

t = ['09:08:55.7654-JAN092002', '15:42:02.6589-FEB082016']
Also: %B = full month name; %m = numeric month;
<hr>

date_strs = ['2014-01-01', '2014-04-01','2014-07-01', '2014-10-01']
```

<hr>

## From DatetimeIndex to Python datetime objects
dti = pd.DatetimeIndex(pd.date_range(
<hr>

df['date'] = [x.date() for x in df['TS']]
Note: converts to datatime.date or datetime.time. But
<hr>

df = DataFrame(np.random.randn(20,3))
Note: from period to timestamp defaults to the point in
<hr>

pi = pd.period_range('1960-01','2015-12',freq='M')

<hr>

dr = pd.date_range('2013-01-01', '2013-12-31', freq='D')

<hr>

```python
# 1st example returns string not Timestamp

<hr>

## The tail of a time-series DataFrame
df = df.last("5M") # the last five months
```

<hr>


## Upsampling and downsampling
# upsample from quarterly to monthly
```

<hr>

## Time zones
```	

<hr>

## Row selection with a time-series index
# start with the play data above
```
<hr>

## The Series.dt accessor attribute
t = ['2012-04-14 04:06:56.307000', '2011-05-14 06:14:24.457000', '2010-06-14 08:23:07.520000']
```

(<a href="#top">Back to top</a>)
<hr>

# Working with missing and non-finite data
s = Series( [8,None,float('nan'),np.nan])

<hr>

df = df.dropna() # drop all rows with NaN

<hr>

df.fillna(0, inplace=True) # np.nan -> 0

<hr>


s = Series([float('inf'), float('-inf'),np.inf, -np.inf])
```


```python
b = np.isfinite(s)
```
(<a href="#top">Back to top</a>)
<hr>


# Working with Categorical Data
s = Series(['a','b','a','c','b','d','a'],
```
<hr>

s = Series(['a','b','a','c','b','d','a'], dtype='category')

<hr>

s = Series(list('abc'), dtype='category')
Trap: category must be ordered for it to be sorted
<hr>

s = Series(list('abc'), dtype='category')
```
<hr>

s = s.cat.add_categories([4])

<hr>

s = s.cat.remove_categories([4])
```
(<a href="#top">Back to top</a>)
<hr>