---
title: "Air Quality in Salt Lake City"
date: 2020-02-17
tags: [Python, Pandas, Data Visualization]
header:
  image: #"/image/"
excerpt: "Using Python to collect and explore EPA air quality data."
---

**Abstract**:

Here I investigate how Salt Lake City's air quality has changed over time by downloading 38 years worth of data from the EPA, segmenting it into various dataframes with pandas, then exploring things visually with matplotlib.

**Intro**:

As a child, I remember watching my dad clean the fish tank. This was a bit of a monthly ritual, as over that length of time it would go from clear to cloudy to “we should not be pet owners” dirty. I always felt bad for the fish in the days preceding this cleaning – when they were clearly bopping around in some nasty stuff:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/dirty_tank_compressed.jpg" alt="dirty fish tank">

</center>

<p style="text-align: center; font-style: italic;"> Not the fish tank of my childhood, but similar. </p>

Household fish aren’t the only ones subjected to such conditions. Many of us live in areas with dirty air. Here’s a map of PM2.5 pollution (tiny particles that damage lungs) across the U.S.:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/US_air_3.jpg" alt="U.S. map of PM2.5 air pollution">

</center>

<p style="text-align: center; font-style: italic;"> PM2.5 pollution across the U.S. </p>

The blotch of red towards the middle-left of the country is Salt Lake City, where due to the perimeter of mountains surrounding the inhabited valley floor (forming a bowl-like geometry), nasty air can get trapped and accumulate. This is especially bad in the winter, when a blanket of warm air forms a “lid” on top of the colder valley floor – this is called the *winter inversion*, which is a spooky name.

I had heard about this prior to moving to SLC, but so far this winter air quality hasn't seemed to be an issue. Is this inversion business just media hype? Is nasty air a thing of the past? I was curious to see what the air quality in SLC is today and how it has changed over time.

