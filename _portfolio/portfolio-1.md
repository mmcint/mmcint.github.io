---
title: "Portfolio item number 1"
excerpt: "Short description of portfolio item number 1<br/><img src='/images/500x300.png'>"
collection: portfolio
---

This is an item in your portfolio. It can be have images or nice text. If you name the file .md, it will be parsed as markdown. If you name the file .html, it will be parsed as HTML. 

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

![test image](../images/image-alignment-580x300.jpg)
