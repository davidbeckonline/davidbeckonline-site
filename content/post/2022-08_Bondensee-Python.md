---
title: "Bodensee Level Analysis with Python"
date: 2022-08-20T04:03:52Z
draft: false

featured_image: "/images/2022-08_Bodensee/8_graph.jpg"
---

## Analyzing the level of Lake Bodensee with Python

**In this post I want to discuss a data analysis experiment I recently ran. The goal was to use AWS SageMaker and Python to visualize public data. This post is structured as follows: First, I am going to explain the goal. Then, I will briefly describe the setup. After that, I approached the analysis.**

### Gaol

I like to dig into data. I enjoy questioning data. And I like data visualization. 
A simple but interesting data visualization I really enjoy can be found on the [website]() Hochwasservorhersagezentrale
Baden-Württemberg (HVZ BaWü). This is what is looks like:

![hvz-website](/images/2022-08_Bodensee/2022-08_hvz_website.jpg?classes=border)

The graph shows the current level of Bodensee (Lake Constanze) as a blue line. Furthermore, it shows the long-term mean for each day (green line), the maximum value (red line), and minimum value (black line). *Long-term* in this case means that the values since year 1850 are considered (Update: They seem to have shortened the lookback window to time frame 1980 until 2021).

My goal is to mimic this visual and adjust it a little bit to a few values that I am interested in.

### Setup

I build this analysis in AWS. I simply enjoy working in this cloud environment. And I want to learn more about it. SageMaker is the service of choice in this case. This was recommended to me. I have to admit that I always thought that SageMaker is only for Machine Learning (ML). But of course, ML is all about data analysis. And with that, SageMaker contains all the tools you might want to use for data analysis. So that was my choice.

This was my first project with Jupyter Notebooks. But that experience was excellent. Data was analyzed with Pandas (a Python Data Analysis Library). matplotlib was used for data visualization. But more on that later.

### Data Analysis

#### Get source data

The mentioned HVZ is a public institution and with that, the data is also publicly available. You can download the data from
https://udo.lubw.baden-wuerttemberg.de/public/pages/home/welcome.xhtml
Sorry, all in German.
Select Wasser --> Oberflächengewässer --> Hydrologische Landespegel --> Hydrologische Landespegel (2x!)

![lubw-selection](/images/2022-08_Bodensee/2022-08_lubw-website.jpg?classes=border)

On the next page, select
* Gewässer = Bodensee
* Station = Konstanz Bodensee
* Komponente = Wasserstand, W-Stand, cm
* Produkt = Tagesmaximalwerte
* Zeitraum = 01.01.1850 - 31.12.2022

You can define a future date. You will get the data that is available.

![lubw-result](/images/2022-08_Bodensee/2022-08_lubw_auswahl.jpg?classes=border)

Export the data. This is about 6 MB.

![lubw-result](/images/2022-08_Bodensee/2022-08_lubw_export.jpg?classes=border)

#### Some data cleanup

The output file needs some tweaks.

![lubw-csv](/images/2022-08_Bodensee/2022-08_csv.jpg?classes=border)

This is what I did
* rows 1 - 6 contain the metadata - I removed this
* rows 7 - 8 are empty - deleted as well
* row 9 contains the column headers - I replaced the word "Gewässer" with "Gewasser", just to get rid of the special character

We will rename the columns to English as one of the first steps of the data processing in Python.

#### Data Analysis

I stored the data in an S3 bucket.

Open SageMaker Studio and launch a Python 3 Notebook.

***

**1/ Import Pandas**

```python
#import pandas
import pandas as pd
```
***

**2/ Read csv file**

```python
# read csv file from S3 bucket
bucket ='s3davibec/Bodensee'
data_key = 'Max_KN_1850-bis-heute_clean.csv'
data_location = 's3://{}/{}'.format(bucket, data_key)

df = pd.read_csv(data_location)

# print
print(df.head())
```

Resulting in

![results-step-2](/images/2022-08_Bodensee/2_results.jpg)

***

**3/ Rename to English**