Poking around, I came across an article in the Deseret News titled “Visualizing SLC air pollution in 35 years and what it tells us.” The [article]( https://www.deseret.com/2015/5/7/20564270/visualizing-slc-air-pollution-in-35-years-and-what-it-tells-us) links to a visualization which unfortunately returns “site can’t be reached.” I sent the author a note to let her know, and in the meantime decided to see if I could pull air quality data and visualize it myself.

Fortunately, the EPA makes air quality data available to the public via their AirData Quality Monitors [web app](https://epa.maps.arcgis.com/apps/webappviewer/index.html?id=5f239fd3e72f424f98ef3d5def547eb5&extent=-146.2334,13.1913,-46.3896,56.5319). Below are all the monitors across the U.S. for the 5 primary pollutants used in evaluating air quality (carbon monoxide, nitrogen dioxide, ozone, PM2.5, and sulfur dioxide):

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/US_monitors_reduced.png" alt="EPA air quality monitors in the U.S.">

</center>

<p style="text-align: center; font-style: italic;"> Air quality monitors in the U.S. </p>

Zooming in on SLC, we can see the valley has 5 active monitors:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/valley_monitors_reduced.png" alt="Air quality monitors in Salt Lake City">

</center>

<p style="text-align: center; font-style: italic;"> Air quality monitors in Salt Lake City. </p>

The EPA takes the pollutant concentration data from these monitors (some measured in parts per million, others in parts per billion), and translates them into a more intuitive measurement - AQI. AQI stands for Air Quality Index. This number is calculated for each of the primary pollutants, and the one with the highest AQI is what gets reported. The chart below might help clarify things:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/AQI_calculation_table_reduced.png" alt="AQI table per primary pollutant">

</center>

<p style="text-align: center; font-style: italic;"> Detail on how AQI is calculated per pollutant. </p>

Again, whichever pollutant has the highest AQI is what gets reported that day. While that might seem convoluted at first glance, it does make communicating air quality fairly straightforward. There are 6 categories total, color-coded to help make things a bit more intuitive:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/AQI_table_3_reduced.png" alt="AQI table: 0-50=good, 51-100=moderate,101-150=unhealthy for sensitive groups, 151-200=unhealthy, 210-300=very unhealthy, hazardous=301-500">

</center>

<p style="text-align: center; font-style: italic;"> The Air Quality Index (AQI) categories. </p>

One final summary:
 - The EPA uses standard formulas to turn non-intuitive pollutant concentration data into a more intuitive Air Quality Index, or AQI
 - AQI ranges from 0 to 500, which lower numbers signifying better air quality
 - The AQI reported on a given day is the highest AQI among the 5 primary pollutants it is calculated for (ozone, PM2.5 aka particle pollution, carbon monoxide, sulfur dioxide, and nitrogen dioxide)
 - In large cities (more than 350,000 people), state and local agencies are required to report the AQI to the public daily
 - When the AQI is above 100, agencies must report which groups (such as children or people with asthma), may be sensitive to that pollutant.
 - If two or more pollutants have AQI values above 100 on a given day,agencies must report all the groups that are sensitive to those pollutants. For example, if a community’s AQI is 120 for ozone and 105 for particle pollution, the AQI value for that day would be announced as 120 for ozone. The announcements would note that particle pollution levels were also high, and would alert groups sensitive to ozone or particle pollution about how to protect their health.

Equipped with that background, we can start looking at the data.

**Analysis**:

The EPA has made daily AQI data across the U.S. available for 1980 to today. 2019's data isn't complete just yet, so I pulled data for 1980 to 2018.


```python
#loading packages
import pandas as pd
import requests
import zipfile
import glob
import io

#pulling annual AQI data by county for 1980 to 2018, unzipping and placing the csv files in a folder on my desktop
for i in range(1980, 2019):
    zip_file_url = 'https://aqs.epa.gov/aqsweb/airdata/annual_aqi_by_county_' + str(i) + '.zip'
    r = requests.get(zip_file_url, stream=True)
    z = zipfile.ZipFile(io.BytesIO(r.content))
    z.extractall(r'C:\Users\Pat\Desktop\zip_files')


#combining all csv files into one dataframe, ignoring column headers after the first file
path = r'C:\Users\Pat\Desktop\zip_files'
all_files = glob.glob(path + "/*.csv")

li = []

for filename in all_files:
    df = pd.read_csv(filename, index_col=None, header=0)
    li.append(df)

all_counties_df = pd.concat(li, axis=0, ignore_index=True)


#filtering the datafrarme for Salt Lake county only
df_SLC = all_counties_df[all_counties_df.County == 'Salt Lake']

df_SLC.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State</th>
      <th>County</th>
      <th>Year</th>
      <th>Days with AQI</th>
      <th>Good Days</th>
      <th>Moderate Days</th>
      <th>Unhealthy for Sensitive Groups Days</th>
      <th>Unhealthy Days</th>
      <th>Very Unhealthy Days</th>
      <th>Hazardous Days</th>
      <th>Max AQI</th>
      <th>90th Percentile AQI</th>
      <th>Median AQI</th>
      <th>Days CO</th>
      <th>Days NO2</th>
      <th>Days Ozone</th>
      <th>Days SO2</th>
      <th>Days PM2.5</th>
      <th>Days PM10</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>522</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>1980</td>
      <td>366</td>
      <td>46</td>
      <td>96</td>
      <td>147</td>
      <td>71</td>
      <td>6</td>
      <td>0</td>
      <td>238</td>
      <td>179</td>
      <td>112</td>
      <td>13</td>
      <td>17</td>
      <td>129</td>
      <td>207</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1133</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>1981</td>
      <td>365</td>
      <td>3</td>
      <td>32</td>
      <td>192</td>
      <td>138</td>
      <td>0</td>
      <td>0</td>
      <td>200</td>
      <td>200</td>
      <td>139</td>
      <td>5</td>
      <td>2</td>
      <td>11</td>
      <td>347</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1734</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>1982</td>
      <td>365</td>
      <td>3</td>
      <td>55</td>
      <td>209</td>
      <td>98</td>
      <td>0</td>
      <td>0</td>
      <td>200</td>
      <td>178</td>
      <td>125</td>
      <td>9</td>
      <td>0</td>
      <td>15</td>
      <td>341</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2344</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>1983</td>
      <td>365</td>
      <td>42</td>
      <td>121</td>
      <td>169</td>
      <td>33</td>
      <td>0</td>
      <td>0</td>
      <td>200</td>
      <td>145</td>
      <td>103</td>
      <td>30</td>
      <td>17</td>
      <td>34</td>
      <td>284</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2928</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>1984</td>
      <td>366</td>
      <td>27</td>
      <td>176</td>
      <td>149</td>
      <td>12</td>
      <td>2</td>
      <td>0</td>
      <td>209</td>
      <td>134</td>
      <td>94</td>
      <td>18</td>
      <td>31</td>
      <td>46</td>
      <td>271</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



We now have a dataframe with annual AQI data for Salt Lake county from 1980 to 2018. It includes:
 - the AQI categorical rating for each day of each year
 - the primary pollutant (pollutant with the highest AQI) for each day of each year
 - median AQI for each year

To visualize how air quality has changed over time, we'll graph all three aspects of the dataframe noted above. This can be done by splitting the original dataframe up into three separate ones:


```python
df_AQI = df_SLC[['Year', 'Good Days', 'Moderate Days', 'Unhealthy for Sensitive Groups Days', 'Unhealthy Days', 'Very Unhealthy Days', 'Hazardous Days']]
df_pollutant = df_SLC[['Year', 'Days NO2', 'Days Ozone', 'Days SO2', 'Days PM2.5', 'Days PM10']]
df_median_AQI = df_SLC[['Year', 'Median AQI']]
```


```python
df_AQI.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Good Days</th>
      <th>Moderate Days</th>
      <th>Unhealthy for Sensitive Groups Days</th>
      <th>Unhealthy Days</th>
      <th>Very Unhealthy Days</th>
      <th>Hazardous Days</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>522</td>
      <td>1980</td>
      <td>46</td>
      <td>96</td>
      <td>147</td>
      <td>71</td>
      <td>6</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1133</td>
      <td>1981</td>
      <td>3</td>
      <td>32</td>
      <td>192</td>
      <td>138</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1734</td>
      <td>1982</td>
      <td>3</td>
      <td>55</td>
      <td>209</td>
      <td>98</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2344</td>
      <td>1983</td>
      <td>42</td>
      <td>121</td>
      <td>169</td>
      <td>33</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2928</td>
      <td>1984</td>
      <td>27</td>
      <td>176</td>
      <td>149</td>
      <td>12</td>
      <td>2</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_pollutant.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Days NO2</th>
      <th>Days Ozone</th>
      <th>Days SO2</th>
      <th>Days PM2.5</th>
      <th>Days PM10</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>522</td>
      <td>1980</td>
      <td>17</td>
      <td>129</td>
      <td>207</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1133</td>
      <td>1981</td>
      <td>2</td>
      <td>11</td>
      <td>347</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>1734</td>
      <td>1982</td>
      <td>0</td>
      <td>15</td>
      <td>341</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2344</td>
      <td>1983</td>
      <td>17</td>
      <td>34</td>
      <td>284</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>2928</td>
      <td>1984</td>
      <td>31</td>
      <td>46</td>
      <td>271</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_median_AQI.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Median AQI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>522</td>
      <td>1980</td>
      <td>112</td>
    </tr>
    <tr>
      <td>1133</td>
      <td>1981</td>
      <td>139</td>
    </tr>
    <tr>
      <td>1734</td>
      <td>1982</td>
      <td>125</td>
    </tr>
    <tr>
      <td>2344</td>
      <td>1983</td>
      <td>103</td>
    </tr>
    <tr>
      <td>2928</td>
      <td>1984</td>
      <td>94</td>
    </tr>
  </tbody>
</table>
</div>



The dataframes look good, let's graph things:


```python
import matplotlib.pyplot as plt

fig = df_AQI.plot.area(x='Year', color = ['green', 'yellow','orange', 'red', 'purple', 'black'])
fig.legend(loc ='upper right',frameon=True, bbox_to_anchor=(1.75, 0.7))
plt.ylabel('Days')
plt.savefig(r'C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\assets\images\SLC_Air_Quality\rating_by_year.png', bbox_inches='tight')
#plt.show()
```


![png](output_8_0.png)


<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/rating_by_year.png" alt="Plot of AQI by year from 1980 to 2018">

</center>

<p style="text-align: center; font-style: italic;"> Plot of SLC's Daily AQI from 1980 to 2018. </p>

From 1980 to 2018, there has been a fairly dramatic shift towards improved air quality, as signified by an increase in "Good" and "Moderate" days and a decrease in "Unhealthy" ones. 1982 would've been an especially good year to hold your breath.

However, there does seem to be a decrease in the number of "Good" air quality days in recent years. Looking specifically at that portion of the graph, things become more clear:


```python
df_AQI_recent = df_AQI[df_AQI.Year > 2013]

fig = df_AQI_recent.plot.area(x='Year', color = ['green', 'yellow','orange', 'red', 'purple', 'black'])
fig.legend(loc ='upper right',frameon=True, bbox_to_anchor=(1.75, 0.7))
plt.xticks([2014, 2015, 2016, 2017, 2018])
plt.ylabel('Days')
plt.savefig(r'C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\assets\images\SLC_Air_Quality\recent_AQI.png', bbox_inches='tight')
```


![png](output_10_0.png)


<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/recent_AQI_2.png" alt="Plot of AQI by year from 2014 to 2018">

</center>

<p style="text-align: center; font-style: italic;"> Plot of SLC's Daily AQI from 2014 to 2018. </p>

Since 2014, Salt Lake City has seen a decrease in good air quality days. In their place we've had more "Moderate"
and "Unhealthy for Sensitive Groups" days. This is further evidenced by plotting median AQI per year - great progress was made from 1980 to 2000, at which point progress stalled (and reversed in recent years):


```python
fig = df_median_AQI.plot.area(x='Year', stacked=True, legend = False)
plt.axhline(y=50, linewidth=1, linestyle='-.', color='r')
plt.ylabel('Median AQI')
#plt.savefig(r'C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\assets\images\SLC_Air_Quality\median_AQI.png', bbox_inches='tight')
```


![png](output_12_0.png)


<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/median_AQI.png" alt="Plot of median AQI per year from 1980 to 2018">

</center>

<p style="text-align: center; font-style: italic;"> Plot of SLC's Median AQI per year from 1980 to 2018. </p>

The reference line is drawn at AQI=50. You might recall, an AQI below 50 signifies "Good" air quality, while an AQI between 50 and 100 signifies "Moderate" air quality. Which pollutants are contributing to the recent increase in median AQI?


```python
fig = df_pollutant.plot.area(x='Year', stacked=True)
fig.legend(loc ='upper right',frameon=True, bbox_to_anchor=(1.4, 0.7))
plt.ylabel('Days')
plt.savefig(r'C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\assets\images\SLC_Air_Quality\pollutant_mix.png', bbox_inches='tight')
```


![png](output_14_0.png)


<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/pollutant_mix.png" alt="Plot of largest AQI pollutant per year from 1980 to 2018">

</center>

<p style="text-align: center; font-style: italic;"> Plot of highest AQI pollutant per day from 1980 to 2018. </p>

The plot above shows which pollutant is the primary contributor to poor air quality per day (which of the 5 has the highest AQI), from 1980 to 2018.

Sulfur dioxide was the dominant pollutant from 1980 to 1995, at which point SO2 was greatly reduced and ozone surged. Today ozone is the biggeset contributor to poor air quality, and PM2.5 has also had a concerning rise since 2000.

What causes these pollutants? How harmful are they? Why are they increasing? How can we get cleaner air in the valley?

If you're reading this, I'm not finished yet! (01 Mar 2020)
