---
title: "Air Quality in Salt Lake City"
date: 2020-02-17
tags: [Python, Pandas, Data Visualization]
header:
  image: #"/image/"
excerpt: "Using Python to collect and explore EPA air quality data."
---

**Abstract**:

Here I investigate how Salt Lake City's air quality has changed over time by downloading 38 years worth of data from the EPA, aggregating this information into one dataframe with pandas, then exploring the data visually with matplotlib.

**Intro**:

As a child, I remember watching my dad clean the fish tank. This was a bit of a monthly ritual, as over that length of time it would go from clear to cloudy to “we should not be pet owners” dirty. I always felt bad for the fish in the days preceding this cleaning – when they were clearly bopping around in some nasty stuff:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/dirty_tank_compressed.jpg " alt="dirty fish tank">

</center>

<p style="text-align: center; font-style: italic;"> Not the fish tank of my childhood, but similar. </p>

Household fish aren’t the only ones subjected to such conditions! Many of us live in areas with dirty air. Here’s a map of PM2.5 pollution (tiny particles that damage our lungs) in the U.S.:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/us_air_compressed.jpg " alt="U.S. map of PM2.5 air pollution", class="full">

</center>

<p style="text-align: center; font-style: italic;"> PM2.5 air pollution across the U.S. </p>

The blotch of red towards the middle-left of the country is Salt Lake City, where due to the perimeter of mountains surrounding the inhabited valley floor (forming a bowl-like geometry), nasty air can get trapped and accumulate. This is especially bad in the winter, when a blanket of warm air forms a “lid” on top of the colder valley floor – this is called the *winter inversion*, which is a spooky name.

I had heard about this prior to moving to SLC. But this winter I really haven’t seen a cause for concern. Admittedly, I don’t check air quality often, but if it was really that bad I think I’d have heard? That all said, I was curious to see what the air quality in SLC is today and how it has changed over time.