```python
#Rename all columns to English

df = df.rename(columns={
    'Messstellennummer': 'measuring_point_number',
    'Stationsname': 'station_name',
    'Gewasser': 'waters',
    'Parameter': 'parameter',
    'Datum / Uhrzeit': 'date_time',
    'Zeitbezug': 'time_zone',
    'Wert': 'value',
    'Einheit': 'unit',
    'Produkt': 'value_type'
    })

# print
print(df.head())
```

Resulting in

![results-step-3](/images/2022-08_Bodensee/3_results.jpg)

***

**4/ Add Data Conversions**

We need new different data points, like the month-day combination ["mm_dd"].

```python
#add all date conversions
df['date'] = pd.to_datetime(df['date_time']).dt.date
df['year'] = pd.to_datetime(df['date_time']).dt.year
df['month'] = pd.to_datetime(df['date_time']).dt.month
df['day'] = pd.to_datetime(df['date_time']).dt.day

#convert Month and Day to two digit format
import io

df["month"] = df.month.map("{:02}".format)
df["day"] = df.day.map("{:02}".format)

#new column
df["mm_dd"] = df["month"].astype(str) + "_" + df["day"].astype(str)


# print
print(df.head())
```

Resulting in

![results-step-4](/images/2022-08_Bodensee/4_results.jpg)

***

**5/ Reduce Data to the minimum**

Create a new dataset with a minimum of data to make things easier in the next couple of steps.

```python
# creating a new dataset dfs with the minimum data I will need in the next couple of steps
dfs = df[['value','date','year','mm_dd']]

# print
dfs.head()
```

Resulting in

![results-step-5](/images/2022-08_Bodensee/5_results.jpg)

***

**6/ Check Data Types**

This might be a good time to check data types.

```python
# check the data types of the various fields
# apply the dtype attribute
result = dfs.dtypes

print("Output:")
print(result)
```

Resulting in
```
Output:
value     int64
date     object
year      int64
mm_dd    object
dtype: object
```

***

**7/ Get Min, Max, Mean**

Now I am creating a new data set with the three long-term values. For every day of the year, we calculate the Max, the Min, and the Mean.

```python
# create new dataframe with min max mean values for each month date combination
df_mmm = dfs.groupby('mm_dd').agg(value_min=('value', 'min'), 
                             value_mean=('value', 'mean'), 
                             value_max=('value', 'max')).reset_index()

# print
df_mmm.head()
```

Resulting in

![results-step-7](/images/2022-08_Bodensee/7_results.jpg)

***

**8/ Missed the month info on my way, so I need to bring this back in**

I reduced the data in step 5/. But I missed to keep to month info in. For the data visualization, this info is handy. Therefore I re-create a month column.

```python
# extract the first to characters from field mm_dd in order to get the month information back
df_mmm['month'] = df_mmm['mm_dd'].str[:2]

# print
df_mmm.head()
```
Resulting in

![results-step-8](/images/2022-08_Bodensee/8_results.jpg)

***

**9/ Visualize: Max, Min, Avg**

As a first visualization, I am simply setting up the Max, Min, and Mean value. Comparing this to the original graph from the data source, I can check whether my processing is okay.
I use matplotlib for this purpose.

```python
# print min, max, and mean for long-term data
# ---

# import matplotlib
import matplotlib.pyplot as plt

# plot
df_mmm.plot(x='month', kind='line')

plt.ylim(bottom=0, top=800)

# description
plt.title("Level of Lake Constance: 1850 to 2021")
plt.xlabel("Month")
plt.ylabel("Level [cm]")


# add grid
plt.grid()

plt.show()
```

Resulting in

![graph-step-9](/images/2022-08_Bodensee/9_graph.png)

This looks all-right.

***

10/ Add 2022 data

The intention of this step is to create a data frame with Max, Min, Mean, and 2022 data.

```python
# create dataframe with 2022 data based on dfs
dfs2022 = dfs.loc[dfs['year']==2022]
```


```python
# drop not needed columns
dfs2022 = dfs2022.drop(columns=['date','year'], axis=1)
```


```python
#rename value column
dfs2022 = dfs2022.rename(columns={'value': 'value_2022'})

# print
dfs2022.head()
```

Resulting in

![results-step-10_01](/images/2022-08_Bodensee/10_results_01.jpg)


