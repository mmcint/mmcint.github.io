---
title: 'Scraping Fantasy Football Stats'
date: 2024-12-13
permalink: /posts/2024/12/blog-post-1/
tags:
  - web scraping
  - fantasy football
  - Sports Analytics
---

## Scraping Fantasy Data - December 13th, 2024

This is a notebook that will scrape advanced stats for offensive fantasy football positions. The website used to scrape is FantasyPros. There are some additional metrics calculated after the scraping of the data. The intent is for this to be the first of many notebooks shared analyzing key fantasy football metrics. I intend to provide commentary in the notebooks to catelogue my thoughts and analysis.


```python
import argparse
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time
import concurrent.futures
import logging
```


```python
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def scrape_fantasypros(position, season):
    url = f"https://www.fantasypros.com/nfl/advanced-stats-{position}.php?year={season}"
    try:
        response = requests.get(url)
        response.raise_for_status()
        soup = BeautifulSoup(response.content, 'html.parser')
        table = soup.find('table', {'id': 'data'})

        headers = [th.text for th in table.find_all('th')]
        rows = []
        for tr in table.find_all('tr')[1:]:
            rows.append([td.text for td in tr.find_all('td')])

        df = pd.DataFrame(rows, columns=headers)
        df['Season'] = season
        df['Position'] = position.upper()
        df['Player'] = df['Player'].astype(str)
        df = df.iloc[1:,:]

        if position == 'qb':
          url = f"https://www.fantasypros.com/nfl/stats/qb.php?year={season}"
          response = requests.get(url)
          response.raise_for_status()
          soup = BeautifulSoup(response.content, 'html.parser')
          table = soup.find('table', {'id': 'data'})

          headers = [th.text for th in table.find_all('th')]
          rows = []
          for tr in table.find_all('tr')[1:]:
              rows.append([td.text for td in tr.find_all('td')])

          rushing_list_df = pd.DataFrame(rows, columns=headers)
          rushing_list_df['Season'] = season
          rushing_list_df['Position'] = position.upper()
          out_df = rushing_list_df.rename(columns={'ATT':'Rush_Att', 'YDS': 'Rush_Yds', 'TD':'Rush_TDs'})
          out_df = out_df.drop_duplicates()
          merged_df = df.merge(out_df, how='inner', on=['Player', 'Season'])
          cols = list(merged_df.loc[:, ~merged_df.columns.isin(['Player', 'Position'])].columns)
          merged_df[cols] = merged_df[cols].apply(pd.to_numeric, errors='coerce', axis=1)
          merged_df['ADOT'] = np.round(merged_df['AIR']/merged_df['TGT'])
          return merged_df
        else:
          cols = list(df.loc[:, ~df.columns.isin(['Player', 'Position'])].columns)
          df[cols] = df[cols].apply(pd.to_numeric, errors='coerce', axis=1)
          return df
    except requests.RequestException as e:
        logging.error(f"Error scraping {position.upper()} data for {season}: {str(e)}")
        return None
    except AttributeError as e:
        logging.error(f"Error parsing {position.upper()} data for {season}: {str(e)}")
        return None



def scrape_worker(args):
    position, season = args
    return scrape_fantasypros(position, season)
```


```python
positions = ['qb', 'rb', 'wr', 'te']
seasons = [2023]

scrape_args = [(position, season) for position in positions for season in seasons]

all_data = []

with concurrent.futures.ThreadPoolExecutor(max_workers=2) as executor:
    future_to_args = {executor.submit(scrape_worker, arg): arg for arg in scrape_args}
    for future in concurrent.futures.as_completed(future_to_args):
        args = future_to_args[future]
        try:
            df = future.result()
            if df is not None:
                all_data.append(df)
                logging.info(f"Scraped {args[0].upper()} data for {args[1]}")
        except Exception as e:
            logging.error(f"Error processing {args[0].upper()} data for {args[1]}: {str(e)}")
```


```python
if all_data:
    output_file = f"fantasypros_advanced_stats_{seasons[0]}-{seasons[1]}.xlsx"
    save_to_excel(all_data, output_file)
else:
    logging.warning("No data was successfully scraped.")
```