Poking around, I came across an article in DeseretNews titled “Visualizing SLC air pollution in 35 years and what it tells us.” The article, ([here])( https://www.deseret.com/2015/5/7/20564270/visualizing-slc-air-pollution-in-35-years-and-what-it-tells-us), links to a web-based visualization which unfortunately returns “site can’t be reached.” I sent the article’s author a note to let her know, and in the meantime decided to see if I could pull air quality data and visualize it myself.

The EPA makes air quality data available to the public via their AirData Quality Monitors web app. Below are all the monitors across the U.S. for the 5 primary pollutants used in evaluating air quality (carbon monoxide, nitrogen dioxide, ozone, PM2.5, and sulfur dioxide):

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/US_monitors_reduced.png" alt="EPA air quality monitors in the U.S.">

</center>

<p style="text-align: center; font-style: italic;"> Air quality monitors in the U.S. </p>

Zooming in on SLC, you can see the valley has 5 monitors:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/valley_monitors_reduced.png" alt="Air quality monitors in Salt Lake City">

</center>

<p style="text-align: center; font-style: italic;"> Air quality monitors in Salt Lake City. </p>

The EPA uses data from these monitors to calculate an Air Quality Index (known as AQI), which has six categories:

<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/AQI_table_reduced_2.png" alt="AQI table: 0-50=good, 51-100=moderate,101-150=unhealthy for sensitive groups, 151-200=unhealthy, 210-300=very unhealthy, hazardous=301-500">

</center>

<p style="text-align: center; font-style: italic;"> The Air Quality Index (AQI) categories. </p>


((improve from below this))
From the EPA:
"EPA calculates the AQI for five major air pollutants regulated by the Clean Air Act:

- ground-level ozone
- particle pollution (also known as particulate matter)
- carbon monoxide
- sulfur dioxide
- nitrogen dioxide

For each of these pollutants, EPA has established national air quality standards to protect public health .Ground-level ozone and airborne particles are the two pollutants that pose the greatest threat to human health in this country."

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



We now have a dataframe with annual AQI data for Salt Lake county from 1980 to 2018. It includes the AQI categorical rating for each day of each year, as well as the primary pollutant for each day of each year.

To visualize how air quality has changed over time, we'll graph the AQI categories and primary pollutant separately. Splitting these up into two dataframes:


```python
df_AQI = df_SLC[['Year', 'Good Days', 'Moderate Days', 'Unhealthy for Sensitive Groups Days', 'Unhealthy Days', 'Very Unhealthy Days', 'Hazardous Days']]
df_pollutant = df_SLC[['Year', 'Days NO2', 'Days Ozone', 'Days SO2', 'Days PM2.5', 'Days PM10']]
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



The dataframes look good, let's graph things:


```python
import matplotlib.pyplot as plt

fig = df_AQI.plot.area(x='Year', color = ['green', 'yellow','orange', 'red', 'purple', 'black'])
fig.legend(loc ='upper right',frameon=True, bbox_to_anchor=(1.75, 0.7))
plt.ylabel('Days')
#plt.savefig(r'C:\Users\Pat\Desktop\Patrick-DeBiasse.github.io\assets\images\SLC_Air_Quality\test_4.svg', format='svg', dpi=1200, bbox_inches='tight')
plt.show()
```


![png](output_7_0.png)


<center>

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/SLC_Air_Quality/test_5.png" alt="Plot of AQI by year from 1980 to 2018">

</center>

<p style="text-align: center; font-style: italic;"> Plot of SLC's Daily AQI from 1980 to 2018. </p>

From 1980 to 2018, there has been a fairly dramatic shift towards improved air quality, as signified by an increase in "Good" and "Moderate" days and a decrease in "unhealthy" ones. 1982 would've been an especially good year to hold your breath.

However, there does seem to be a decrease in the number of good air quality days in recent years. Looking specifically at that portion of the graph things become more clear:


```python
df_AQI_recent = df_AQI[df_AQI.Year > 2013]
df_AQI_recent
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
      <td>33087</td>
      <td>2014</td>
      <td>220</td>
      <td>123</td>
      <td>20</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>34147</td>
      <td>2015</td>
      <td>180</td>
      <td>158</td>
      <td>25</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>35204</td>
      <td>2016</td>
      <td>189</td>
      <td>150</td>
      <td>23</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>36263</td>
      <td>2017</td>
      <td>149</td>
      <td>172</td>
      <td>43</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <td>37318</td>
      <td>2018</td>
      <td>146</td>
      <td>178</td>
      <td>39</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
ax = df_AQI_recent.plot.area(x='Year', color = ['green', 'yellow','orange', 'red', 'purple', 'black'])
ax.legend(loc ='upper right',frameon=True, bbox_to_anchor=(1.75, 0.7))
```




    <matplotlib.legend.Legend at 0x117b2f10848>




![png](output_10_1.png)


Since 2014, Salt Lake City has seen a decrease in good air quality days. In their place we've had more "moderate"
and "unhealthy for sensitive groups" days. What pollutants are contributing to this slide?


```python
ax = df_pollutant.plot.area(x='Year', stacked=True)
ax.legend(loc ='upper right',frameon=True, bbox_to_anchor=(1.4, 0.7))
```




    <matplotlib.legend.Legend at 0x117b47878c8>




![png](output_12_1.png)


Sulfur dioxide was the dominant pollutant from 1980 to 1995, at which point ozone surged. Today ozone is the biggeset contributor to poor air quality. PM2.5 has also had a concerning rise since 2000.

What causes these pollutants? How harmful are they? How can we get cleaner air in the valley?

**Discussion**:

1) What causes these pollutants?
Ozone is a

increase in SLC population requires more energy, more cars, more infrastructure

**Conclusion**:

discuss most promising ways to improve air quality

if you coach youth sports, you consider canceling practice if the AQI goes above 100.


SO2 went from the primary pollutant up to 1995, when ozone overtook it. As of 2018 ozone is far and away the largest contributor to poor air quality in Salt Lake City.Is anything being done about it?  

https://www.slc.gov/sustainability/air-quality/


https://deq.utah.gov/air-quality/what-is-ozone



**Bonus**:

(Not really a bonus, just me investigating what data is collected by your avergae air quality monitor).

Earlier, I mentioned that the EPA uses data from monitors spread across the United States to evaluate air quality. There are more than 4,000 of these monitors! I was curious to see what data comes off of these - none of the work below changed the above conclusions, but if you're curious as well, let's take a look:

Data from these monitors is available on the EPA's [Air Quality Service site](https://aqs.epa.gov/aqsweb/airdata/download_files.html). I queried the [AQS data API]( https://aqs.epa.gov/aqsweb/documents/data_api.html#param) - first requesting metadata (operational information on each monitor). This query returned data in a JSON file format:


```python
import json

with open(r'C:\Users\Pat\Desktop\JSON air quality data\active stations\byCounty - active stations.json') as json_data:
    data = json.load(json_data)

print(data)
```

    {'Header': [{'status': 'Success', 'request_time': '2020-02-21T11:15:27-05:00', 'url': 'https://aqs.epa.gov/data/api/monitors/byCounty?email=patrick.debiasse@gmail.com&key=khakihawk63&param=44201&bdate=20090501&edate=20190502&state=49&county=035', 'rows': 9}], 'Data': [{'state_code': '49', 'county_code': '035', 'site_number': '0015', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '2014-03-12', 'close_date': None, 'concurred_exclusions': 'All (2014-03-12 - Present)', 'dominant_source': None, 'measurement_scale': None, 'measurement_scale_def': None, 'monitoring_objective': 'GENERAL/BACKGROUND', 'last_method_code': '190', 'last_method_description': 'Instrumental - UV absorption photometry/UV 2B model 202 and 205', 'last_method_begin_date': '2014-03-12', 'naaqs_primary_monitor': 'Y', 'qa_primary_monitor': None, 'monitor_type': 'NON-EPA FEDERAL', 'networks': None, 'monitoring_agency_code': '1110', 'monitoring_agency': 'US Forest Service', 'si_id': 100183, 'latitude': 40.569, 'longitude': -111.659, 'datum': 'WGS84', 'lat_lon_accuracy': 15.0, 'elevation': 2940.0, 'probe_height': None, 'pl_probe_location': None, 'local_site_name': 'Snowbird', 'address': 'Snowbird', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Not in a City', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '4002', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '2019-01-01', 'close_date': None, 'concurred_exclusions': None, 'dominant_source': None, 'measurement_scale': None, 'measurement_scale_def': None, 'monitoring_objective': 'HIGHEST CONCENTRATION', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2019-01-01', 'naaqs_primary_monitor': 'Y', 'qa_primary_monitor': None, 'monitor_type': 'SPM', 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 104474, 'latitude': 40.662878, 'longitude': -111.901188, 'datum': 'NAD83', 'lat_lon_accuracy': 1.0, 'elevation': 1295.0, 'probe_height': None, 'pl_probe_location': None, 'local_site_name': None, 'address': '4951 South Galleria Dr', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Murray', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '2005', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '2018-04-18', 'close_date': None, 'concurred_exclusions': None, 'dominant_source': None, 'measurement_scale': None, 'measurement_scale_def': None, 'monitoring_objective': 'POPULATION EXPOSURE', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2018-04-18', 'naaqs_primary_monitor': 'Y', 'qa_primary_monitor': None, 'monitor_type': 'SLAMS', 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 104256, 'latitude': 40.598056, 'longitude': -111.894167, 'datum': 'NAD83', 'lat_lon_accuracy': 1.0, 'elevation': 1.0, 'probe_height': None, 'pl_probe_location': None, 'local_site_name': None, 'address': '8449 S. Monroe St.', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Midvale', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '3010', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '2018-01-01', 'close_date': None, 'concurred_exclusions': None, 'dominant_source': None, 'measurement_scale': 'NEIGHBORHOOD', 'measurement_scale_def': '500 M TO 4KM', 'monitoring_objective': 'POPULATION EXPOSURE', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2018-01-01', 'naaqs_primary_monitor': 'Y', 'qa_primary_monitor': None, 'monitor_type': 'SLAMS', 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 93647, 'latitude': 40.78422, 'longitude': -111.931, 'datum': 'WGS84', 'lat_lon_accuracy': 20.0, 'elevation': 1286.0, 'probe_height': 4.0, 'pl_probe_location': 'TOP OF BUILDING', 'local_site_name': 'ROSE PARK', 'address': '1250 NORTH 1400 WEST', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '3013', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '2015-01-30', 'close_date': None, 'concurred_exclusions': None, 'dominant_source': None, 'measurement_scale': None, 'measurement_scale_def': None, 'monitoring_objective': 'GENERAL/BACKGROUND', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2015-01-30', 'naaqs_primary_monitor': 'Y', 'qa_primary_monitor': None, 'monitor_type': 'SLAMS', 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 99160, 'latitude': 40.496392, 'longitude': -112.036298, 'datum': 'WGS84', 'lat_lon_accuracy': 1.0, 'elevation': 1.0, 'probe_height': None, 'pl_probe_location': None, 'local_site_name': None, 'address': '14058 Mirabella Dr.', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Herriman', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '0003', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '1980-12-01', 'close_date': '2011-10-02', 'concurred_exclusions': None, 'dominant_source': 'AREA', 'measurement_scale': 'NEIGHBORHOOD', 'measurement_scale_def': '500 M TO 4KM', 'monitoring_objective': 'POPULATION EXPOSURE', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2004-07-20', 'naaqs_primary_monitor': None, 'qa_primary_monitor': None, 'monitor_type': 'SLAMS', 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 15814, 'latitude': 40.646667, 'longitude': -111.849722, 'datum': 'WGS84', 'lat_lon_accuracy': 24.29, 'elevation': 1335.0, 'probe_height': 4.0, 'pl_probe_location': 'TOP OF BUILDING', 'local_site_name': 'Cottonwood', 'address': '5715 S. 1400 E., SALT LAKE CITY', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Cottonwood West', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '1007', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '2019-01-01', 'close_date': '2019-12-31', 'concurred_exclusions': None, 'dominant_source': None, 'measurement_scale': None, 'measurement_scale_def': None, 'monitoring_objective': 'GENERAL/BACKGROUND', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2019-01-01', 'naaqs_primary_monitor': None, 'qa_primary_monitor': None, 'monitor_type': None, 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 104476, 'latitude': 40.712146, 'longitude': -112.111275, 'datum': 'NAD83', 'lat_lon_accuracy': 1.0, 'elevation': 1.0, 'probe_height': None, 'pl_probe_location': None, 'local_site_name': None, 'address': '9228 West 2700 South', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Magna', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '2004', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '1994-05-17', 'close_date': '2014-09-30', 'concurred_exclusions': None, 'dominant_source': 'AREA', 'measurement_scale': 'URBAN SCALE', 'measurement_scale_def': '4 KM TO 50 KM', 'monitoring_objective': 'HIGHEST CONCENTRATION', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2001-05-01', 'naaqs_primary_monitor': None, 'qa_primary_monitor': None, 'monitor_type': None, 'networks': None, 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 15839, 'latitude': 40.736389, 'longitude': -112.210278, 'datum': 'WGS84', 'lat_lon_accuracy': 24.29, 'elevation': 1284.0, 'probe_height': 4.0, 'pl_probe_location': 'TOP OF BUILDING', 'local_site_name': 'UTM COORDINATES AT PROBE LOCATION', 'address': '12100 W 1200 S, LAKEPOINT, UTAH', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Not in a city', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'parameter_name': 'Ozone', 'open_date': '1997-05-01', 'close_date': None, 'concurred_exclusions': None, 'dominant_source': 'AREA', 'measurement_scale': 'NEIGHBORHOOD', 'measurement_scale_def': '500 M TO 4KM', 'monitoring_objective': 'UNKNOWN', 'last_method_code': '087', 'last_method_description': 'INSTRUMENTAL - ULTRA VIOLET ABSORPTION', 'last_method_begin_date': '2003-07-23', 'naaqs_primary_monitor': 'Y', 'qa_primary_monitor': None, 'monitor_type': 'SLAMS', 'networks': 'NCORE', 'monitoring_agency_code': '1113', 'monitoring_agency': 'Utah Department Of Environmental Quality', 'si_id': 15845, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'lat_lon_accuracy': 24.29, 'elevation': 1306.0, 'probe_height': 4.0, 'pl_probe_location': 'TOP OF BUILDING', 'local_site_name': 'Hawthorne', 'address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state_name': 'Utah', 'county_name': 'Salt Lake', 'city_name': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa_name': 'Salt Lake City, UT', 'csa_code': '482', 'csa_name': 'Salt Lake City-Provo-Orem, UT', 'tribal_code': None, 'tribe_name': None}]}


The JSON data was formatted with 2 dictionaries ('Header' and 'Data'), each of which contained a list of key-value pairs. I converted the JSON data within the 'Data' dictionary into a pandas dataframe so it'd be easier to read:


```python
from pandas.io.json import json_normalize

df = json_normalize(data['Data'])

df_2 = df[df['close_date'] == 'None']
df_2
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
      <th>state_code</th>
      <th>county_code</th>
      <th>site_number</th>
      <th>parameter_code</th>
      <th>poc</th>
      <th>parameter_name</th>
      <th>open_date</th>
      <th>close_date</th>
      <th>concurred_exclusions</th>
      <th>dominant_source</th>
      <th>...</th>
      <th>address</th>
      <th>state_name</th>
      <th>county_name</th>
      <th>city_name</th>
      <th>cbsa_code</th>
      <th>cbsa_name</th>
      <th>csa_code</th>
      <th>csa_name</th>
      <th>tribal_code</th>
      <th>tribe_name</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
<p>0 rows × 41 columns</p>
</div>



Now equipped with site numebrs, I queried the EPA's Air Quality System API again, this time requesting annual summary data from site 3006 (no real reason for picking this one other than it's currently active and sort of near my house). In order to automate this a bit more, I used the requests library:


```python
#defining API parameters
parameters = {
    "email": "patrick.debiasse@gmail.com",
    "key": "khakihawk63",
    "param": "88101,44201,42602,42101,42401", #these are codes for the 5 primary pollutants used to assess air quality
    "bdate": "20001201",
    "edate": "20001202",
    "state": "49",
    "county": "035",
    "site": "3006"
}

#requesting and printing the JSON data
json_data = requests.get("https://aqs.epa.gov/data/api/annualData/bySite?email=test@aqs.api&key=test&param=44201&bdate=20170618&edate=20170618&state=37&county=183&site=0014", params=parameters).json()
print(json_data)
```

    {'Header': [{'status': 'Success', 'request_time': '2020-02-29T20:18:31-05:00', 'url': 'https://aqs.epa.gov/data/api/annualData/bySite?email=test@aqs.api&key=test&param=44201&bdate=20170618&edate=20170618&state=37&county=183&site=0014&email=patrick.debiasse%40gmail.com&key=khakihawk63&param=88101%2C44201%2C42602%2C42101%2C42401&bdate=20001201&edate=20001202&state=49&county=035&site=3006', 'rows': 20}], 'Data': [{'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '42101', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Carbon monoxide', 'sample_duration': '1 HOUR', 'pollutant_standard': 'CO 1-hour 1971', 'metric_used': 'Obseved hourly values', 'method': 'INSTRUMENTAL - NONDISPERSIVE INFRARED', 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'No Events', 'observation_count': 3554, 'observation_percent': 40.0, 'validity_indicator': 'Y', 'valid_day_count': 151, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 94, 'primary_exceedance_count': 0, 'secondary_exceedance_count': 0, 'certification_indicator': 'Certified', 'arithmetic_mean': 1.422651, 'standard_deviation': 1.313468, 'first_max_value': 10.4, 'first_max_datetime': '2000-02-03 08:00', 'second_max_value': 10.0, 'second_max_datetime': '2000-12-08 08:00', 'third_max_value': 9.9, 'third_max_datetime': '2000-12-04 08:00', 'fourth_max_value': 9.6, 'fourth_max_datetime': '2000-02-08 08:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 6.4, 'ninety_eighth_percentile': 5.3, 'ninety_fifth_percentile': 4.3, 'ninetieth_percentile': 3.0, 'seventy_fifth_percentile': 1.9, 'fiftieth_percentile': 1.0, 'tenth_percentile': 0.4, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-06-05'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '42101', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Carbon monoxide', 'sample_duration': '8-HR RUN AVG END HOUR', 'pollutant_standard': 'CO 8-hour 1971', 'metric_used': '8-Hour running average (end hour) of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'No Events', 'observation_count': 3579, 'observation_percent': 41.0, 'validity_indicator': 'Y', 'valid_day_count': 152, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 0, 'secondary_exceedance_count': 0, 'certification_indicator': 'Certified', 'arithmetic_mean': 1.427717, 'standard_deviation': 1.014632, 'first_max_value': 5.2, 'first_max_datetime': '2000-11-28 00:00', 'second_max_value': 5.1, 'second_max_datetime': '2000-11-28 01:00', 'third_max_value': 5.0, 'third_max_datetime': '2000-11-27 23:00', 'fourth_max_value': 4.9, 'fourth_max_datetime': '2000-02-03 09:00', 'first_max_nonoverlap_value': 5.2, 'first_max_n_o_datetime': '2000-11-28 00:00', 'second_max_nonoverlap_value': 4.9, 'second_max_n_o_datetime': '2000-02-03 09:00', 'ninety_ninth_percentile': 4.5, 'ninety_eighth_percentile': 4.2, 'ninety_fifth_percentile': 3.7, 'ninetieth_percentile': 3.0, 'seventy_fifth_percentile': 1.9, 'fiftieth_percentile': 1.1, 'tenth_percentile': 0.5, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-06-05'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '42602', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Nitrogen dioxide (NO2)', 'sample_duration': '1 HOUR', 'pollutant_standard': 'NO2 Annual 1971', 'metric_used': 'Observed values', 'method': 'INSTRUMENTAL - CHEMILUMINESCENCE', 'year': 2000, 'units_of_measure': 'Parts per billion', 'event_type': 'No Events', 'observation_count': 8593, 'observation_percent': 98.0, 'validity_indicator': 'Y', 'valid_day_count': 362, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 191, 'primary_exceedance_count': None, 'secondary_exceedance_count': None, 'certification_indicator': 'Certified', 'arithmetic_mean': 26.476667, 'standard_deviation': 15.436433, 'first_max_value': 119.0, 'first_max_datetime': '2000-12-30 11:00', 'second_max_value': 102.0, 'second_max_datetime': '2000-11-22 11:00', 'third_max_value': 101.0, 'third_max_datetime': '2000-12-30 10:00', 'fourth_max_value': 97.0, 'fourth_max_datetime': '2000-12-20 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 65.0, 'ninety_eighth_percentile': 59.0, 'ninety_fifth_percentile': 52.0, 'ninetieth_percentile': 46.0, 'seventy_fifth_percentile': 38.0, 'fiftieth_percentile': 25.0, 'tenth_percentile': 7.0, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2017-05-19'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '42602', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Nitrogen dioxide (NO2)', 'sample_duration': '1 HOUR', 'pollutant_standard': 'NO2 1-hour', 'metric_used': 'Daily Maximum 1-hour average', 'method': 'INSTRUMENTAL - CHEMILUMINESCENCE', 'year': 2000, 'units_of_measure': 'Parts per billion', 'event_type': 'No Events', 'observation_count': 8593, 'observation_percent': 98.0, 'validity_indicator': 'Y', 'valid_day_count': 362, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 191, 'primary_exceedance_count': 2, 'secondary_exceedance_count': None, 'certification_indicator': 'Certified', 'arithmetic_mean': 46.29558, 'standard_deviation': 12.53752, 'first_max_value': 119.0, 'first_max_datetime': '2000-12-30 11:00', 'second_max_value': 102.0, 'second_max_datetime': '2000-11-22 11:00', 'third_max_value': 97.0, 'third_max_datetime': '2000-12-20 10:00', 'fourth_max_value': 97.0, 'fourth_max_datetime': '2000-12-29 13:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 97.0, 'ninety_eighth_percentile': 79.0, 'ninety_fifth_percentile': 65.0, 'ninetieth_percentile': 61.0, 'seventy_fifth_percentile': 51.0, 'fiftieth_percentile': 46.0, 'tenth_percentile': 32.0, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2017-05-19'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '1 HOUR', 'pollutant_standard': 'Ozone 1-hour 1979', 'metric_used': 'Daily maxima of observed hourly values (between 9:00 AM and 8:00 PM)', 'method': 'INSTRUMENTAL - ULTRA VIOLET', 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Excluded', 'observation_count': 3423, 'observation_percent': 99.0, 'validity_indicator': 'Y', 'valid_day_count': 151, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 82, 'primary_exceedance_count': 0, 'secondary_exceedance_count': 0, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.059729, 'standard_deviation': 0.014052, 'first_max_value': 0.112, 'first_max_datetime': '2000-08-12 14:00', 'second_max_value': 0.095, 'second_max_datetime': '2000-07-25 15:00', 'third_max_value': 0.088, 'third_max_datetime': '2000-08-07 15:00', 'fourth_max_value': 0.086, 'fourth_max_datetime': '2000-08-06 13:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.095, 'ninety_eighth_percentile': 0.086, 'ninety_fifth_percentile': 0.078, 'ninetieth_percentile': 0.075, 'seventy_fifth_percentile': 0.069, 'fiftieth_percentile': 0.059, 'tenth_percentile': 0.037, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '1 HOUR', 'pollutant_standard': 'Ozone 1-hour 1979', 'metric_used': 'Daily maxima of observed hourly values (between 9:00 AM and 8:00 PM)', 'method': 'INSTRUMENTAL - ULTRA VIOLET', 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Inclucded', 'observation_count': 3590, 'observation_percent': 99.0, 'validity_indicator': 'Y', 'valid_day_count': 151, 'required_day_count': 153, 'exceptional_data_count': 167, 'null_observation_count': 82, 'primary_exceedance_count': 2, 'secondary_exceedance_count': 2, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.062119, 'standard_deviation': 0.017838, 'first_max_value': 0.134, 'first_max_datetime': '2000-08-02 15:00', 'second_max_value': 0.133, 'second_max_datetime': '2000-08-01 14:00', 'third_max_value': 0.116, 'third_max_datetime': '2000-07-30 13:00', 'fourth_max_value': 0.112, 'fourth_max_datetime': '2000-08-12 14:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.133, 'ninety_eighth_percentile': 0.112, 'ninety_fifth_percentile': 0.094, 'ninetieth_percentile': 0.078, 'seventy_fifth_percentile': 0.071, 'fiftieth_percentile': 0.061, 'tenth_percentile': 0.043, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '1 HOUR', 'pollutant_standard': 'Ozone 1-hour 1979', 'metric_used': 'Daily maxima of observed hourly values (between 9:00 AM and 8:00 PM)', 'method': 'INSTRUMENTAL - ULTRA VIOLET', 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Concurred Events Excluded', 'observation_count': 3423, 'observation_percent': 99.0, 'validity_indicator': 'Y', 'valid_day_count': 151, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 82, 'primary_exceedance_count': 0, 'secondary_exceedance_count': 0, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.059729, 'standard_deviation': 0.014052, 'first_max_value': 0.112, 'first_max_datetime': '2000-08-12 14:00', 'second_max_value': 0.095, 'second_max_datetime': '2000-07-25 15:00', 'third_max_value': 0.088, 'third_max_datetime': '2000-08-07 15:00', 'fourth_max_value': 0.086, 'fourth_max_datetime': '2000-08-06 13:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.095, 'ninety_eighth_percentile': 0.086, 'ninety_fifth_percentile': 0.078, 'ninetieth_percentile': 0.075, 'seventy_fifth_percentile': 0.069, 'fiftieth_percentile': 0.059, 'tenth_percentile': 0.037, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-Hour 1997', 'metric_used': 'Daily maximum of 8 hour running average of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Excluded', 'observation_count': 3458, 'observation_percent': 97.0, 'validity_indicator': 'Y', 'valid_day_count': 149, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 0, 'secondary_exceedance_count': 0, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.050699, 'standard_deviation': 0.011961, 'first_max_value': 0.082, 'first_max_datetime': '2000-08-12 11:00', 'second_max_value': 0.075, 'second_max_datetime': '2000-06-04 11:00', 'third_max_value': 0.072, 'third_max_datetime': '2000-07-25 12:00', 'fourth_max_value': 0.072, 'fourth_max_datetime': '2000-08-06 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.075, 'ninety_eighth_percentile': 0.072, 'ninety_fifth_percentile': 0.066, 'ninetieth_percentile': 0.065, 'seventy_fifth_percentile': 0.059, 'fiftieth_percentile': 0.051, 'tenth_percentile': 0.029, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-Hour 2008', 'metric_used': 'Daily maximum of 8 hour running average of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Excluded', 'observation_count': 3458, 'observation_percent': 97.0, 'validity_indicator': 'Y', 'valid_day_count': 149, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 1, 'secondary_exceedance_count': 1, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.050699, 'standard_deviation': 0.011961, 'first_max_value': 0.082, 'first_max_datetime': '2000-08-12 11:00', 'second_max_value': 0.075, 'second_max_datetime': '2000-06-04 11:00', 'third_max_value': 0.072, 'third_max_datetime': '2000-07-25 12:00', 'fourth_max_value': 0.072, 'fourth_max_datetime': '2000-08-06 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.075, 'ninety_eighth_percentile': 0.072, 'ninety_fifth_percentile': 0.066, 'ninetieth_percentile': 0.065, 'seventy_fifth_percentile': 0.059, 'fiftieth_percentile': 0.051, 'tenth_percentile': 0.029, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-hour 2015', 'metric_used': 'Daily maximum of 8-hour running average', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Excluded', 'observation_count': 2454, 'observation_percent': 98.0, 'validity_indicator': 'Y', 'valid_day_count': 150, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 5, 'secondary_exceedance_count': 5, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.050896, 'standard_deviation': 0.012017, 'first_max_value': 0.082, 'first_max_datetime': '2000-08-12 11:00', 'second_max_value': 0.075, 'second_max_datetime': '2000-06-04 11:00', 'third_max_value': 0.072, 'third_max_datetime': '2000-07-25 12:00', 'fourth_max_value': 0.072, 'fourth_max_datetime': '2000-08-06 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.075, 'ninety_eighth_percentile': 0.072, 'ninety_fifth_percentile': 0.066, 'ninetieth_percentile': 0.065, 'seventy_fifth_percentile': 0.06, 'fiftieth_percentile': 0.051, 'tenth_percentile': 0.031, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-Hour 1997', 'metric_used': 'Daily maximum of 8 hour running average of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Inclucded', 'observation_count': 3619, 'observation_percent': 97.0, 'validity_indicator': 'Y', 'valid_day_count': 149, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 5, 'secondary_exceedance_count': 5, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.052356, 'standard_deviation': 0.014049, 'first_max_value': 0.094, 'first_max_datetime': '2000-08-01 11:00', 'second_max_value': 0.093, 'second_max_datetime': '2000-08-02 11:00', 'third_max_value': 0.092, 'third_max_datetime': '2000-07-31 11:00', 'fourth_max_value': 0.088, 'fourth_max_datetime': '2000-07-30 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.093, 'ninety_eighth_percentile': 0.092, 'ninety_fifth_percentile': 0.075, 'ninetieth_percentile': 0.066, 'seventy_fifth_percentile': 0.061, 'fiftieth_percentile': 0.052, 'tenth_percentile': 0.034, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-Hour 2008', 'metric_used': 'Daily maximum of 8 hour running average of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Inclucded', 'observation_count': 3619, 'observation_percent': 97.0, 'validity_indicator': 'Y', 'valid_day_count': 149, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 7, 'secondary_exceedance_count': 7, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.052356, 'standard_deviation': 0.014049, 'first_max_value': 0.094, 'first_max_datetime': '2000-08-01 11:00', 'second_max_value': 0.093, 'second_max_datetime': '2000-08-02 11:00', 'third_max_value': 0.092, 'third_max_datetime': '2000-07-31 11:00', 'fourth_max_value': 0.088, 'fourth_max_datetime': '2000-07-30 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.093, 'ninety_eighth_percentile': 0.092, 'ninety_fifth_percentile': 0.075, 'ninetieth_percentile': 0.066, 'seventy_fifth_percentile': 0.061, 'fiftieth_percentile': 0.052, 'tenth_percentile': 0.034, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-hour 2015', 'metric_used': 'Daily maximum of 8-hour running average', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Events Inclucded', 'observation_count': 2566, 'observation_percent': 98.0, 'validity_indicator': 'Y', 'valid_day_count': 150, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 12, 'secondary_exceedance_count': 12, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.052533, 'standard_deviation': 0.01406, 'first_max_value': 0.094, 'first_max_datetime': '2000-08-01 11:00', 'second_max_value': 0.093, 'second_max_datetime': '2000-08-02 11:00', 'third_max_value': 0.092, 'third_max_datetime': '2000-07-31 11:00', 'fourth_max_value': 0.088, 'fourth_max_datetime': '2000-07-30 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.093, 'ninety_eighth_percentile': 0.092, 'ninety_fifth_percentile': 0.075, 'ninetieth_percentile': 0.066, 'seventy_fifth_percentile': 0.061, 'fiftieth_percentile': 0.053, 'tenth_percentile': 0.035, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-Hour 1997', 'metric_used': 'Daily maximum of 8 hour running average of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Concurred Events Excluded', 'observation_count': 3458, 'observation_percent': 97.0, 'validity_indicator': 'Y', 'valid_day_count': 149, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 0, 'secondary_exceedance_count': 0, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.050699, 'standard_deviation': 0.011961, 'first_max_value': 0.082, 'first_max_datetime': '2000-08-12 11:00', 'second_max_value': 0.075, 'second_max_datetime': '2000-06-04 11:00', 'third_max_value': 0.072, 'third_max_datetime': '2000-07-25 12:00', 'fourth_max_value': 0.072, 'fourth_max_datetime': '2000-08-06 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.075, 'ninety_eighth_percentile': 0.072, 'ninety_fifth_percentile': 0.066, 'ninetieth_percentile': 0.065, 'seventy_fifth_percentile': 0.059, 'fiftieth_percentile': 0.051, 'tenth_percentile': 0.029, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-Hour 2008', 'metric_used': 'Daily maximum of 8 hour running average of observed hourly values', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Concurred Events Excluded', 'observation_count': 3619, 'observation_percent': 97.0, 'validity_indicator': 'Y', 'valid_day_count': 149, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 7, 'secondary_exceedance_count': 7, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.052356, 'standard_deviation': 0.014049, 'first_max_value': 0.094, 'first_max_datetime': '2000-08-01 11:00', 'second_max_value': 0.093, 'second_max_datetime': '2000-08-02 11:00', 'third_max_value': 0.092, 'third_max_datetime': '2000-07-31 11:00', 'fourth_max_value': 0.088, 'fourth_max_datetime': '2000-07-30 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.093, 'ninety_eighth_percentile': 0.092, 'ninety_fifth_percentile': 0.075, 'ninetieth_percentile': 0.066, 'seventy_fifth_percentile': 0.061, 'fiftieth_percentile': 0.052, 'tenth_percentile': 0.034, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '44201', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'Ozone', 'sample_duration': '8-HR RUN AVG BEGIN HOUR', 'pollutant_standard': 'Ozone 8-hour 2015', 'metric_used': 'Daily maximum of 8-hour running average', 'method': None, 'year': 2000, 'units_of_measure': 'Parts per million', 'event_type': 'Concurred Events Excluded', 'observation_count': 2566, 'observation_percent': 98.0, 'validity_indicator': 'Y', 'valid_day_count': 150, 'required_day_count': 153, 'exceptional_data_count': 0, 'null_observation_count': 0, 'primary_exceedance_count': 12, 'secondary_exceedance_count': 12, 'certification_indicator': 'Certified', 'arithmetic_mean': 0.052533, 'standard_deviation': 0.01406, 'first_max_value': 0.094, 'first_max_datetime': '2000-08-01 11:00', 'second_max_value': 0.093, 'second_max_datetime': '2000-08-02 11:00', 'third_max_value': 0.092, 'third_max_datetime': '2000-07-31 11:00', 'fourth_max_value': 0.088, 'fourth_max_datetime': '2000-07-30 10:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 0.093, 'ninety_eighth_percentile': 0.092, 'ninety_fifth_percentile': 0.075, 'ninetieth_percentile': 0.066, 'seventy_fifth_percentile': 0.061, 'fiftieth_percentile': 0.053, 'tenth_percentile': 0.035, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-07-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '88101', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'PM2.5 - Local Conditions', 'sample_duration': '24 HOUR', 'pollutant_standard': 'PM25 24-hour 2006', 'metric_used': 'Daily Mean', 'method': 'R & P Model 2025 PM2.5 Sequential w/WINS - GRAVIMETRIC', 'year': 2000, 'units_of_measure': 'Micrograms/cubic meter (LC)', 'event_type': 'No Events', 'observation_count': 329, 'observation_percent': 90.0, 'validity_indicator': 'Y', 'valid_day_count': 329, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 37, 'primary_exceedance_count': 19, 'secondary_exceedance_count': 19, 'certification_indicator': 'Certification not required', 'arithmetic_mean': 11.250456, 'standard_deviation': 11.330148, 'first_max_value': 72.4, 'first_max_datetime': '2000-12-30 00:00', 'second_max_value': 66.3, 'second_max_datetime': '2000-12-31 00:00', 'third_max_value': 64.1, 'third_max_datetime': '2000-11-22 00:00', 'fourth_max_value': 63.6, 'fourth_max_datetime': '2000-01-01 00:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 63.6, 'ninety_eighth_percentile': 51.4, 'ninety_fifth_percentile': 37.7, 'ninetieth_percentile': 23.3, 'seventy_fifth_percentile': 11.8, 'fiftieth_percentile': 7.2, 'tenth_percentile': 3.8, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-05-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '88101', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'PM2.5 - Local Conditions', 'sample_duration': '24 HOUR', 'pollutant_standard': 'PM25 Annual 2006', 'metric_used': 'Quarterly Means of Daily Means', 'method': 'R & P Model 2025 PM2.5 Sequential w/WINS - GRAVIMETRIC', 'year': 2000, 'units_of_measure': 'Micrograms/cubic meter (LC)', 'event_type': 'No Events', 'observation_count': 329, 'observation_percent': 90.0, 'validity_indicator': 'Y', 'valid_day_count': 329, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 37, 'primary_exceedance_count': None, 'secondary_exceedance_count': None, 'certification_indicator': 'Certification not required', 'arithmetic_mean': 11.250456, 'standard_deviation': 11.330148, 'first_max_value': 72.4, 'first_max_datetime': '2000-12-30 00:00', 'second_max_value': 66.3, 'second_max_datetime': '2000-12-31 00:00', 'third_max_value': 64.1, 'third_max_datetime': '2000-11-22 00:00', 'fourth_max_value': 63.6, 'fourth_max_datetime': '2000-01-01 00:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 63.6, 'ninety_eighth_percentile': 51.4, 'ninety_fifth_percentile': 37.7, 'ninetieth_percentile': 23.3, 'seventy_fifth_percentile': 11.8, 'fiftieth_percentile': 7.2, 'tenth_percentile': 3.8, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-05-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '88101', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'PM2.5 - Local Conditions', 'sample_duration': '24 HOUR', 'pollutant_standard': 'PM25 24-hour 2012', 'metric_used': 'Daily Mean', 'method': 'R & P Model 2025 PM2.5 Sequential w/WINS - GRAVIMETRIC', 'year': 2000, 'units_of_measure': 'Micrograms/cubic meter (LC)', 'event_type': 'No Events', 'observation_count': 329, 'observation_percent': 90.0, 'validity_indicator': 'Y', 'valid_day_count': 329, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 37, 'primary_exceedance_count': 19, 'secondary_exceedance_count': 19, 'certification_indicator': 'Certification not required', 'arithmetic_mean': 11.250456, 'standard_deviation': 11.330148, 'first_max_value': 72.4, 'first_max_datetime': '2000-12-30 00:00', 'second_max_value': 66.3, 'second_max_datetime': '2000-12-31 00:00', 'third_max_value': 64.1, 'third_max_datetime': '2000-11-22 00:00', 'fourth_max_value': 63.6, 'fourth_max_datetime': '2000-01-01 00:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 63.6, 'ninety_eighth_percentile': 51.4, 'ninety_fifth_percentile': 37.7, 'ninetieth_percentile': 23.3, 'seventy_fifth_percentile': 11.8, 'fiftieth_percentile': 7.2, 'tenth_percentile': 3.8, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-05-21'}, {'state_code': '49', 'county_code': '035', 'site_number': '3006', 'parameter_code': '88101', 'poc': 1, 'latitude': 40.736389, 'longitude': -111.872222, 'datum': 'WGS84', 'parameter': 'PM2.5 - Local Conditions', 'sample_duration': '24 HOUR', 'pollutant_standard': 'PM25 Annual 2012', 'metric_used': 'Quarterly Means of Daily Means', 'method': 'R & P Model 2025 PM2.5 Sequential w/WINS - GRAVIMETRIC', 'year': 2000, 'units_of_measure': 'Micrograms/cubic meter (LC)', 'event_type': 'No Events', 'observation_count': 329, 'observation_percent': 90.0, 'validity_indicator': 'Y', 'valid_day_count': 329, 'required_day_count': 366, 'exceptional_data_count': 0, 'null_observation_count': 37, 'primary_exceedance_count': None, 'secondary_exceedance_count': None, 'certification_indicator': 'Certification not required', 'arithmetic_mean': 11.250456, 'standard_deviation': 11.330148, 'first_max_value': 72.4, 'first_max_datetime': '2000-12-30 00:00', 'second_max_value': 66.3, 'second_max_datetime': '2000-12-31 00:00', 'third_max_value': 64.1, 'third_max_datetime': '2000-11-22 00:00', 'fourth_max_value': 63.6, 'fourth_max_datetime': '2000-01-01 00:00', 'first_max_nonoverlap_value': None, 'first_max_n_o_datetime': None, 'second_max_nonoverlap_value': None, 'second_max_n_o_datetime': None, 'ninety_ninth_percentile': 63.6, 'ninety_eighth_percentile': 51.4, 'ninety_fifth_percentile': 37.7, 'ninetieth_percentile': 23.3, 'seventy_fifth_percentile': 11.8, 'fiftieth_percentile': 7.2, 'tenth_percentile': 3.8, 'local_site_name': 'Hawthorne', 'site_address': '1675 SOUTH 600 EAST, SALT LAKE CITY', 'state': 'Utah', 'county': 'Salt Lake', 'city': 'Salt Lake City', 'cbsa_code': '41620', 'cbsa': 'Salt Lake City, UT', 'date_of_last_change': '2018-05-21'}]}


Once again, the JSON data is coming in as 2 dictionaries ('Header' and 'Data'), which each contain their own lists of key-value
pairs. For sanity's sake we'll convert this to a dataframe:

((make sure I'm describing the JSON data correctly))


```python
df = pd.DataFrame((json_data['Data']))
df.head()
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
      <th>state_code</th>
      <th>county_code</th>
      <th>site_number</th>
      <th>parameter_code</th>
      <th>poc</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>datum</th>
      <th>parameter</th>
      <th>sample_duration</th>
      <th>...</th>
      <th>fiftieth_percentile</th>
      <th>tenth_percentile</th>
      <th>local_site_name</th>
      <th>site_address</th>
      <th>state</th>
      <th>county</th>
      <th>city</th>
      <th>cbsa_code</th>
      <th>cbsa</th>
      <th>date_of_last_change</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>49</td>
      <td>035</td>
      <td>3006</td>
      <td>42101</td>
      <td>1</td>
      <td>40.736389</td>
      <td>-111.872222</td>
      <td>WGS84</td>
      <td>Carbon monoxide</td>
      <td>1 HOUR</td>
      <td>...</td>
      <td>1.000</td>
      <td>0.400</td>
      <td>Hawthorne</td>
      <td>1675 SOUTH 600 EAST, SALT LAKE CITY</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>Salt Lake City</td>
      <td>41620</td>
      <td>Salt Lake City, UT</td>
      <td>2018-06-05</td>
    </tr>
    <tr>
      <td>1</td>
      <td>49</td>
      <td>035</td>
      <td>3006</td>
      <td>42101</td>
      <td>1</td>
      <td>40.736389</td>
      <td>-111.872222</td>
      <td>WGS84</td>
      <td>Carbon monoxide</td>
      <td>8-HR RUN AVG END HOUR</td>
      <td>...</td>
      <td>1.100</td>
      <td>0.500</td>
      <td>Hawthorne</td>
      <td>1675 SOUTH 600 EAST, SALT LAKE CITY</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>Salt Lake City</td>
      <td>41620</td>
      <td>Salt Lake City, UT</td>
      <td>2018-06-05</td>
    </tr>
    <tr>
      <td>2</td>
      <td>49</td>
      <td>035</td>
      <td>3006</td>
      <td>42602</td>
      <td>1</td>
      <td>40.736389</td>
      <td>-111.872222</td>
      <td>WGS84</td>
      <td>Nitrogen dioxide (NO2)</td>
      <td>1 HOUR</td>
      <td>...</td>
      <td>25.000</td>
      <td>7.000</td>
      <td>Hawthorne</td>
      <td>1675 SOUTH 600 EAST, SALT LAKE CITY</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>Salt Lake City</td>
      <td>41620</td>
      <td>Salt Lake City, UT</td>
      <td>2017-05-19</td>
    </tr>
    <tr>
      <td>3</td>
      <td>49</td>
      <td>035</td>
      <td>3006</td>
      <td>42602</td>
      <td>1</td>
      <td>40.736389</td>
      <td>-111.872222</td>
      <td>WGS84</td>
      <td>Nitrogen dioxide (NO2)</td>
      <td>1 HOUR</td>
      <td>...</td>
      <td>46.000</td>
      <td>32.000</td>
      <td>Hawthorne</td>
      <td>1675 SOUTH 600 EAST, SALT LAKE CITY</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>Salt Lake City</td>
      <td>41620</td>
      <td>Salt Lake City, UT</td>
      <td>2017-05-19</td>
    </tr>
    <tr>
      <td>4</td>
      <td>49</td>
      <td>035</td>
      <td>3006</td>
      <td>44201</td>
      <td>1</td>
      <td>40.736389</td>
      <td>-111.872222</td>
      <td>WGS84</td>
      <td>Ozone</td>
      <td>1 HOUR</td>
      <td>...</td>
      <td>0.059</td>
      <td>0.037</td>
      <td>Hawthorne</td>
      <td>1675 SOUTH 600 EAST, SALT LAKE CITY</td>
      <td>Utah</td>
      <td>Salt Lake</td>
      <td>Salt Lake City</td>
      <td>41620</td>
      <td>Salt Lake City, UT</td>
      <td>2018-07-21</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 55 columns</p>
</div>




```python
#checking column names
df.columns.tolist()
```




    ['state_code',
     'county_code',
     'site_number',
     'parameter_code',
     'poc',
     'latitude',
     'longitude',
     'datum',
     'parameter',
     'sample_duration',
     'pollutant_standard',
     'metric_used',
     'method',
     'year',
     'units_of_measure',
     'event_type',
     'observation_count',
     'observation_percent',
     'validity_indicator',
     'valid_day_count',
     'required_day_count',
     'exceptional_data_count',
     'null_observation_count',
     'primary_exceedance_count',
     'secondary_exceedance_count',
     'certification_indicator',
     'arithmetic_mean',
     'standard_deviation',
     'first_max_value',
     'first_max_datetime',
     'second_max_value',
     'second_max_datetime',
     'third_max_value',
     'third_max_datetime',
     'fourth_max_value',
     'fourth_max_datetime',
     'first_max_nonoverlap_value',
     'first_max_n_o_datetime',
     'second_max_nonoverlap_value',
     'second_max_n_o_datetime',
     'ninety_ninth_percentile',
     'ninety_eighth_percentile',
     'ninety_fifth_percentile',
     'ninetieth_percentile',
     'seventy_fifth_percentile',
     'fiftieth_percentile',
     'tenth_percentile',
     'local_site_name',
     'site_address',
     'state',
     'county',
     'city',
     'cbsa_code',
     'cbsa',
     'date_of_last_change']



Quite a bit going on here. Many of these column names are self-explanatory, but I referenced this helpful [about page](https://www.epa.gov/outdoor-air-quality-data/about-air-data-reports) for the more obscure ones.

I was curious to see how many different sample durations and pollutant standards are used in assessing air quality:


```python
df['sample_duration'].unique()
```




    array(['1 HOUR', '8-HR RUN AVG END HOUR', '8-HR RUN AVG BEGIN HOUR',
           '24-HR BLK AVG', '3-HR BLK AVG', '5 MINUTE'], dtype=object)




```python
df['pollutant_standard'].unique()
```




    array(['CO 1-hour 1971', 'CO 8-hour 1971', 'NO2 Annual 1971',
           'NO2 1-hour', 'Ozone 1-hour 1979', 'Ozone 8-Hour 1997',
           'Ozone 8-Hour 2008', 'Ozone 8-hour 2015', 'PM25 24-hour 2006',
           'PM25 Annual 2006', 'PM25 24-hour 2012', 'PM25 Annual 2012'],
          dtype=object)



A few revisions over the years!

To help raise public awareness of air quality, I think placing air quality monitors at street level downtown and having live, color-coded displays announcing current AQI (and perhaps a 7-day plotted history) could be of use. Something near Pioneer Park could attract a lot of eyeballs. Having a Tesla sales booth stationed next to it while the Farmer's market is going on might result in a few sales..