```python
df_mmm_2022 = pd.merge(df_mmm, dfs2022, how="left", on="mm_dd")

# print
df_mmm_2022.head()
```

Resulting in

![results-step-10_02](/images/2022-08_Bodensee/10_results_02.jpg)


Visualize the data.

```python
# print min, max, and mean for long-term data
# ---

# import matplotlib
import matplotlib.pyplot as plt

# plot
df_mmm_2022.plot(x='month', kind='line')

plt.ylim(bottom=0, top=800)

# description
plt.title("Level of Lake Constance: 1850 to 2021")
plt.xlabel("Month")
plt.ylabel("Level [cm]")


# add grid
plt.grid()

plt.show()
```

Resulting in

![graph-step-10](/images/2022-08_Bodensee/10_graph.png)

***

**11/ Check for years with low levels at that time of the year**

In August, people thought that Lake Constance might see a record low for this time of the year. All Germany saw a record droud.
With that, I wanted to understand which years saw lower sea levels.

Get lowest value for the fifth of August:

```python
# get the lowest values on a certain day of the year
# ---

# filter data frame by value
filter_value = {'08_05'}
dfs_min = dfs.loc[dfs['mm_dd'].isin(filter_value)]

# print
dfs_min.head()
```

Resulting in

![results-step-11_01](/images/2022-08_Bodensee/11_results_01.jpg)

Now this data needs to be sorted.

```python
dfs_min = dfs_min.sort_values(by='value', ascending=True)

# print
dfs_min.head(10)
```

Resulting in

![results-step-11_02](/images/2022-08_Bodensee/11_results_02.jpg)

Now we know that the water level was lower than in 2022 in four occasions before: 
* 1949
* 2003
* 2006
* 1964
This is in ascending order. 1949 saw the lowest value ever recorded on 08/05.

It is important to note that there are five years > year 2000 in this (negative) Top10 list.

***

**12a/ Dive deeper into **

Let us have a closer look at the 1949 data.

```python
# create dataframe with 1949 data based on dfs
dfs1949 = dfs.loc[df['year'] == 1949]
```

```python
# drop not needed columns
dfs1949 = dfs1949.drop(columns=['date','year'], axis=1)
```

```python
#rename value column
dfs1949 = dfs1949.rename(columns={'value': 'value_1949'})

# print
dfs1949.head()
```

Resulting in

![results-step-12_01](/images/2022-08_Bodensee/12_results_01.jpg)

```python
# merge the 1949 data to the dataframe created for 2022
df_mmm_1949 = pd.merge(df_mmm_2022, dfs1949, how="left", on="mm_dd")

# print
df_mmm_1949.head()
```

Resulting in

![results-step-12_02](/images/2022-08_Bodensee/12_results_02.jpg)

Create a visualization

```python
# print min, max, and mean for long-term data
# ---

# import matplotlib
import matplotlib.pyplot as plt

# plot
df_mmm_1949.plot(x='month', kind='line')

plt.ylim(bottom=0, top=800)

# description
plt.title("Level of Lake Constance: 1850 to 2021 + 2022 and 1949")
plt.xlabel("Month")
plt.ylabel("Level [cm]")

# add grid
plt.grid()

plt.show()
```

Resulting in

![graph-step-12](/images/2022-08_Bodensee/12_graph.png)


**12b/ Zoom in summer 1949**

We need to have a closer look here.

```python
# let us zoom in a little bit
# ---

# filter data frame by value
filter_value = {'05','06','07','08'}
df_mmm_1949_zoom = df_mmm_1949.loc[df_mmm_1949['month'].isin(filter_value)]

# drop not needed columns
df_mmm_1949_zoom = df_mmm_1949_zoom.drop(columns=['value_max'], axis=1)

# print
df_mmm_1949_zoom.head()
```

Resulting in

![results-step-12_03](/images/2022-08_Bodensee/12_results_03.jpg)

Probably good to check the number of entries.

```python
# check number of rows in df
len(df_mmm_1949_zoom)
```

Resulting in 

```
124 
```
That sounds right.

Now we visualize this.

