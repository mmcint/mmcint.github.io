---
title: "DS Bootcamp EDA Capstone Project"
excerpt: "EDA Project for the Thinkful DS Bootcamp"
collection: portfolio
---

### Introduction: 

The Gulf of Mexico (GOM) is considered one of the most prolific oil and gas basins in the world. The basin has produced to date more than 5.2 Billion Barrels of Oil Equivalent (BBOE). The GOM is divided into protraction areas and these protraction areas are further broken into 3 mile by 3 mile lease blocks. The blocks are controled the Beauru of Ocean and Energy Managment. Oil and gas companies with a license to operate in the GOM can bid on available leases in yearly lease sales. Lease terms vary from 5-10 years depending on water depth.

One of the jobs of the subsurface staff at an oil and gas company is to monitor competitor lease aquisition. This can show where a specific company, or the industry as a whole, are focusing money and exploration efforts. Competitor intelligence can also help companies create their own strategies for leasing, building exploration campaigns, or provide justification to drill a well. It's important to remember that the lease boudries created by the federal government do not control the geology in the subsurface. Oil and gas fields frequently cross lease boundries, so understanding what targets companies around your own leases and infrastructure are pursuing can help protect your own reserves and identify potential targets for joint ventures and future exploration.

### Hypothesis:

Ho: There is no significant difference in the mean bid (USD) per lease block per company between 2014-2020.
Ha: There is a significant difference in the mean bid (USD) per lease block per company between 2014-2020

### Data:

The data is publicly available on the BOEM website. This dataset was pulled from the GOMSmart database, which organizes lease information into a .csv format. The dataset contains 1708 records and 17 variables. I've only imported 10 variables to be used for this analysis. There are no missing values.



```python
%matplotlib inline
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

url = "https://github.com/mmcintire00/tf_exp_design/blob/main/gom_ls_high_bids_2014-2020.csv?raw=true"
oil_price = "https://github.com/mmcintire00/tf_exp_design/blob/main/eia_us_monthly_crude_prices_2014-2020.csv?raw=true"

oil_price_df = pd.read_csv(oil_price)
gom_df = pd.read_csv(url, usecols=["Sale No.", "Sale Date", "Area Block", "Total Bid", "% Share", "Net Bid", "Sortname", "Status", "Primary Term"]) # only need these 8 columns for analysis
gom_df = gom_df.rename(columns={'Sortname':'company'}) # rename column for companyies
gom_df = gom_df.rename(columns={'Area Block': 'area_block'})
gom_df = gom_df.rename(columns={'% Share': 'pct_share'})
gom_df = gom_df.rename(columns={'Net Bid': 'net_bid'})
gom_df = gom_df.rename(columns={'Total Bid': 'total_bid'})
```


```python
import plotly.graph_objects as go
import ipywidgets as ipy
from plotly.subplots import make_subplots
import plotly.express as px
import re
```


```python
gom_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1708 entries, 0 to 1707
    Data columns (total 9 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   Sale No.      1708 non-null   object 
     1   Sale Date     1708 non-null   object 
     2   area_block    1708 non-null   object 
     3   total_bid     1708 non-null   int64  
     4   company       1708 non-null   object 
     5   pct_share     1708 non-null   float64
     6   net_bid       1708 non-null   float64
     7   Status        1708 non-null   object 
     8   Primary Term  1708 non-null   int64  
    dtypes: float64(2), int64(2), object(5)
    memory usage: 120.2+ KB



```python
print(gom_df.isnull().sum())
```

    Sale No.        0
    Sale Date       0
    area_block      0
    total_bid       0
    company         0
    pct_share       0
    net_bid         0
    Status          0
    Primary Term    0
    dtype: int64



```python
gom_df['year'] = pd.DatetimeIndex(gom_df['Sale Date']).year # create new column with year for grouping
gom_df.head()
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
      <th>Sale No.</th>
      <th>Sale Date</th>
      <th>area_block</th>
      <th>total_bid</th>
      <th>company</th>
      <th>pct_share</th>
      <th>net_bid</th>
      <th>Status</th>
      <th>Primary Term</th>
      <th>year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC767</td>
      <td>18450550</td>
      <td>NOBLE ENERGY INC</td>
      <td>50.0</td>
      <td>9225275.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>1</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC767</td>
      <td>18450550</td>
      <td>RIDGEWOOD ENERGY CORPORATION</td>
      <td>50.0</td>
      <td>9225275.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>2</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>LLOG BLUEWATER HOLDINGS LLC</td>
      <td>70.0</td>
      <td>656288.50</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>3</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>25.0</td>
      <td>234388.75</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>4</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>HOUSTON ENERGY LP</td>
      <td>5.0</td>
      <td>46877.75</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
    </tr>
  </tbody>
</table>
</div>