```python
# print zoom
# ---

# import matplotlib
import matplotlib.pyplot as plt

# plot
df_mmm_1949_zoom.plot(x='mm_dd', kind='line', rot=90)

# plt.ylim(bottom=0, top=800)

# description
plt.title("Level of Lake Constance: 1850 to 2021 + 2022 and 1949")
plt.xlabel("MM_DD")
plt.ylabel("Level [cm]")

# add grid
plt.grid()

plt.show()
```

Resulting in

![graph-step-12_2](/images/2022-08_Bodensee/12_graph_02.png)


***

**13/ All time Max**

Next, we have a look into the all-time max value.

```python
# now let us check out the max value
# ---

# get the max values for all columns
print(dfs.max())
```

Resulting in

```
value           576
date     2022-08-06
year           2022
mm_dd         12_31
dtype: object
```

But in which year?

```python
# now we know that the highest value is 576, but in which year did that occur?

dfs_max = dfs.loc[df['value']==576]

# print
dfs_max
```

Resulting in

![results-step-13_01](/images/2022-08_Bodensee/13_results_01.jpg)

The answer is 1890.
Now we want to see what that year looked like.

```python
# create dataframe with 1890 data based on dfs
dfs1890 = dfs.loc[df['year'] == 1890]
```

```python
# drop not needed columns
dfs1890 = dfs1890.drop(columns=['date','year'], axis=1)
```

```python
#rename value column
dfs1890 = dfs1890.rename(columns={'value': 'value_1890'})

# print
dfs1890.head()
```

Resulting in

![results-step-13_02](/images/2022-08_Bodensee/13_results_02.jpg)

Merge the data.

```python
dfs_mmm_1890 = pd.merge(dfs1890, df_mmm, how="left", on="mm_dd")

# print
dfs_mmm_1890.head()
```

Resulting in

![results-step-13_03](/images/2022-08_Bodensee/13_results_03.jpg)

And now we visualize

```python
# print min, max, and mean for long-term data
# ---

# import matplotlib
import matplotlib.pyplot as plt

# plot
dfs_mmm_1890.plot(x='month', kind='line')

plt.ylim(bottom=0, top=800)

# description
plt.title("Level of Lake Constance: 1850 to 2021 + 1890")
plt.xlabel("Month")
plt.ylabel("Level [cm]")

# add grid
plt.grid()

plt.show()
```

Resulting in

![graph-step-13](/images/2022-08_Bodensee/13_graph.png)

***

**14/ Check year 1999**

I still remember the flood from summer 1999. At that time, the military took us to school. Let us have a look at this year.

```python
# and what was 1999
# ---

# create dataframe with 1999 data based on dfs
dfs1999 = dfs.loc[df['year'] == 1999]
```

```python
# drop not needed columns
dfs1999 = dfs1999.drop(columns=['date','year'], axis=1)
```

```python
#rename value column
dfs1999 = dfs1999.rename(columns={'value': 'value_1999'})

# print
dfs1999.head()
```

Resulting in

![results-step-14_01](/images/2022-08_Bodensee/14_results_01.jpg)

```python
dfs_1890_1999 = pd.merge(dfs1890, dfs1999, how="left", on="mm_dd")

# print

dfs_1890_1999.head()
```

Resulting in

![results-step-14_02](/images/2022-08_Bodensee/14_results_02.jpg)

```python
dfs_mmm_1890_1999 = pd.merge(dfs_1890_1999, df_mmm, how="left", on="mm_dd")

# print
dfs_mmm_1890_1999.head()
```

Resulting in

![results-step-14_03](/images/2022-08_Bodensee/14_results_03.jpg)

And visualization again

```python
# print min, max, and mean for long-term data
# ---

# import matplotlib
import matplotlib.pyplot as plt

# plot
dfs_mmm_1890_1999.plot(x='month', kind='line')

plt.ylim(bottom=0, top=800)

# description
plt.title("Level of Lake Constance: 1850 to 2021 + 1949 and 1999")
plt.xlabel("Month")
plt.ylabel("Level [cm]")

# add grid
plt.grid()

plt.show()#
```

Resulting in

![graph-step-14](/images/2022-08_Bodensee/14_graph.png)