```python
oil_price_df.head()
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
      <th>date</th>
      <th>us_crude_price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Jan-2014</td>
      <td>89.57</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Feb-2014</td>
      <td>96.86</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mar-2014</td>
      <td>96.17</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Apr-2014</td>
      <td>96.49</td>
    </tr>
    <tr>
      <th>4</th>
      <td>May-2014</td>
      <td>95.74</td>
    </tr>
  </tbody>
</table>
</div>




```python
# number of bids total per year
gom_df[['total_bid']].groupby(gom_df['year']).agg(['count'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>total_bid</th>
    </tr>
    <tr>
      <th></th>
      <th>count</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014</th>
      <td>340</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>194</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>179</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>257</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>224</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>353</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>161</td>
    </tr>
  </tbody>
</table>
</div>




```python
# sum of net bids from 2014 through 2020
print(int(sum(gom_df['net_bid'])))
```

    2858303852



```python
# bid statistics for each year
grouped_df = gom_df.groupby(['year']).agg({'net_bid': ['mean', 'min', 'max']})
grouped_df.columns = ['bid_mean', 'bid_min', 'bid_max']
grouped_df.head(7)
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
      <th>bid_mean</th>
      <th>bid_min</th>
      <th>bid_max</th>
    </tr>
    <tr>
      <th>year</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2014</th>
      <td>2.672540e+06</td>
      <td>18963.8750</td>
      <td>68790000.00</td>
    </tr>
    <tr>
      <th>2015</th>
      <td>2.792015e+06</td>
      <td>39129.7000</td>
      <td>49612901.65</td>
    </tr>
    <tr>
      <th>2016</th>
      <td>9.275250e+05</td>
      <td>22610.0875</td>
      <td>13225126.00</td>
    </tr>
    <tr>
      <th>2017</th>
      <td>1.453271e+06</td>
      <td>19144.1250</td>
      <td>24056719.00</td>
    </tr>
    <tr>
      <th>2018</th>
      <td>1.253246e+06</td>
      <td>15144.4250</td>
      <td>25919784.00</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>1.113279e+06</td>
      <td>8000.0000</td>
      <td>24495776.00</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>1.209673e+06</td>
      <td>62840.0000</td>
      <td>11114050.00</td>
    </tr>
  </tbody>
</table>
</div>



#Data Clean-up:

#### Need to clean company names bc some companies report names with slight changes over the time span of analysis.


```python
# function to normalize company names
def clean_company(c):
  if 'LLOG' in c:
    return 'LLOG'
  elif 'STATOIL' in c:
    return 'EQUINOR GULF OF MEXICO LLC'
  elif 'FREEPORT' in c:
    return 'FREEPORT MCMORAN'
  elif 'ANADARKO' in c:
    return 'ANADARKO'
  elif 'TALOS' in c:
    return 'TALOS ENERGY'
  elif 'DEEP GULF' in c:
    return 'DEEP GULF ENERGY'
  elif 'FIELDWOOD' in c:
    return 'FIELDWOOD ENERGY'
  else:
    return c 

gom_df['clean_company'] = gom_df.company.apply(lambda x: clean_company(x)) 
```

#### Extract area name from 'area_block' and convert to 'clean_area' for analysis by protraction area.


```python
# function to filter by protraction area
def protraction_area_filter(x):
  if "MC" in x:
    return "Mississippi Canyon"
  elif "AT" in x:
    return "Atwater Valley"
  elif "GC" in x:
    return "Green Canyon"
  elif "WR" in x:
    return "Walker Ridge"
  elif "KC" in x:
    return "Keathley Canyon"
  elif "GB" in x:
    return "Garden Banks"
  elif "AC" in x:
    return "Alaminos Canyon"
  elif "EB" in x:
    return "East Breaks"
  elif "DC" in x:
    return "Desoto Canyon"
  elif "LL" in x:
    return "Lloyd Ridge"
  else:
    return x

gom_df['clean_area'] = gom_df.area_block.apply(lambda x: protraction_area_filter(x)) 
```


```python
gom_df['clean_company'].unique()
```




    array(['NOBLE ENERGY INC', 'RIDGEWOOD ENERGY CORPORATION', 'LLOG',
           'RED WILLOW OFFSHORE LLC', 'HOUSTON ENERGY LP',
           'EQUINOR GULF OF MEXICO LLC', 'FREEPORT MCMORAN', 'ANADARKO',
           'BP EXPLORATION & PRODUCTION INC',
           'COBALT INTERNATIONAL ENERGY LP', 'TOTAL E&P USA INC',
           'CHEVRON USA INC', 'TALOS ENERGY', 'CONOCOPHILLIPS COMPANY',
           'STONE ENERGY OFFSHORE LLC', 'ENI PETROLEUM US LLC',
           'DEEP GULF ENERGY', 'SHELL OFFSHORE INC.',
           'BHP BILLITON PETROLEUM (DEEPWATER) INC',
           'MURPHY EXPLORATION & PRODUCTION COMPANY - USA',
           'REPSOL E&P USA INC', 'MARATHON OIL COMPANY',
           'GULFSLOPE ENERGY INC', 'HESS CORPORATION', 'VENARI OFFSHORE LLC',
           'ENVEN ENERGY VENTURES LLC', 'EXXON MOBIL CORPORATION',
           'ECOPETROL AMERICA INC', 'NAVITAS PETROLEUM US LLC',
           'APACHE SHELF EXPLORATION LLC', 'FIELDWOOD ENERGY',
           'SAMSON OFFSHORE LLC', 'PXP OFFSHORE LLC',
           'UNION OIL COMPANY OF CALIFORNIA', 'FOCUS EXPLORATION LLC',
           'PEREGRINE OIL & GAS II LLC', 'WALTER OIL & GAS CORPORATION',
           'CL&F OFFSHORE LLC', 'BEACON OFFSHORE ENERGY EXPLORATION LLC',
           'W & T OFFSHORE INC',
           'KOSMOS ENERGY GULF OF MEXICO OPERATIONS LLC',
           'CSL EXPLORATION LP'], dtype=object)



#### Keep only companies that submitted bids each year of analysis (2014-2020)


```python
cleanCompNames = gom_df['clean_company'].unique()

condensed_gom = gom_df[['clean_company', 'year']] # Subset df to just these columns
newGom = condensed_gom.groupby(['clean_company']).agg({'year': ['max']}) # This sucks, but it works
newGom = newGom.reset_index() # reset index 

# Flatten multi-index DF AT THE COLUMN LEVEL (artifact from groupby above) to a 2D object
# allows us to access 1D series (columns)
newGom.columns = newGom.columns.get_level_values(0)

cleanedGom = newGom[newGom['year'] != 2020].reset_index() # Reset index so we don't have out of order indices
cleanedGom = cleanedGom['clean_company'] # Filter to just company names
cleanedGom = cleanedGom.to_list() # turn that into a list object (as opposed to pandas series)

# remove companies that did not submit bids all 7 years of study
gom_clean_df = gom_df[~gom_df['clean_company'].isin(cleanedGom)]
gom_clean_df.head(10)
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
      <th>Sale No.</th>
      <th>Sale Date</th>
      <th>area_block</th>
      <th>total_bid</th>
      <th>company</th>
      <th>pct_share</th>
      <th>net_bid</th>
      <th>Status</th>
      <th>Primary Term</th>
      <th>year</th>
      <th>clean_company</th>
      <th>clean_area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC767</td>
      <td>18450550</td>
      <td>RIDGEWOOD ENERGY CORPORATION</td>
      <td>50.0</td>
      <td>9225275.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>RIDGEWOOD ENERGY CORPORATION</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>2</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>LLOG BLUEWATER HOLDINGS LLC</td>
      <td>70.0</td>
      <td>656288.50</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
      <td>LLOG</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>3</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>25.0</td>
      <td>234388.75</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>4</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>HOUSTON ENERGY LP</td>
      <td>5.0</td>
      <td>46877.75</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
      <td>HOUSTON ENERGY LP</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>5</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC845</td>
      <td>1546207</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>1546207.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>6</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC847</td>
      <td>749608</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>749608.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>7</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC890</td>
      <td>1215308</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>1215308.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>8</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC891</td>
      <td>749608</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>749608.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>11</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC988</td>
      <td>757753</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>75.0</td>
      <td>568314.75</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>12</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC988</td>
      <td>757753</td>
      <td>HOUSTON ENERGY LP</td>
      <td>25.0</td>
      <td>189438.25</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>HOUSTON ENERGY LP</td>
      <td>Green Canyon</td>
    </tr>
  </tbody>
</table>
</div>




```python
gom_clean_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1386 entries, 1 to 1707
    Data columns (total 12 columns):
     #   Column         Non-Null Count  Dtype  
    ---  ------         --------------  -----  
     0   Sale No.       1386 non-null   object 
     1   Sale Date      1386 non-null   object 
     2   area_block     1386 non-null   object 
     3   total_bid      1386 non-null   int64  
     4   company        1386 non-null   object 
     5   pct_share      1386 non-null   float64
     6   net_bid        1386 non-null   float64
     7   Status         1386 non-null   object 
     8   Primary Term   1386 non-null   int64  
     9   year           1386 non-null   int64  
     10  clean_company  1386 non-null   object 
     11  clean_area     1386 non-null   object 
    dtypes: float64(2), int64(3), object(7)
    memory usage: 140.8+ KB



```python
oil_price_df.plot.line(x='date', y='us_crude_price')
plt.ylabel(ylabel='Price per barrel (USD)')
plt.xlabel(xlabel='Date')
plt.title(label='US Crude Oil Price')
```




    Text(0.5, 1.0, 'US Crude Oil Price')




    
![png](tf_experimental_design_capstone_files/tf_experimental_design_capstone_18_1.png)
    



```python
fig = go.Figure()
fig.add_trace(go.Scatter(x=oil_price_df['date'], y=oil_price_df['us_crude_price'], mode='lines', name='lines'))
fig.update_layout(title_text='Crude Oil Prices 2014-2020')
fig.show()
```


<html>
<head><meta charset="utf-8" /></head>
<body>
    <div>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-AMS-MML_SVG"></script><script type="text/javascript">if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}</script>
                <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script>
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>    
            <div id="7740656c-5b7e-4249-948d-4cc89a564613" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">

                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("7740656c-5b7e-4249-948d-4cc89a564613")) {
                    Plotly.newPlot(
                        '7740656c-5b7e-4249-948d-4cc89a564613',
                        [{"mode": "lines", "name": "lines", "type": "scatter", "x": ["Jan-2014", "Feb-2014", "Mar-2014", "Apr-2014", "May-2014", "Jun-2014", "Jul-2014", "Aug-2014", "Sep-2014", "Oct-2014", "Nov-2014", "Dec-2014", "Jan-2015", "Feb-2015", "Mar-2015", "Apr-2015", "May-2015", "Jun-2015", "Jul-2015", "Aug-2015", "Sep-2015", "Oct-2015", "Nov-2015", "Dec-2015", "Jan-2016", "Feb-2016", "Mar-2016", "Apr-2016", "May-2016", "Jun-2016", "Jul-2016", "Aug-2016", "Sep-2016", "Oct-2016", "Nov-2016", "Dec-2016", "Jan-2017", "Feb-2017", "Mar-2017", "Apr-2017", "May-2017", "Jun-2017", "Jul-2017", "Aug-2017", "Sep-2017", "Oct-2017", "Nov-2017", "Dec-2017", "Jan-2018", "Feb-2018", "Mar-2018", "Apr-2018", "May-2018", "Jun-2018", "Jul-2018", "Aug-2018", "Sep-2018", "Oct-2018", "Nov-2018", "Dec-2018", "Jan-2019", "Feb-2019", "Mar-2019", "Apr-2019", "May-2019", "Jun-2019", "Jul-2019", "Aug-2019", "Sep-2019", "Oct-2019", "Nov-2019", "Dec-2019", "Jan-2020", "Feb-2020", "Mar-2020", "Apr-2020", "May-2020", "Jun-2020", "Jul-2020", "Aug-2020", "Sep-2020", "Oct-2020", "Nov-2020", "Dec-2020", "Jan-2021"], "y": [89.57, 96.86, 96.17, 96.49, 95.74, 98.68, 96.7, 90.72, 86.87, 78.84, 71.07, 54.86, 43.06, 44.35, 42.66, 49.3, 54.38, 55.88, 47.7, 39.98, 41.6, 42.34, 38.19, 32.26, 27.02, 25.52, 31.87, 35.59, 41.02, 43.96, 40.71, 40.46, 40.55, 45.0, 41.65, 47.12, 48.19, 49.41, 46.39, 47.23, 45.19, 42.17, 43.42, 44.96, 47.17, 49.12, 55.19, 56.98, 62.25, 61.18, 60.68, 63.5, 66.16, 62.8, 67.0, 62.64, 63.54, 65.18, 55.65, 47.63, 48.0, 52.6, 57.46, 63.0, 59.73, 54.34, 56.47, 53.63, 55.14, 53.14, 54.96, 58.41, 56.86, 50.03, 31.8, 15.99, 18.09, 33.53, 37.55, 39.42, 36.89, 36.46, 38.23, 43.97, 49.76]}],
                        {"template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Crude Oil Prices 2014-2020"}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('7740656c-5b7e-4249-948d-4cc89a564613');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };

            </script>
        </div>
</body>
</html>


#### Remove protraction areas that did not recieve bids for all 7 years of analysis (2014-2020)


```python
# remove protraction areas that do not recieve bids for all 7 years of study
remove_area = ['Lloyd Ridge', 'Desoto Canyon', 'East Breaks']
gom_clean_df = gom_clean_df[~gom_clean_df['clean_area'].isin(remove_area)]
gom_clean_df.head(10)
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
      <th>Sale No.</th>
      <th>Sale Date</th>
      <th>area_block</th>
      <th>total_bid</th>
      <th>company</th>
      <th>pct_share</th>
      <th>net_bid</th>
      <th>Status</th>
      <th>Primary Term</th>
      <th>year</th>
      <th>clean_company</th>
      <th>clean_area</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC767</td>
      <td>18450550</td>
      <td>RIDGEWOOD ENERGY CORPORATION</td>
      <td>50.0</td>
      <td>9225275.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>RIDGEWOOD ENERGY CORPORATION</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>2</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>LLOG BLUEWATER HOLDINGS LLC</td>
      <td>70.0</td>
      <td>656288.50</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
      <td>LLOG</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>3</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>25.0</td>
      <td>234388.75</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>4</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC816</td>
      <td>937555</td>
      <td>HOUSTON ENERGY LP</td>
      <td>5.0</td>
      <td>46877.75</td>
      <td>RELINQ</td>
      <td>10</td>
      <td>2014</td>
      <td>HOUSTON ENERGY LP</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>5</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC845</td>
      <td>1546207</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>1546207.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>6</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC847</td>
      <td>749608</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>749608.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>7</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC890</td>
      <td>1215308</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>1215308.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>8</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC891</td>
      <td>749608</td>
      <td>STATOIL GULF OF MEXICO LLC</td>
      <td>100.0</td>
      <td>749608.00</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>EQUINOR GULF OF MEXICO LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>11</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC988</td>
      <td>757753</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>75.0</td>
      <td>568314.75</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>RED WILLOW OFFSHORE LLC</td>
      <td>Green Canyon</td>
    </tr>
    <tr>
      <th>12</th>
      <td>231</td>
      <td>3/19/2014</td>
      <td>GC988</td>
      <td>757753</td>
      <td>HOUSTON ENERGY LP</td>
      <td>25.0</td>
      <td>189438.25</td>
      <td>PRIMRY</td>
      <td>10</td>
      <td>2014</td>
      <td>HOUSTON ENERGY LP</td>
      <td>Green Canyon</td>
    </tr>
  </tbody>
</table>
</div>




```python
plot = go.Figure(data=go.Violin(x=gom_clean_df['year'], y=gom_clean_df['net_bid']))
plot.update_layout(title_text='Net Lease Bids between 2014-2020')
plot.update_yaxes(title_text='Net Lease Bid (USD)')
plot.update_xaxes(title_text='Year')
plot.show()
```


<html>
<head><meta charset="utf-8" /></head>
<body>
    <div>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-AMS-MML_SVG"></script><script type="text/javascript">if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}</script>
                <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script>
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>    
            <div id="ed0f6398-235e-4ce4-83d8-136b882f5c15" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">

                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("ed0f6398-235e-4ce4-83d8-136b882f5c15")) {
                    Plotly.newPlot(
                        'ed0f6398-235e-4ce4-83d8-136b882f5c15',
                        [{"type": "violin", "x": [2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2014, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2015, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2016, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2017, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2018, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2019, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020, 2020], "y": [9225275.0, 656288.5, 234388.75, 46877.75, 1546207.0, 749608.0, 1215308.0, 749608.0, 568314.75, 189438.25, 3675500.0, 1514809.0, 1155500.0, 1421519.0, 784747.6608, 82552.4883, 558476.8509, 4807621.2608, 505743.5883, 3421412.1509, 654809.0, 1194809.0, 784747.6608, 82552.4883, 558476.8509, 8014809.0, 6014809.0, 300564.8, 1194809.0, 1194809.0, 300564.8, 300564.8, 1260300.0, 768500.0, 634500.0, 1155500.0, 546150.0, 654809.0, 654809.0, 654809.0, 2014809.0, 6006910.7, 2145325.25, 429065.05, 1194809.0, 1806910.7, 645325.25, 129065.05, 834600.0, 1232778.0, 1073216.0, 810129.6, 289332.0, 57866.4, 992756.0, 548461.815, 2843452.0, 35938136.0, 654809.0, 654809.0, 379277.5, 180156.8125, 18963.875, 180156.8125, 8149944.075, 857888.85, 8149944.075, 3302512.5, 471787.5, 5661450.0, 707714.0, 752888.0, 752888.0, 7500125.0, 1194809.0, 62356855.0, 596888.0, 654809.0, 3001125.0, 1152125.0, 1788888.0, 399244.4, 372775.0, 250164.8, 749608.0, 749608.0, 654809.0, 654809.0, 1555555.4, 555555.5, 111111.1, 500564.8, 299596.4, 1089000.0, 1210000.0, 239364.8, 2040564.8, 443520.0, 443520.0, 685177.0, 7844809.0, 593280.0, 1194809.0, 305280.0, 1194809.0, 750000.0, 220317.0, 270890.0, 261105.0, 547200.0, 547200.0, 218880.0, 5422549.925, 6256788.375, 9725725.0, 585725.0, 585725.0, 7975092.0, 2690237.0, 772237.0, 2453928.85, 258308.3, 2453928.85, 813897.5, 640834.6, 228869.5, 45773.9, 950216.0, 635725.0, 407877.75, 42934.5, 407877.75, 457584.5, 411826.05, 45758.45, 717115.0, 1771500.0, 1025725.0, 596383.0, 1049060.0, 5005430.7, 843996.0, 1413500.0, 1413500.0, 843996.0, 413500.0, 413500.0, 1413500.0, 1413500.0, 1413500.0, 1413500.0, 18668138.0269, 8000938.0269, 240647.2, 8225725.0, 9673896.0, 1458208.0, 3867896.0, 547815.8, 195648.5, 39129.7, 647916.5, 231398.75, 46279.75, 730216.0, 324647.2, 2525725.0, 3125725.0, 725725.0, 855725.0, 585725.0, 1125725.0, 725725.0, 5255725.0, 758394.0, 2172222.0, 1522222.0, 2172222.0, 1222222.0, 2172222.0, 1522222.0, 772222.0, 1893333.0, 1893333.0, 49612901.65, 2611205.35, 1203333.0, 469116.75, 30839461.5, 32355096.0, 725725.0, 4694730.75, 1421691.0, 997122.0, 585000.0, 775092.0, 1025092.0, 1275092.0, 1550237.0, 3280237.0, 625489.0, 625489.0, 625489.0, 625489.0, 1370216.0, 1370216.0, 4995125.0, 834796.5, 469116.75, 6448139.25, 997122.0, 725725.0, 725725.0, 1151027.0, 671027.0, 671027.0, 1750314.0, 931027.0, 1506250.0, 2806250.0, 1351027.0, 521027.0, 621027.0, 621027.0, 739321.0, 8539321.0, 5739321.0, 7439321.0, 3339321.0, 1206250.0, 1206250.0, 1106250.0, 906250.0, 906250.0, 906250.0, 906250.0, 1106250.0, 906250.0, 721027.0, 576000.0, 576000.0, 576000.0, 576000.0, 576000.0, 1151027.0, 1151027.0, 906250.0, 1151027.0, 1151027.0, 671027.0, 821027.0, 821027.0, 1351027.0, 4406250.0, 3106250.0, 2321027.0, 464774.625, 50461.245, 479972.0175, 463588.125, 48798.75, 463588.125, 1321681.0, 1355355.0, 621681.0, 621681.0, 621681.0, 621681.0, 610624.0, 621681.0, 621681.0, 621681.0, 761108.0, 636829.0, 710624.0, 710624.0, 621681.0, 13225126.0, 571681.0, 822126.0, 1089624.0, 3500624.0, 676108.0, 761108.0, 626829.0, 752003.0, 312312.0, 312312.0, 626829.0, 208187.1792, 208249.6416, 208187.1792, 382929.6, 1838108.0, 318929.6, 921681.0, 214761.0465, 243493.25, 22610.0875, 214761.0465, 621681.0, 422922.5, 422922.5, 921681.0, 10265331.0, 474659.196, 1089624.0, 596383.0, 5347258.554, 596383.0, 7010624.0, 968084.6675, 937432.125, 101778.345, 968084.6675, 385540.6805, 373333.275, 40533.327000000005, 385540.6805, 367411.653, 416566.5, 38681.175, 367411.653, 7089624.0, 479972.0175, 402929.6, 1089624.0, 300000.0, 1125000.0, 173923.75, 243493.25, 34784.75, 243493.25, 593392.8, 31231.2, 593392.8, 31231.2, 1206557.0, 2866687.0, 736676.0, 152252.25, 426306.3, 30450.45, 152252.25, 426306.3, 30450.45, 650000.0, 925000.0, 278929.6, 621681.0, 610624.0, 710624.0, 685178.76, 710624.0, 2765270.0, 1565270.0, 621681.0, 287537.5, 805105.0, 57507.5, 921681.0, 721681.0, 156156.0, 218618.4, 31231.2, 218618.4, 2425126.0, 390140.0, 546196.0, 78028.0, 546196.0, 390140.0, 546196.0, 78028.0, 546196.0, 2125126.0, 390140.0, 546196.0, 78028.0, 546196.0, 156156.0, 218618.4, 31231.2, 218618.4, 674888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 608888.0, 878552.0, 774888.0, 749888.0, 699888.0, 400027.5, 400027.5, 400028.0, 750719.0, 5056719.0, 576915.0, 3056719.0, 655555.0, 757757.0, 605707.0, 605707.0, 900719.0, 900719.0, 900719.0, 591730.0, 900719.0, 750719.0, 750227.0, 900227.0, 968976.0, 12600227.0, 3769784.0, 6833767.0, 802787.0, 676915.0, 309171.8, 32544.4, 309171.8, 576000.0, 18327777.0, 576000.0, 585150.0, 603000.0, 585150.0, 625260.0, 1001260.0, 625260.0, 711115.0, 625260.0, 625260.0, 3038291.0, 625260.0, 940823.0, 650719.0, 3630260.0, 5134285.0, 2560730.0, 625260.0, 763367.0, 4056719.0, 591730.0, 591730.0, 605707.0, 2560730.0, 2620787.0, 3722737.0, 650719.0, 1372908.0, 1640115.0, 1640115.0, 875555.0, 585250.0, 1556719.0, 750719.0, 650719.0, 68202.0, 68202.0, 238707.0, 382689.0, 591976.0, 2156719.0, 4556719.0, 613976.0, 669976.0, 726250.0, 895260.0, 11307829.0, 763367.0, 638888.75, 127777.75, 1788888.5, 445788.672, 176126.328, 763367.0, 430019.07200000004, 169895.928, 331776.0, 395136.0, 757757.0, 94444.625, 264444.95, 94444.625, 37777.85, 264444.95, 206944.375, 579444.25, 206944.375, 82777.75, 579444.25, 568010.8, 405722.0, 81144.4, 568010.8, 75252.5, 75252.5, 376262.5, 225757.5, 24056719.0, 591730.0, 591730.0, 591730.0, 749319.0, 591730.0, 591730.0, 3710829.0, 650719.0, 650719.0, 1661976.0, 608976.0, 1231976.0, 1788999.0, 1400260.0, 631976.0, 21237976.0, 375321.6, 3618976.0, 11124976.0, 582976.0, 955976.0, 2250719.0, 887480.0, 619726.0, 619726.0, 643726.0, 619726.0, 643726.0, 643726.0, 619726.0, 643726.0, 619726.0, 643726.0, 737480.0, 1077480.0, 2179258.0, 2005689.0, 2005689.0, 3503689.0, 1218215.0, 751689.0, 607689.0, 3183116.0, 5683116.0, 2754689.0, 1518215.0, 1618215.0, 1518215.0, 1714458.0, 1714458.0, 1095305.0, 3801680.0, 581680.0, 1507689.0, 616680.0, 608888.0, 612888.0, 512807.5, 181869.1875, 382882.5, 19144.125, 181869.1875, 576000.0, 603689.0, 655689.0, 576000.0, 115000.0, 115000.0, 115000.0, 115000.0, 602689.0, 608689.0, 606689.0, 607689.0, 12100717.0, 597717.0, 581680.0, 511252.5, 511252.5, 581680.0, 1766829.0, 286363.0, 204545.0, 40909.0, 286363.0, 264517.75, 188941.25, 37788.25, 264517.75, 836888.0, 593829.0, 607689.0, 1003689.0, 605689.0, 605689.0, 1923829.0, 4500725.0, 950725.0, 1095305.0, 1095305.0, 799976.0, 1599976.0, 1999976.0, 599976.0, 572071.5, 63563.5, 320945.625, 33783.75, 320945.625, 368493.125, 38788.75, 368493.125, 1701680.0, 1001680.0, 2120402.0, 1402624.0, 1301988.0, 1101988.0, 590710.0, 800526.0, 800526.0, 1001168.0, 144889.0, 876889.0, 1201889.0, 610526.0, 651988.0, 651988.0, 851988.0, 771988.0, 651988.0, 771988.0, 651889.0, 610526.0, 1001205.0, 610526.0, 610526.0, 657776.0, 657776.0, 1200526.0, 1501319.0, 4130860.0, 1501319.0, 703976.0, 715777.0, 435505.2992, 311098.3248, 62203.4872, 804776.0, 1205976.0, 486888.5, 590710.0, 690710.0, 1268215.0, 1001116.0, 641988.0, 1268215.0, 1268215.0, 641988.0, 1001116.0, 1001116.0, 641988.0, 641988.0, 1001116.0, 1001116.0, 1001116.0, 2101988.0, 6501988.0, 3801988.0, 1201988.0, 1402624.0, 650168.0, 1501319.0, 650168.0, 650168.0, 1710305.0, 1710305.0, 1286286.0, 1475287.0, 717402.0, 595330.0, 702331.0, 650168.0, 4133333.0, 651889.0, 851889.0, 2900526.0, 369000.0, 1308112.0, 7000728.0, 4240226.0, 587427.0, 1782035.0, 1367275.0, 1103014.0, 1121260.0, 3887975.0, 603014.0, 4601988.0, 603014.0, 603014.0, 1001988.0, 587427.0, 609000.0, 362750.0, 601507.0, 601507.0, 368833.25, 553249.875, 25919784.0, 1006507.0, 1006507.0, 2009784.0, 143872.0375, 302888.5, 15144.425, 143872.0375, 339444.25, 339444.25, 11116013.0, 850228.0, 950248.0, 3003880.0, 701776.0, 1001776.0, 1001776.0, 1001776.0, 1001776.0, 701776.0, 1104508.0, 701776.0, 701776.0, 904776.0, 1101576.0, 701776.0, 901776.0, 619880.0, 701988.0, 425617.0, 425617.0, 3602880.0, 1753014.0, 1509784.0, 292499.5, 292499.5, 589784.0, 224803.25, 359685.2, 152866.21, 603014.0, 603014.0, 701776.0, 701776.0, 313920.0, 78480.0, 78480.0, 156960.0, 708000.0, 604000.0, 610000.0, 750198.0, 3250112.0, 1352908.0, 3333116.0, 587427.0, 587427.0, 610527.0, 1267427.0, 853014.0, 651988.0, 651988.0, 651988.0, 1067505.0, 582450.0, 110207.0, 1841626.0, 984979.2, 800524.0, 492663.2, 625000.0, 4067505.0, 413660.5, 413660.5, 800524.0, 1001988.0, 298178.7, 695750.3, 600524.0, 801988.0, 880524.0, 500000.0, 589909.0, 801988.0, 801988.0, 801988.0, 801988.0, 801988.0, 801988.0, 801988.0, 801988.0, 771988.0, 751988.0, 1048178.7, 2445750.3, 10100991.0, 9003010.0, 6003010.0, 607505.0, 4567505.0, 24495776.0, 605524.0, 600524.0, 607505.0, 801988.0, 600524.0, 2067505.0, 651988.0, 96473.97, 607505.0, 1601988.0, 620000.0, 801988.0, 801988.0, 607505.0, 607505.0, 4100991.0, 500909.0, 1067505.0, 1815046.2, 756269.25, 453761.55, 589909.0, 607505.0, 1307777.0, 970909.0, 589909.0, 453014.4, 302009.6, 600000.0, 4311212.0, 600000.0, 403626.3, 607505.0, 607505.0, 607505.0, 607505.0, 607505.0, 607505.0, 700524.0, 600524.0, 4096676.0, 651988.0, 651988.0, 650524.0, 751988.0, 751988.0, 731988.0, 751988.0, 751988.0, 751988.0, 771988.0, 731988.0, 771988.0, 8000.0, 1008038.5, 302411.55, 504019.25, 201607.7, 1537073.0, 175000.0, 200000.0, 150000.0, 453333.0, 188888.75, 113333.25, 300000.0, 300000.0, 600524.0, 1209776.0, 605524.0, 600524.0, 801988.0, 801988.0, 801988.0, 801988.0, 801988.0, 811988.0, 811988.0, 701988.0, 801988.0, 801988.0, 751988.0, 751988.0, 7201988.0, 8201988.0, 801988.0, 701988.0, 701988.0, 701988.0, 1201988.0, 3500106.5, 607505.0, 607505.0, 1430837.0, 615829.0, 493048.45, 87008.55, 514080.85, 90720.15, 514080.85, 90720.15, 1430837.0, 514080.85, 90720.15, 1100524.0, 605524.0, 605524.0, 302311.5, 302311.5, 825311.5, 825311.5, 474811.5, 474811.5, 609993.0, 613784.0, 611884.0, 750000.0, 620000.0, 670000.0, 300858.0, 300858.0, 576000.0, 850319.0, 1550319.0, 910319.0, 576000.0, 910319.0, 910319.0, 910319.0, 576000.0, 2101989.0, 627989.0, 22510319.0, 585989.0, 1910319.0, 802599.0, 802599.0, 301299.5, 150649.75, 150649.75, 836004.0, 1000505.0, 608021.0, 743347.0, 589000.0, 631321.0, 631321.0, 1851989.0, 615224.0, 615224.0, 865224.0, 3101989.0, 615224.0, 1205676.0, 1268018.0, 814018.0, 801004.0, 958018.0, 715224.0, 1125224.0, 6710012.0, 2170018.0, 615224.0, 615224.0, 1835018.0, 910319.0, 910319.0, 670319.0, 1310319.0, 750319.0, 911776.0, 911776.0, 809776.0, 627776.0, 627776.0, 1201004.0, 5601004.0, 814018.0, 2173018.0, 310858.0, 310858.0, 1510319.0, 4010505.0, 1001505.0, 850505.0, 631321.0, 631321.0, 302209.0, 302209.0, 631321.0, 875224.0, 1125224.0, 589000.0, 875224.0, 589000.0, 589000.0, 360949.5, 360949.5, 442999.5, 442999.5, 705505.0, 4201505.0, 1801505.0, 705505.0, 2101505.0, 705505.0, 871716.0, 711876.0, 627776.0, 627776.0, 631321.0, 850505.0, 631321.0, 850505.0, 4010505.0, 589000.0, 900207.0, 701207.0, 607505.0, 736121.0, 736121.0, 576630.0, 576630.0, 576630.0, 611116.0, 576630.0, 576630.0, 219940.0, 62840.0, 188520.0, 157100.0, 621212.0, 220412.5, 188925.0, 220412.5, 621212.0, 621212.0, 1141027.0, 2490120.0, 609116.0, 2018028.0, 106212.0, 758676.0, 758676.0, 852610.0, 576630.0, 1104050.0, 937610.0, 576000.0, 1016816.0, 2560009.0, 11114050.0, 150630.0, 150630.0, 3602610.0, 5114050.0, 4002610.0, 6002610.0, 950630.0, 187579.75, 562739.25, 150079.75, 450239.25, 1804114.0, 575079.75, 1725239.25, 150079.75, 450239.25, 621212.0, 374887.5, 374887.5, 758676.0, 582676.0, 758676.0, 852610.0, 576630.0, 576630.0, 2202610.0, 576000.0, 576000.0, 1504050.0, 736121.0, 621212.0, 621212.0, 1300630.0, 581234.0, 7282402.0, 517520.25, 600311.0, 2401229.0, 2604084.0, 2125121.0, 2213751.0, 1500311.0, 576406.0, 765406.0, 901229.0, 2301229.0, 576406.0, 901229.0, 600311.0, 4210311.0, 600311.0, 851229.0, 851229.0, 1401229.0, 1401229.0, 851229.0, 1801229.0, 2401229.0, 2001229.0, 2001229.0, 312711.5, 312711.5, 1259979.0, 980195.8, 3920783.2, 980195.8, 3920783.2, 2399995.8, 9599983.2, 498195.8, 1992783.2, 195995.8, 783983.2, 651229.0, 751229.0, 576406.0, 1100406.0, 576406.0, 576406.0, 357748.2, 149061.75, 89437.05, 597000.0, 597000.0, 597000.0, 597000.0, 597000.0, 597000.0, 597000.0, 679979.0, 315828.0, 315828.0, 315828.0, 315828.0, 3006215.0, 375828.0, 375828.0, 375828.0, 375828.0, 1653215.0, 600311.0, 1503215.0, 751229.0, 673777.0, 600311.0, 600311.0, 577229.0, 577229.0, 1201229.0, 591000.0, 1801229.0, 627000.0, 1187926.0, 2507926.0, 576406.0, 1594229.0, 7300311.0, 951357.0, 774520.0, 531000.0, 1145478.0, 563000.0, 172506.75]}],
                        {"template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Net Lease Bids between 2014-2020"}, "xaxis": {"title": {"text": "Year"}}, "yaxis": {"title": {"text": "Net Lease Bid (USD)"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('ed0f6398-235e-4ce4-83d8-136b882f5c15');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };

            </script>
        </div>
</body>
</html>



```python
gom_clean_df.hist(figsize=(16,20), bins=15, xlabelsize=8, ylabelsize=8)
```




    array([[<matplotlib.axes._subplots.AxesSubplot object at 0x7f04c9414a50>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x7f04c91c3b50>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x7f04c9185210>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x7f04c913c890>],
           [<matplotlib.axes._subplots.AxesSubplot object at 0x7f04c90f2ed0>,
            <matplotlib.axes._subplots.AxesSubplot object at 0x7f04c90b3590>]],
          dtype=object)




    
![png](tf_experimental_design_capstone_files/tf_experimental_design_capstone_23_1.png)
    


## Data Exploration:

#### Import Seaborn for analysis. Change style to dark.


```python
import seaborn as sns
sns.set_style('dark')
```

#### Create dataframe grouped by 'clean_company' and 'year'. also returns 'mean_bid' as  column


```python
gom_clean_mean_comp_df = gom_clean_df.groupby(['clean_company', 'year'],as_index=False).net_bid.mean()
gom_clean_mean_comp_df = gom_clean_mean_comp_df.rename(columns={'net_bid': 'mean_bid'})
gom_clean_mean_comp_df.head(20)
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
      <th>clean_company</th>
      <th>year</th>
      <th>mean_bid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANADARKO</td>
      <td>2014</td>
      <td>1.354914e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ANADARKO</td>
      <td>2015</td>
      <td>1.054771e+06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ANADARKO</td>
      <td>2016</td>
      <td>7.500000e+05</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ANADARKO</td>
      <td>2017</td>
      <td>1.174756e+06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ANADARKO</td>
      <td>2018</td>
      <td>2.496378e+06</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ANADARKO</td>
      <td>2019</td>
      <td>1.327920e+06</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ANADARKO</td>
      <td>2020</td>
      <td>1.618839e+06</td>
    </tr>
    <tr>
      <th>7</th>
      <td>BEACON OFFSHORE ENERGY EXPLORATION LLC</td>
      <td>2018</td>
      <td>3.697685e+05</td>
    </tr>
    <tr>
      <th>8</th>
      <td>BEACON OFFSHORE ENERGY EXPLORATION LLC</td>
      <td>2019</td>
      <td>7.588155e+05</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BEACON OFFSHORE ENERGY EXPLORATION LLC</td>
      <td>2020</td>
      <td>2.660336e+05</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2014</td>
      <td>1.462080e+06</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2015</td>
      <td>8.439588e+05</td>
    </tr>
    <tr>
      <th>12</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2016</td>
      <td>3.042849e+06</td>
    </tr>
    <tr>
      <th>13</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2017</td>
      <td>3.745056e+05</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2018</td>
      <td>4.130860e+06</td>
    </tr>
    <tr>
      <th>15</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2019</td>
      <td>2.141915e+06</td>
    </tr>
    <tr>
      <th>16</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2020</td>
      <td>3.331367e+06</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BP EXPLORATION &amp; PRODUCTION INC</td>
      <td>2014</td>
      <td>1.474602e+06</td>
    </tr>
    <tr>
      <th>18</th>
      <td>BP EXPLORATION &amp; PRODUCTION INC</td>
      <td>2015</td>
      <td>1.059883e+06</td>
    </tr>
    <tr>
      <th>19</th>
      <td>BP EXPLORATION &amp; PRODUCTION INC</td>
      <td>2016</td>
      <td>8.399226e+05</td>
    </tr>
  </tbody>
</table>
</div>



#### Pivot table companing mean bid for each company between 2014-2020


```python
gom_clean_mean_comp_pivot = gom_clean_mean_comp_df.pivot(index='clean_company', columns='year', values='mean_bid')
gom_clean_mean_comp_pivot.head(12)
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
      <th>year</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>2018</th>
      <th>2019</th>
      <th>2020</th>
    </tr>
    <tr>
      <th>clean_company</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ANADARKO</th>
      <td>1.354914e+06</td>
      <td>1.054771e+06</td>
      <td>7.500000e+05</td>
      <td>1.174756e+06</td>
      <td>2.496378e+06</td>
      <td>1.327920e+06</td>
      <td>1.618839e+06</td>
    </tr>
    <tr>
      <th>BEACON OFFSHORE ENERGY EXPLORATION LLC</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.697685e+05</td>
      <td>7.588155e+05</td>
      <td>2.660336e+05</td>
    </tr>
    <tr>
      <th>BHP BILLITON PETROLEUM (DEEPWATER) INC</th>
      <td>1.462080e+06</td>
      <td>8.439588e+05</td>
      <td>3.042849e+06</td>
      <td>3.745056e+05</td>
      <td>4.130860e+06</td>
      <td>2.141915e+06</td>
      <td>3.331367e+06</td>
    </tr>
    <tr>
      <th>BP EXPLORATION &amp; PRODUCTION INC</th>
      <td>1.474602e+06</td>
      <td>1.059883e+06</td>
      <td>8.399226e+05</td>
      <td>5.418415e+05</td>
      <td>8.993085e+05</td>
      <td>6.804593e+05</td>
      <td>1.057090e+06</td>
    </tr>
    <tr>
      <th>CHEVRON USA INC</th>
      <td>1.586688e+07</td>
      <td>7.146583e+06</td>
      <td>2.024031e+06</td>
      <td>1.813836e+06</td>
      <td>1.769308e+06</td>
      <td>1.424382e+06</td>
      <td>1.424760e+06</td>
    </tr>
    <tr>
      <th>CL&amp;F OFFSHORE LLC</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.112109e+05</td>
      <td>7.848000e+04</td>
      <td>3.024115e+05</td>
      <td>6.284000e+04</td>
    </tr>
    <tr>
      <th>CSL EXPLORATION LP</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.887225e+05</td>
    </tr>
    <tr>
      <th>ENVEN ENERGY VENTURES LLC</th>
      <td>6.851770e+05</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.030000e+05</td>
      <td>8.543191e+05</td>
      <td>6.427941e+05</td>
      <td>7.542508e+05</td>
    </tr>
    <tr>
      <th>EQUINOR GULF OF MEXICO LLC</th>
      <td>1.048022e+06</td>
      <td>3.673357e+06</td>
      <td>1.603307e+06</td>
      <td>2.911800e+06</td>
      <td>8.216644e+05</td>
      <td>3.022304e+06</td>
      <td>2.147971e+06</td>
    </tr>
    <tr>
      <th>HESS CORPORATION</th>
      <td>1.149500e+06</td>
      <td>2.073237e+06</td>
      <td>2.165270e+06</td>
      <td>3.883558e+06</td>
      <td>2.792999e+06</td>
      <td>2.142789e+06</td>
      <td>1.847926e+06</td>
    </tr>
    <tr>
      <th>HOUSTON ENERGY LP</th>
      <td>2.485760e+05</td>
      <td>4.413414e+05</td>
      <td>7.029613e+04</td>
      <td>1.624241e+05</td>
      <td>2.668589e+05</td>
      <td>1.598152e+05</td>
      <td>1.432904e+05</td>
    </tr>
    <tr>
      <th>KOSMOS ENERGY GULF OF MEXICO OPERATIONS LLC</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.944773e+05</td>
      <td>3.127115e+05</td>
    </tr>
  </tbody>
</table>
</div>



plot mean bids, grouped by company, between 2014-2020


```python
plt.figure(figsize=(12,10))
plt.title(label='Mean Net Lease Sale Bid Per Company Between 2014-2020')
plt.ylabel(ylabel='Mean Net Lease Bid ($MM)')
plt.xlabel(xlabel='Year')
sns.lineplot(data=gom_clean_mean_comp_df, x='year', y='mean_bid', hue='clean_company', style='clean_company', markers=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f04c864e690>




    
![png](tf_experimental_design_capstone_files/tf_experimental_design_capstone_32_1.png)
    



```python
plt.figure(figsize=(15,13))
plt.title(label='Mean Net Lease Sale Bid Per Company Between 2014-2020')
plt.ylabel(ylabel='Mean Net Lease Bid ($MM)')
plt.xlabel(xlabel='Year')
sns.pointplot(data=gom_clean_df, x='year', y='net_bid', hue='clean_company')

```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f04c845d3d0>




    
![png](tf_experimental_design_capstone_files/tf_experimental_design_capstone_33_1.png)
    


Looking at median this time instead of mean. Create dataframe grouped by 'clean_company' and 'year'. also returns 'median_bid' as column


```python
gom_clean_median_comp_df = gom_clean_df.groupby(['clean_company', 'year'],as_index=False).net_bid.median()
gom_clean_median_comp_df = gom_clean_median_comp_df.rename(columns={'net_bid': 'median_bid'})
gom_clean_median_comp_df.head(20)
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
      <th>clean_company</th>
      <th>year</th>
      <th>median_bid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANADARKO</td>
      <td>2014</td>
      <td>1155500.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ANADARKO</td>
      <td>2015</td>
      <td>1203333.000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ANADARKO</td>
      <td>2016</td>
      <td>787500.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ANADARKO</td>
      <td>2017</td>
      <td>625260.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ANADARKO</td>
      <td>2018</td>
      <td>3003880.000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ANADARKO</td>
      <td>2019</td>
      <td>705505.000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ANADARKO</td>
      <td>2020</td>
      <td>1578215.000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>BEACON OFFSHORE ENERGY EXPLORATION LLC</td>
      <td>2018</td>
      <td>369768.500</td>
    </tr>
    <tr>
      <th>8</th>
      <td>BEACON OFFSHORE ENERGY EXPLORATION LLC</td>
      <td>2019</td>
      <td>514080.850</td>
    </tr>
    <tr>
      <th>9</th>
      <td>BEACON OFFSHORE ENERGY EXPLORATION LLC</td>
      <td>2020</td>
      <td>220412.500</td>
    </tr>
    <tr>
      <th>10</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2014</td>
      <td>1006250.000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2015</td>
      <td>608888.000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2016</td>
      <td>1077480.000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2017</td>
      <td>376063.164</td>
    </tr>
    <tr>
      <th>14</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2018</td>
      <td>4130860.000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2019</td>
      <td>910319.000</td>
    </tr>
    <tr>
      <th>16</th>
      <td>BHP BILLITON PETROLEUM (DEEPWATER) INC</td>
      <td>2020</td>
      <td>1304050.000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>BP EXPLORATION &amp; PRODUCTION INC</td>
      <td>2014</td>
      <td>1151027.000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>BP EXPLORATION &amp; PRODUCTION INC</td>
      <td>2015</td>
      <td>950216.000</td>
    </tr>
    <tr>
      <th>19</th>
      <td>BP EXPLORATION &amp; PRODUCTION INC</td>
      <td>2016</td>
      <td>621681.000</td>
    </tr>
  </tbody>
</table>
</div>




```python

```

Pivot table with index=clean_company and columns=year. values are median bids


```python
gom_clean_median_comp_pivot = gom_clean_median_comp_df.pivot(index='clean_company', columns='year', values='median_bid')
gom_clean_median_comp_pivot.head(12)
gom_clean_median_comp_pivot.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 21 entries, ANADARKO to W & T OFFSHORE INC
    Data columns (total 7 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   2014    16 non-null     float64
     1   2015    14 non-null     float64
     2   2016    13 non-null     float64
     3   2017    17 non-null     float64
     4   2018    17 non-null     float64
     5   2019    20 non-null     float64
     6   2020    21 non-null     float64
    dtypes: float64(7)
    memory usage: 1.3+ KB


plot median bids, grouped by company between 2014-2020


```python
plt.figure(figsize=(12,10))
plt.title(label='Median Net Lease Sale Bid Per Company Between 2014-2020')
plt.ylabel(ylabel='Median Net Lease Bid ($MM)')
plt.xlabel(xlabel='Year')
sns.lineplot(data=gom_clean_median_comp_df, x='year', y='median_bid', hue='clean_company', style='clean_company', markers=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f04c69705d0>




    
![png](tf_experimental_design_capstone_files/tf_experimental_design_capstone_40_1.png)
    



```python
fig = px.line(data_frame=gom_clean_median_comp_df, x='year', y='median_bid', color='clean_company', title='Median Lease Bid Price per Company Between 2014-2020', width=1200, height=800, hover_data=['clean_company']).for_each_trace(lambda t: t.update(name=t.name.split("=")[1]))
fig.show()
```


<html>
<head><meta charset="utf-8" /></head>
<body>
    <div>
            <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-AMS-MML_SVG"></script><script type="text/javascript">if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}</script>
                <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: 'local'};</script>
        <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>    
            <div id="f471f371-6520-4a45-b7d1-a436884dbfb1" class="plotly-graph-div" style="height:800px; width:1200px;"></div>
            <script type="text/javascript">

                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("f471f371-6520-4a45-b7d1-a436884dbfb1")) {
                    Plotly.newPlot(
                        'f471f371-6520-4a45-b7d1-a436884dbfb1',
                        [{"customdata": [["ANADARKO"], ["ANADARKO"], ["ANADARKO"], ["ANADARKO"], ["ANADARKO"], ["ANADARKO"], ["ANADARKO"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=ANADARKO", "line": {"color": "#636efa", "dash": "solid"}, "mode": "lines", "name": "ANADARKO", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [1155500.0, 1203333.0, 787500.0, 625260.0, 3003880.0, 705505.0, 1578215.0], "yaxis": "y"}, {"customdata": [["BEACON OFFSHORE ENERGY EXPLORATION LLC"], ["BEACON OFFSHORE ENERGY EXPLORATION LLC"], ["BEACON OFFSHORE ENERGY EXPLORATION LLC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=BEACON OFFSHORE ENERGY EXPLORATION LLC", "line": {"color": "#EF553B", "dash": "solid"}, "mode": "lines", "name": "BEACON OFFSHORE ENERGY EXPLORATION LLC", "showlegend": true, "type": "scatter", "x": [2018, 2019, 2020], "xaxis": "x", "y": [369768.5, 514080.85, 220412.5], "yaxis": "y"}, {"customdata": [["BHP BILLITON PETROLEUM (DEEPWATER) INC"], ["BHP BILLITON PETROLEUM (DEEPWATER) INC"], ["BHP BILLITON PETROLEUM (DEEPWATER) INC"], ["BHP BILLITON PETROLEUM (DEEPWATER) INC"], ["BHP BILLITON PETROLEUM (DEEPWATER) INC"], ["BHP BILLITON PETROLEUM (DEEPWATER) INC"], ["BHP BILLITON PETROLEUM (DEEPWATER) INC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=BHP BILLITON PETROLEUM (DEEPWATER) INC", "line": {"color": "#00cc96", "dash": "solid"}, "mode": "lines", "name": "BHP BILLITON PETROLEUM (DEEPWATER) INC", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [1006250.0, 608888.0, 1077480.0, 376063.164, 4130860.0, 910319.0, 1304050.0], "yaxis": "y"}, {"customdata": [["BP EXPLORATION & PRODUCTION INC"], ["BP EXPLORATION & PRODUCTION INC"], ["BP EXPLORATION & PRODUCTION INC"], ["BP EXPLORATION & PRODUCTION INC"], ["BP EXPLORATION & PRODUCTION INC"], ["BP EXPLORATION & PRODUCTION INC"], ["BP EXPLORATION & PRODUCTION INC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=BP EXPLORATION & PRODUCTION INC", "line": {"color": "#ab63fa", "dash": "solid"}, "mode": "lines", "name": "BP EXPLORATION & PRODUCTION INC", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [1151027.0, 950216.0, 621681.0, 576000.0, 610526.0, 615224.0, 576630.0], "yaxis": "y"}, {"customdata": [["CHEVRON USA INC"], ["CHEVRON USA INC"], ["CHEVRON USA INC"], ["CHEVRON USA INC"], ["CHEVRON USA INC"], ["CHEVRON USA INC"], ["CHEVRON USA INC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=CHEVRON USA INC", "line": {"color": "#FFA15A", "dash": "solid"}, "mode": "lines", "name": "CHEVRON USA INC", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [6589321.0, 625489.0, 718608.0, 1372908.0, 1268215.0, 886018.0, 755320.5], "yaxis": "y"}, {"customdata": [["CL&F OFFSHORE LLC"], ["CL&F OFFSHORE LLC"], ["CL&F OFFSHORE LLC"], ["CL&F OFFSHORE LLC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=CL&F OFFSHORE LLC", "line": {"color": "#19d3f3", "dash": "solid"}, "mode": "lines", "name": "CL&F OFFSHORE LLC", "showlegend": true, "type": "scatter", "x": [2017, 2018, 2019, 2020], "xaxis": "x", "y": [84848.5625, 78480.0, 302411.55, 62840.0], "yaxis": "y"}, {"customdata": [["CSL EXPLORATION LP"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=CSL EXPLORATION LP", "line": {"color": "#FF6692", "dash": "solid"}, "mode": "lines", "name": "CSL EXPLORATION LP", "showlegend": true, "type": "scatter", "x": [2020], "xaxis": "x", "y": [188722.5], "yaxis": "y"}, {"customdata": [["ENVEN ENERGY VENTURES LLC"], ["ENVEN ENERGY VENTURES LLC"], ["ENVEN ENERGY VENTURES LLC"], ["ENVEN ENERGY VENTURES LLC"], ["ENVEN ENERGY VENTURES LLC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=ENVEN ENERGY VENTURES LLC", "line": {"color": "#B6E880", "dash": "solid"}, "mode": "lines", "name": "ENVEN ENERGY VENTURES LLC", "showlegend": true, "type": "scatter", "x": [2014, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [685177.0, 603000.0, 702331.0, 589000.0, 597000.0], "yaxis": "y"}, {"customdata": [["EQUINOR GULF OF MEXICO LLC"], ["EQUINOR GULF OF MEXICO LLC"], ["EQUINOR GULF OF MEXICO LLC"], ["EQUINOR GULF OF MEXICO LLC"], ["EQUINOR GULF OF MEXICO LLC"], ["EQUINOR GULF OF MEXICO LLC"], ["EQUINOR GULF OF MEXICO LLC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=EQUINOR GULF OF MEXICO LLC", "line": {"color": "#FF97FF", "dash": "solid"}, "mode": "lines", "name": "EQUINOR GULF OF MEXICO LLC", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [1032986.0, 1413500.0, 1206557.0, 955976.0, 703976.0, 860776.0, 771329.6], "yaxis": "y"}, {"customdata": [["HESS CORPORATION"], ["HESS CORPORATION"], ["HESS CORPORATION"], ["HESS CORPORATION"], ["HESS CORPORATION"], ["HESS CORPORATION"], ["HESS CORPORATION"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=HESS CORPORATION", "line": {"color": "#FECB52", "dash": "solid"}, "mode": "lines", "name": "HESS CORPORATION", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [1149500.0, 2120237.0, 2165270.0, 1780805.0, 851889.0, 799449.0, 1847926.0], "yaxis": "y"}, {"customdata": [["HOUSTON ENERGY LP"], ["HOUSTON ENERGY LP"], ["HOUSTON ENERGY LP"], ["HOUSTON ENERGY LP"], ["HOUSTON ENERGY LP"], ["HOUSTON ENERGY LP"], ["HOUSTON ENERGY LP"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=HOUSTON ENERGY LP", "line": {"color": "#636efa", "dash": "solid"}, "mode": "lines", "name": "HOUSTON ENERGY LP", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [120088.07500000001, 45773.9, 40533.327000000005, 40909.0, 156960.0, 102026.7, 131656.0], "yaxis": "y"}, {"customdata": [["KOSMOS ENERGY GULF OF MEXICO OPERATIONS LLC"], ["KOSMOS ENERGY GULF OF MEXICO OPERATIONS LLC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=KOSMOS ENERGY GULF OF MEXICO OPERATIONS LLC", "line": {"color": "#EF553B", "dash": "solid"}, "mode": "lines", "name": "KOSMOS ENERGY GULF OF MEXICO OPERATIONS LLC", "showlegend": true, "type": "scatter", "x": [2019, 2020], "xaxis": "x", "y": [310858.0, 312711.5], "yaxis": "y"}, {"customdata": [["LLOG"], ["LLOG"], ["LLOG"], ["LLOG"], ["LLOG"], ["LLOG"], ["LLOG"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=LLOG", "line": {"color": "#00cc96", "dash": "solid"}, "mode": "lines", "name": "LLOG", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [576000.0, 640834.6, 426306.3, 382882.5, 486888.5, 610000.0, 375828.0], "yaxis": "y"}, {"customdata": [["MURPHY EXPLORATION & PRODUCTION COMPANY - USA"], ["MURPHY EXPLORATION & PRODUCTION COMPANY - USA"], ["MURPHY EXPLORATION & PRODUCTION COMPANY - USA"], ["MURPHY EXPLORATION & PRODUCTION COMPANY - USA"], ["MURPHY EXPLORATION & PRODUCTION COMPANY - USA"], ["MURPHY EXPLORATION & PRODUCTION COMPANY - USA"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=MURPHY EXPLORATION & PRODUCTION COMPANY - USA", "line": {"color": "#ab63fa", "dash": "solid"}, "mode": "lines", "name": "MURPHY EXPLORATION & PRODUCTION COMPANY - USA", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [3001125.0, 1049060.0, 512807.5, 364342.6, 585280.9, 576406.0], "yaxis": "y"}, {"customdata": [["RED WILLOW OFFSHORE LLC"], ["RED WILLOW OFFSHORE LLC"], ["RED WILLOW OFFSHORE LLC"], ["RED WILLOW OFFSHORE LLC"], ["RED WILLOW OFFSHORE LLC"], ["RED WILLOW OFFSHORE LLC"], ["RED WILLOW OFFSHORE LLC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=RED WILLOW OFFSHORE LLC", "line": {"color": "#FFA15A", "dash": "solid"}, "mode": "lines", "name": "RED WILLOW OFFSHORE LLC", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [715036.4554, 409851.9, 367411.653, 188941.25, 267950.78740000003, 346454.0, 1083544.875], "yaxis": "y"}, {"customdata": [["REPSOL E&P USA INC"], ["REPSOL E&P USA INC"], ["REPSOL E&P USA INC"], ["REPSOL E&P USA INC"], ["REPSOL E&P USA INC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=REPSOL E&P USA INC", "line": {"color": "#19d3f3", "dash": "solid"}, "mode": "lines", "name": "REPSOL E&P USA INC", "showlegend": true, "type": "scatter", "x": [2014, 2016, 2017, 2019, 2020], "xaxis": "x", "y": [299596.4, 752003.0, 115000.0, 401974.5, 375828.0], "yaxis": "y"}, {"customdata": [["RIDGEWOOD ENERGY CORPORATION"], ["RIDGEWOOD ENERGY CORPORATION"], ["RIDGEWOOD ENERGY CORPORATION"], ["RIDGEWOOD ENERGY CORPORATION"], ["RIDGEWOOD ENERGY CORPORATION"], ["RIDGEWOOD ENERGY CORPORATION"], ["RIDGEWOOD ENERGY CORPORATION"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=RIDGEWOOD ENERGY CORPORATION", "line": {"color": "#FF6692", "dash": "solid"}, "mode": "lines", "name": "RIDGEWOOD ENERGY CORPORATION", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [558476.8509, 1771500.0, 422922.5, 320945.625, 362750.0, 388561.5, 374887.5], "yaxis": "y"}, {"customdata": [["SHELL OFFSHORE INC."], ["SHELL OFFSHORE INC."], ["SHELL OFFSHORE INC."], ["SHELL OFFSHORE INC."], ["SHELL OFFSHORE INC."], ["SHELL OFFSHORE INC."], ["SHELL OFFSHORE INC."]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=SHELL OFFSHORE INC.", "line": {"color": "#B6E880", "dash": "solid"}, "mode": "lines", "name": "SHELL OFFSHORE INC.", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [750000.0, 725725.0, 900124.0, 900719.0, 771988.0, 801988.0, 1201229.0], "yaxis": "y"}, {"customdata": [["TALOS ENERGY"], ["TALOS ENERGY"], ["TALOS ENERGY"], ["TALOS ENERGY"], ["TALOS ENERGY"], ["TALOS ENERGY"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=TALOS ENERGY", "line": {"color": "#FF97FF", "dash": "solid"}, "mode": "lines", "name": "TALOS ENERGY", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [546150.0, 813897.5, 585150.0, 587427.0, 582450.0, 180043.25], "yaxis": "y"}, {"customdata": [["TOTAL E&P USA INC"], ["TOTAL E&P USA INC"], ["TOTAL E&P USA INC"], ["TOTAL E&P USA INC"], ["TOTAL E&P USA INC"], ["TOTAL E&P USA INC"], ["TOTAL E&P USA INC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=TOTAL E&P USA INC", "line": {"color": "#FECB52", "dash": "solid"}, "mode": "lines", "name": "TOTAL E&P USA INC", "showlegend": true, "type": "scatter", "x": [2014, 2015, 2016, 2017, 2018, 2019, 2020], "xaxis": "x", "y": [300564.8, 324647.2, 382929.6, 825227.0, 950248.0, 3451608.5, 1016816.0], "yaxis": "y"}, {"customdata": [["W & T OFFSHORE INC"], ["W & T OFFSHORE INC"]], "hoverlabel": {"namelength": 0}, "hovertemplate": "clean_company=%{customdata[0]}<br>year=%{x}<br>median_bid=%{y}", "legendgroup": "clean_company=W & T OFFSHORE INC", "line": {"color": "#636efa", "dash": "solid"}, "mode": "lines", "name": "W & T OFFSHORE INC", "showlegend": true, "type": "scatter", "x": [2019, 2020], "xaxis": "x", "y": [250000.0, 576000.0], "yaxis": "y"}],
                        {"height": 800, "legend": {"tracegroupgap": 0}, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Median Lease Bid Price per Company Between 2014-2020"}, "width": 1200, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "year"}}, "yaxis": {"anchor": "x", "domain": [0.0, 1.0], "title": {"text": "median_bid"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('f471f371-6520-4a45-b7d1-a436884dbfb1');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };

            </script>
        </div>
</body>
</html>



```python
gom_clean_mean_area_df = gom_clean_df.groupby(['clean_area', 'year'],as_index=False).net_bid.mean()
gom_clean_mean_area_df = gom_clean_mean_area_df.rename(columns={'net_bid': 'mean_bid'})
gom_clean_mean_area_df.head(20)
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
      <th>clean_area</th>
      <th>year</th>
      <th>mean_bid</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alaminos Canyon</td>
      <td>2014</td>
      <td>1.652942e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alaminos Canyon</td>
      <td>2015</td>
      <td>6.088880e+05</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Alaminos Canyon</td>
      <td>2016</td>
      <td>9.008133e+05</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alaminos Canyon</td>
      <td>2017</td>
      <td>2.197352e+06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alaminos Canyon</td>
      <td>2018</td>
      <td>1.478622e+06</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Alaminos Canyon</td>
      <td>2019</td>
      <td>1.609189e+06</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Alaminos Canyon</td>
      <td>2020</td>
      <td>1.255317e+06</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Atwater Valley</td>
      <td>2014</td>
      <td>1.751100e+06</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Atwater Valley</td>
      <td>2015</td>
      <td>1.099271e+06</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Atwater Valley</td>
      <td>2016</td>
      <td>2.082080e+05</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Atwater Valley</td>
      <td>2017</td>
      <td>3.323054e+06</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Atwater Valley</td>
      <td>2018</td>
      <td>4.549420e+05</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Atwater Valley</td>
      <td>2019</td>
      <td>8.061249e+05</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Atwater Valley</td>
      <td>2020</td>
      <td>6.736971e+05</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Garden Banks</td>
      <td>2014</td>
      <td>6.644522e+05</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Garden Banks</td>
      <td>2015</td>
      <td>9.325114e+05</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Garden Banks</td>
      <td>2016</td>
      <td>6.207213e+05</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Garden Banks</td>
      <td>2017</td>
      <td>1.724607e+06</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Garden Banks</td>
      <td>2018</td>
      <td>1.781582e+06</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Garden Banks</td>
      <td>2019</td>
      <td>7.734629e+05</td>
    </tr>
  </tbody>
</table>
</div>




```python
gom_clean_mean_area_pivot = gom_clean_mean_area_df.pivot(index='clean_area', columns='year', values='mean_bid')
gom_clean_mean_area_pivot.head(7)
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
      <th>year</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>2018</th>
      <th>2019</th>
      <th>2020</th>
    </tr>
    <tr>
      <th>clean_area</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Alaminos Canyon</th>
      <td>1.652942e+06</td>
      <td>6.088880e+05</td>
      <td>9.008133e+05</td>
      <td>2.197352e+06</td>
      <td>1.478622e+06</td>
      <td>1.609189e+06</td>
      <td>1.255317e+06</td>
    </tr>
    <tr>
      <th>Atwater Valley</th>
      <td>1.751100e+06</td>
      <td>1.099271e+06</td>
      <td>2.082080e+05</td>
      <td>3.323054e+06</td>
      <td>4.549420e+05</td>
      <td>8.061249e+05</td>
      <td>6.736971e+05</td>
    </tr>
    <tr>
      <th>Garden Banks</th>
      <td>6.644522e+05</td>
      <td>9.325114e+05</td>
      <td>6.207213e+05</td>
      <td>1.724607e+06</td>
      <td>1.781582e+06</td>
      <td>7.734629e+05</td>
      <td>8.557376e+05</td>
    </tr>
    <tr>
      <th>Green Canyon</th>
      <td>1.314117e+06</td>
      <td>3.916824e+06</td>
      <td>9.796576e+05</td>
      <td>1.116057e+06</td>
      <td>9.217483e+05</td>
      <td>1.134563e+06</td>
      <td>1.641427e+06</td>
    </tr>
    <tr>
      <th>Keathley Canyon</th>
      <td>7.096310e+05</td>
      <td>1.394466e+06</td>
      <td>7.796458e+05</td>
      <td>1.325181e+06</td>
      <td>8.101580e+05</td>
      <td>1.199269e+06</td>
      <td>6.452818e+05</td>
    </tr>
    <tr>
      <th>Mississippi Canyon</th>
      <td>5.007435e+06</td>
      <td>2.360593e+06</td>
      <td>1.185099e+06</td>
      <td>1.304549e+06</td>
      <td>1.896590e+06</td>
      <td>1.778580e+06</td>
      <td>1.235954e+06</td>
    </tr>
    <tr>
      <th>Walker Ridge</th>
      <td>6.044766e+05</td>
      <td>4.287633e+06</td>
      <td>5.025201e+05</td>
      <td>2.230396e+06</td>
      <td>9.651852e+05</td>
      <td>8.898308e+05</td>
      <td>1.802950e+06</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.figure(figsize=(12,10))
plt.title(label='Mean Lease Sale Bid per Protraction Area Between 2014-2020')
plt.ylabel(ylabel='Mean Lease Bid ($MM)')
plt.xlabel(xlabel='Year')
sns.lineplot(data=gom_clean_mean_area_df, x='year', y='mean_bid', hue='clean_area', style='clean_area', markers=True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x7f04c67df6d0>




    
![png](tf_experimental_design_capstone_files/tf_experimental_design_capstone_44_1.png)
    



```python
 fig = px.line(data_frame=gom_clean_median_area_df, x='year', y='median_bid', width=1000, color='clean_area', title='Median Lease Bid Price per Protraction Area Between 2014-2020', animation_group='clean_area', labels={'median_bid': 'Median Lease Bid (USD)'})
 fig.show()
```


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-32-798172f63a5c> in <module>()
    ----> 1 fig = px.line(data_frame=gom_clean_median_area_df, x='year', y='median_bid', width=1000, color='clean_area', title='Median Lease Bid Price per Protraction Area Between 2014-2020', animation_group='clean_area', labels={'median_bid': 'Median Lease Bid (USD)'})
          2 fig.show()


    NameError: name 'gom_clean_median_area_df' is not defined


Looking at median of net bids per area. Create dataframe grouped by 'clean_area' and 'year'. also returns 'median_bid' as column


```python
gom_clean_median_area_df = gom_clean_df.groupby(['clean_area', 'year'],as_index=False).net_bid.median()
gom_clean_median_area_df = gom_clean_median_area_df.rename(columns={'net_bid': 'median_bid'})
gom_clean_median_area_df.head(20)
```

Pivot table for median dataframe. index=clean_area, columns=year, values=median_bid


```python
gom_clean_median_area_pivot = gom_clean_median_area_df.pivot(index='clean_area', columns='year', values='median_bid')
gom_clean_median_area_pivot.head(7)
```

plot median bids, grouped by area between 2014-2020


```python
plt.figure(figsize=(12,10))
plt.title(label='Median Lease Sale Bid per Protraction Area Between 2014-2020')
plt.ylabel(ylabel='Median Lease Bid ($MM)')
plt.xlabel(xlabel='Year')
sns.lineplot(data=gom_clean_median_area_df, x='year', y='median_bid', hue='clean_area', style='clean_area', markers=True)
```

Methods:



```python
# bid statistics per company per year
grouped_year_company_df = gom_clean_df.groupby(['year','clean_company']).agg({'net_bid': ['mean','median', 'min', 'max', 'count']})
grouped_year_company_df.columns = ['bid_mean', 'bid_median', 'bid_min', 'bid_max', 'bid_count']
grouped_year_company_df.head(30)
```

Line graphs for mean bid per year for each company in list


```python
# line graphs for mean bid per year per company
#grouped_year_company_df.sort_values(by ='clean_company', ascending=False).head(10).plot(kind='bar') #top 10 companies 
temp= gom_clean_df.clean_company.unique()
df= grouped_year_company_df.reset_index()
for c in temp: 
  print('company:', c)
  df[df.clean_company==c].groupby('year')['bid_mean','bid_median', 'bid_min', 'bid_max'].mean().plot(kind='line')
  plt.title(label=c)
  plt.xlabel(xlabel='Year')
  plt.ylabel(ylabel='Net Bid Amount ($MM)')
  plt.show()
```


```python
# bid statistics per company per year
grouped_year_area_df = gom_clean_df.groupby(['year','clean_area']).agg({'net_bid': ['mean', 'median', 'min', 'max', 'count']})
grouped_year_area_df.columns = ['bid_mean', 'bid_median', 'bid_min', 'bid_max', 'bid_count']
grouped_year_area_df.head(30)
```


```python
# bar and ine graphs for mean bid per area per company
#grouped_year_area_df.sort_values(by ='clean_company', ascending=False).head(10).plot(kind='line') #top 10 companies 
temp= gom_clean_df.clean_area.unique()
df= grouped_year_area_df.reset_index()
for c in temp: 
  print('area:', c)
  df[df.clean_area==c].groupby('year')['bid_mean', 'bid_median', 'bid_min', 'bid_max'].mean().plot(kind='line')
  plt.ylabel(ylabel='Bid Amount ($MM)')
  plt.title(label=c)
  plt.show()
```


```python
gom_2014_df = gom_clean_df[gom_clean_df['year'] == 2014]
gom_2015_df = gom_clean_df[gom_clean_df['year'] == 2015]
gom_2016_df = gom_clean_df[gom_clean_df['year'] == 2016]
gom_2017_df = gom_clean_df[gom_clean_df['year'] == 2017]
gom_2018_df = gom_clean_df[gom_clean_df['year'] == 2018]
gom_2019_df = gom_clean_df[gom_clean_df['year'] == 2019]
gom_2020_df = gom_clean_df[gom_clean_df['year'] == 2020]
```


```python
# mean bids per year per company
gom_14_net_mean =  gom_2014_df.groupby(['clean_company'],as_index=False).net_bid.mean()
gom_15_net_mean =  gom_2015_df.groupby(['clean_company'],as_index=False).net_bid.mean()
gom_16_net_mean =  gom_2016_df.groupby(['clean_company'],as_index=False).net_bid.mean()
gom_17_net_mean =  gom_2017_df.groupby(['clean_company'],as_index=False).net_bid.mean()
gom_18_net_mean =  gom_2018_df.groupby(['clean_company'],as_index=False).net_bid.mean()
gom_19_net_mean =  gom_2019_df.groupby(['clean_company'],as_index=False).net_bid.mean()
gom_20_net_mean =  gom_2020_df.groupby(['clean_company'],as_index=False).net_bid.mean()
```


```python
gom_14_net_mean.head()
```


```python
plt.hist(gom_14_net_mean['net_bid'], alpha = .3)
plt.hist(gom_15_net_mean['net_bid'], alpha = .3)
plt.hist(gom_16_net_mean['net_bid'], alpha = .3)
plt.hist(gom_17_net_mean['net_bid'], alpha = .3)
plt.hist(gom_18_net_mean['net_bid'], alpha = .3)
plt.hist(gom_19_net_mean['net_bid'], alpha = .3)
plt.hist(gom_20_net_mean['net_bid'], alpha = .3)
plt.legend(['2014','2015','2016','2017','2018','2019','2020'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Mean Net Bids for 2014-2020')
plt.show()
```


```python
means = px.histogram(gom_clean_df, x='net_bid', color='year', title='Net Lease Bids per Year', opacity=.5)
means.show()
```


```python
print(stats.describe(gom_14_net_mean['net_bid']))
print(stats.describe(gom_15_net_mean['net_bid']))
print(stats.describe(gom_16_net_mean['net_bid']))
print(stats.describe(gom_17_net_mean['net_bid']))
print(stats.describe(gom_18_net_mean['net_bid']))
print(stats.describe(gom_19_net_mean['net_bid']))
print(stats.describe(gom_20_net_mean['net_bid']))
```

Comparing median bids per year instead of mean bids


```python
# median bids per year per company
gom_14_net_median =  gom_2014_df.groupby(['clean_company'],as_index=False).net_bid.median()
gom_15_net_median =  gom_2015_df.groupby(['clean_company'],as_index=False).net_bid.median()
gom_16_net_median =  gom_2016_df.groupby(['clean_company'],as_index=False).net_bid.median()
gom_17_net_median =  gom_2017_df.groupby(['clean_company'],as_index=False).net_bid.median()
gom_18_net_median =  gom_2018_df.groupby(['clean_company'],as_index=False).net_bid.median()
gom_19_net_median =  gom_2019_df.groupby(['clean_company'],as_index=False).net_bid.median()
gom_20_net_median =  gom_2020_df.groupby(['clean_company'],as_index=False).net_bid.median()
```


```python
gom_14_net_median.head()
```


```python
plt.hist(gom_14_net_median['net_bid'], alpha = .3)
plt.hist(gom_15_net_median['net_bid'], alpha = .3)
plt.hist(gom_16_net_median['net_bid'], alpha = .3)
plt.hist(gom_17_net_median['net_bid'], alpha = .3)
plt.hist(gom_18_net_median['net_bid'], alpha = .3)
plt.hist(gom_19_net_median['net_bid'], alpha = .3)
plt.hist(gom_20_net_median['net_bid'], alpha = .3)
plt.legend(['2014','2015','2016','2017','2018','2019','2020'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Median Net Bids for 2014-2020')
plt.show()
```


```python
fig2 = px.scatter(gom_clean_df, x="net_bid", y="year",
	         size="net_bid", color="clean_company",
                 hover_name="clean_company", size_max=60, width=1000)
fig2.show()
```


```python
print(stats.describe(gom_14_net_median['net_bid']))
print(stats.describe(gom_15_net_median['net_bid']))
print(stats.describe(gom_16_net_median['net_bid']))
print(stats.describe(gom_17_net_median['net_bid']))
print(stats.describe(gom_18_net_median['net_bid']))
print(stats.describe(gom_19_net_median['net_bid']))
print(stats.describe(gom_20_net_median['net_bid']))
```

It appears that results from 2014 are not normal, driven by one outlier bid. 2015-2020 appear to be normal skewness & kurtosis between -3 & 3.

Mean net bids seem relatively normal, with the exception of the 2014 distribution. There appears to be another outlier bid in 2015. This makes sense because oil was hovering around 100 dollars a barrel until September 2014. The second lease sale of the year occured in August 2014. With oil prices high companies were likely more willing to submit large bids for blocks of interest. In 2015, there is another outlier bid relative to the rest of the distribution. Budget cuts may not have taken effect by the first lease sale of the year in March. In 2016 oil dropped below 50 dollars a barrel and the industry was hemorraging cash. All companies cut budgets and laid off large numbers of workers. This can be seen in the mean bid data with mean bids across companies becoming more similar.

kruskal for means, data not normal


```python
stats.kruskal(gom_14_net_mean['net_bid'], gom_15_net_mean['net_bid'],gom_16_net_mean['net_bid'], gom_17_net_mean['net_bid'], gom_18_net_mean['net_bid'], gom_19_net_mean['net_bid'], gom_20_net_mean['net_bid'])
```

Means result:  reject null hypothesis. test statistic > 1.96. significant difference in mean lease sale bids between 2014-2020

kruskal for medians, data not normal


```python
stats.kruskal(gom_14_net_median['net_bid'], gom_15_net_median['net_bid'],gom_16_net_median['net_bid'], gom_17_net_median['net_bid'], gom_18_net_median['net_bid'], gom_19_net_median['net_bid'], gom_20_net_median['net_bid'])
```

Medians result: reject null hypothesis. test statistic > 1.96. significant difference in median lease sale bids between 2014-2020

pairwise test for mean bids


```python
from statsmodels.stats.multicomp import pairwise_tukeyhsd

tukey_mean = pairwise_tukeyhsd(endog = gom_clean_mean_comp_df['mean_bid'], # comparing mean bid data
                          groups= gom_clean_mean_comp_df['clean_company'], #group by company
                          alpha = 0.05) # significance level
tukey_mean.summary()
```

Pairwise for medians grouped by company


```python
tukey_median = pairwise_tukeyhsd(endog = gom_clean_median_comp_df['median_bid'], # comparing mean bid data
                          groups= gom_clean_mean_comp_df['clean_company'], #group by company
                          alpha = 0.05) # significance level
tukey_median.summary()
```

## Look at meta bid price for top two companies in each year. Will give granular understanding of year wise price without using a point estimate.

## Hypothesis:

* Ho: There is no significant difference in the net bid (USD) for the top two companies in any year.
* Ha: There is a significant difference in the net bid (USD) for the top two companies in any year.


```python
gom_14_net_count =  gom_2014_df.groupby(['clean_company'],as_index=False).net_bid.count()
gom_15_net_count =  gom_2015_df.groupby(['clean_company'],as_index=False).net_bid.count()
gom_16_net_count =  gom_2016_df.groupby(['clean_company'],as_index=False).net_bid.count()
gom_17_net_count =  gom_2017_df.groupby(['clean_company'],as_index=False).net_bid.count()
gom_18_net_count =  gom_2018_df.groupby(['clean_company'],as_index=False).net_bid.count()
gom_19_net_count =  gom_2019_df.groupby(['clean_company'],as_index=False).net_bid.count()
gom_20_net_count =  gom_2020_df.groupby(['clean_company'],as_index=False).net_bid.count()
```

Bid count per company between 2014-2020


```python
# Number of bids submitted per company per year
gom_clean_count_comp_df = gom_clean_df.groupby(['clean_company', 'year'],as_index=False).net_bid.count()
gom_clean_count_comp_df = gom_clean_count_comp_df.rename(columns={'net_bid': 'bid_count'})
gom_clean_count_comp_df.head(20)
```


```python
gom_clean_count_comp_pivot = gom_clean_count_comp_df.pivot(index='clean_company', columns='year', values='bid_count')
gom_clean_count_comp_pivot.head(12)
```


```python
plt.figure(figsize=(14,12))
plt.title(label='Bid Count per Company Between 2014-2020')
plt.ylabel(ylabel='# of Bids')
plt.xlabel(xlabel='Year')
sns.lineplot(data=gom_clean_count_comp_df, x='year', y='bid_count', hue='clean_company', style='clean_company', markers=True)
```


```python
# top 2 bid count companies from 2014
gom_2014_bp = gom_2014_df[gom_2014_df['clean_company'] == 'BP EXPLORATION & PRODUCTION INC']
gom_2014_llog = gom_2014_df[gom_2014_df['clean_company'] == 'LLOG']
```


```python
plt.hist(gom_2014_bp['net_bid'], alpha = .3)
plt.hist(gom_2014_llog['net_bid'], alpha = .3)
plt.legend(['BP', 'LLOG'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2014')
plt.show()
```


```python
print(stats.describe(gom_2014_bp['net_bid']))
print(stats.describe(gom_2014_llog['net_bid']))
```


```python
stats.kruskal(gom_2014_bp['net_bid'], gom_2014_llog['net_bid'])
```

2014 results: reject null bc p value less than .05


```python
# top 2 bid count companies from 2015
gom_2015_bhp = gom_2015_df[gom_2015_df['clean_company'] == 'BHP BILLITON PETROLEUM (DEEPWATER) INC']
gom_2015_shell = gom_2015_df[gom_2015_df['clean_company'] == 'SHELL OFFSHORE INC.']
```


```python
plt.hist(gom_2015_bhp['net_bid'], alpha = .3)
plt.hist(gom_2015_shell['net_bid'], alpha = .3)
plt.legend(['BHP', 'Shell'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2015')
plt.show()
```


```python
print(stats.describe(gom_2015_bhp['net_bid']))
print(stats.describe(gom_2015_shell['net_bid']))
```

skewness and kurtosis within -3 and 3, run t-test


```python
stats.ttest_ind(gom_2015_bhp['net_bid'], gom_2015_shell['net_bid'])
```

2015 result: reject null hypothesis bc p-value less than .05


```python
# top 2 bid count companies from 2016
gom_2016_bp = gom_2016_df[gom_2016_df['clean_company'] == 'BP EXPLORATION & PRODUCTION INC']
gom_2016_hou = gom_2016_df[gom_2016_df['clean_company'] == 'HOUSTON ENERGY LP']
```


```python
plt.hist(gom_2016_bp['net_bid'], alpha = .3)
plt.hist(gom_2016_hou['net_bid'], alpha = .3)
plt.legend(['BP', 'Houston Energy'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2016')
plt.show()
```


```python
print(stats.describe(gom_2016_bp['net_bid']))
print(stats.describe(gom_2016_hou['net_bid']))
```

distribution is not normal -> kruskal wallis


```python
stats.kruskal(gom_2016_bp['net_bid'], gom_2016_hou['net_bid'])
```

2016 result: reject null. test statistic greater than 1.96 and p value less than .05


```python
# top 2 bid count companies from 2017
gom_2017_shell = gom_2017_df[gom_2017_df['clean_company'] == 'SHELL OFFSHORE INC.']
gom_2017_cvx = gom_2017_df[gom_2017_df['clean_company'] == 'CHEVRON USA INC']
```


```python
plt.hist(gom_2017_shell['net_bid'], alpha = .3)
plt.hist(gom_2017_cvx['net_bid'], alpha = .3)
plt.legend(['Shell', 'Chevron'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2017')
plt.show()
```


```python
print(stats.describe(gom_2017_shell['net_bid']))
print(stats.describe(gom_2017_cvx['net_bid']))
```

distribution is not normal -> kruskal wallis


```python
stats.kruskal(gom_2017_shell['net_bid'], gom_2017_cvx['net_bid'])
```

2017 result: accept null hypothesis


```python
# top 2 bid count companies from 2018
gom_2018_eqnr = gom_2018_df[gom_2018_df['clean_company'] == 'EQUINOR GULF OF MEXICO LLC']
gom_2018_cvx = gom_2018_df[gom_2018_df['clean_company'] == 'CHEVRON USA INC']
```


```python
plt.hist(gom_2018_eqnr['net_bid'], alpha = .3)
plt.hist(gom_2018_cvx['net_bid'], alpha = .3)
plt.legend(['Equinor', 'Chevron'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2018')
plt.show()
```


```python
print(stats.describe(gom_2018_eqnr['net_bid']))
print(stats.describe(gom_2018_cvx['net_bid']))
```

distribution is not normal -> kruskal wallis


```python
stats.kruskal(gom_2018_eqnr['net_bid'], gom_2018_cvx['net_bid'])
```

2018 result: reject null bc test statistic > 1.96 and p-value < .05


```python
# top 2 bid count companies from 2019
gom_2019_shell = gom_2019_df[gom_2019_df['clean_company'] == 'SHELL OFFSHORE INC.']
gom_2019_apc = gom_2019_df[gom_2019_df['clean_company'] == 'ANADARKO']
```


```python
plt.hist(gom_2019_shell['net_bid'], alpha = .3)
plt.hist(gom_2019_apc['net_bid'], alpha = .3)
plt.legend(['Shell', 'Andarko'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2019')
plt.show()
```


```python
print(stats.describe(gom_2019_shell['net_bid']))
print(stats.describe(gom_2019_apc['net_bid']))
```

distribution is not normal -> kruskal wallis


```python
stats.kruskal(gom_2019_shell['net_bid'], gom_2019_apc['net_bid'])
```

2019 result: accept null hypothesis


```python
# top 2 bid count companies from 2020
gom_2020_bp = gom_clean_df[gom_clean_df['clean_company'] == 'BP EXPLORATION & PRODUCTION INC']
gom_2020_shell = gom_clean_df[gom_clean_df['clean_company'] == 'SHELL OFFSHORE INC.']
```


```python
plt.hist(gom_2020_bp['net_bid'], alpha = .3)
plt.hist(gom_2020_shell['net_bid'], alpha = .3)
plt.legend(['BP', 'Shell'])
plt.xlabel(xlabel='Bid Amount ($MM)')
plt.ylabel(ylabel='# of Bids')
plt.title(label='Comparing Top 2 Companies bids in 2020')
plt.show()
```


```python
print(stats.describe(gom_2020_bp['net_bid']))
print(stats.describe(gom_2020_shell['net_bid']))
```

distribution is not normal -> kruskal wallis


```python
stats.kruskal(gom_2020_bp['net_bid'], gom_2020_shell['net_bid'])
```

2020 result: reject null hypothesis bc test statistic > 1.96 and p-value < .05

## Discussion and Conclusions:

##### After performing the one way ANOVA test the p-value is .499, which is greater than .05. We can assume that the means being compared are similar. When looking at the line plots of the mean bids over the given time frame, the mean values plot similarly. 

##### This dataset is interesting beause it captures two major oil price collapses (2015 & 2020). The means may be similar form 2015-2020 because companies became more disciplined in spending in order to conserve money during the downturns. The lease sale in 2014 shows a larger amount and distribution of lease bids submitted possibly due to high oil prices.

## Future Work:

*   Comparing median to account for outlier bids. Should we think about these bids similar to the homeprices example? One really high bid is not an invalid point. Quickly tested this in the above section. comparing mean bid to median bid. 
*   Comparing bid count per company and per area between 2014-2020. This might show emerging exploration areas better than mean bid amount. See below!
*   adding average oil price per year to main dataframe
*   scraping budgets from companies to compare
*   comparing average stock price for publicly traded companies with mean bid or # of bids
*   permits for exploration wells filed by company 
*   total volumes produced per company to measure exploration success
*   number of leases available vs num bids submitted

## Possible bias:
*   sampling bias: In order to complete a more thorough test I needed eliminate companies that did not participate in all 7 years of lease sales. Some of the companies I had to remove were private equity firms that may not have been constrained by a budget scrutinized by the investment community. Could I change null values to zero and rerun analysis?
*   contextual bias: The big one is likely the effect of oil price, which would cause budgets to tighten and making companies less likely to spend more money on lease bids.







## Future Work Testing:

Bid count per area between 2014-2020


```python
# Number of bids submitted per area per year
gom_clean_count_area_df = gom_clean_df.groupby(['clean_area', 'year'],as_index=False).net_bid.count()
gom_clean_count_area_df = gom_clean_count_area_df.rename(columns={'net_bid': 'bid_count'})
gom_clean_count_area_df.head(20)
```


```python
gom_clean_count_area_pivot = gom_clean_count_area_df.pivot(index='clean_area', columns='year', values='bid_count')
gom_clean_count_area_pivot.head(12)
```


```python
plt.figure(figsize=(14,12))
plt.title(label='Bid Count per Protraction Area Between 2014-2020')
plt.ylabel(ylabel='# of Bids')
plt.xlabel(xlabel='Year')
sns.lineplot(data=gom_clean_count_area_df, x='year', y='bid_count', hue='clean_area', style='clean_area', markers=True)
```
