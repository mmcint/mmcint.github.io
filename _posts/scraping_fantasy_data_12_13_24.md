```python
from google.colab import drive
drive.mount("/gdrive")
!jupyter nbconvert --to markdown "/gdrive/My Drive/Colab Notebooks/scraping_fantasy_data_12-13-24.ipynb"
```

    Mounted at /gdrive
    [NbConvertApp] WARNING | pattern '/gdrive/My Drive/Colab Notebooks/scraping_fantasy_data_12-13-24.ipynb' matched no files
    This application is used to convert notebook files (*.ipynb)
            to various other formats.
    
            WARNING: THE COMMANDLINE INTERFACE MAY CHANGE IN FUTURE RELEASES.
    
    Options
    =======
    The options below are convenience aliases to configurable class-options,
    as listed in the "Equivalent to" description-line of the aliases.
    To see all configurable class-options for some <cmd>, use:
        <cmd> --help-all
    
    --debug
        set log level to logging.DEBUG (maximize logging output)
        Equivalent to: [--Application.log_level=10]
    --show-config
        Show the application's configuration (human-readable format)
        Equivalent to: [--Application.show_config=True]
    --show-config-json
        Show the application's configuration (json format)
        Equivalent to: [--Application.show_config_json=True]
    --generate-config
        generate default config file
        Equivalent to: [--JupyterApp.generate_config=True]
    -y
        Answer yes to any questions instead of prompting.
        Equivalent to: [--JupyterApp.answer_yes=True]
    --execute
        Execute the notebook prior to export.
        Equivalent to: [--ExecutePreprocessor.enabled=True]
    --allow-errors
        Continue notebook execution even if one of the cells throws an error and include the error message in the cell output (the default behaviour is to abort conversion). This flag is only relevant if '--execute' was specified, too.
        Equivalent to: [--ExecutePreprocessor.allow_errors=True]
    --stdin
        read a single notebook file from stdin. Write the resulting notebook with default basename 'notebook.*'
        Equivalent to: [--NbConvertApp.from_stdin=True]
    --stdout
        Write notebook output to stdout instead of files.
        Equivalent to: [--NbConvertApp.writer_class=StdoutWriter]
    --inplace
        Run nbconvert in place, overwriting the existing notebook (only
                relevant when converting to notebook format)
        Equivalent to: [--NbConvertApp.use_output_suffix=False --NbConvertApp.export_format=notebook --FilesWriter.build_directory=]
    --clear-output
        Clear output of current file and save in place,
                overwriting the existing notebook.
        Equivalent to: [--NbConvertApp.use_output_suffix=False --NbConvertApp.export_format=notebook --FilesWriter.build_directory= --ClearOutputPreprocessor.enabled=True]
    --coalesce-streams
        Coalesce consecutive stdout and stderr outputs into one stream (within each cell).
        Equivalent to: [--NbConvertApp.use_output_suffix=False --NbConvertApp.export_format=notebook --FilesWriter.build_directory= --CoalesceStreamsPreprocessor.enabled=True]
    --no-prompt
        Exclude input and output prompts from converted document.
        Equivalent to: [--TemplateExporter.exclude_input_prompt=True --TemplateExporter.exclude_output_prompt=True]
    --no-input
        Exclude input cells and output prompts from converted document.
                This mode is ideal for generating code-free reports.
        Equivalent to: [--TemplateExporter.exclude_output_prompt=True --TemplateExporter.exclude_input=True --TemplateExporter.exclude_input_prompt=True]
    --allow-chromium-download
        Whether to allow downloading chromium if no suitable version is found on the system.
        Equivalent to: [--WebPDFExporter.allow_chromium_download=True]
    --disable-chromium-sandbox
        Disable chromium security sandbox when converting to PDF..
        Equivalent to: [--WebPDFExporter.disable_sandbox=True]
    --show-input
        Shows code input. This flag is only useful for dejavu users.
        Equivalent to: [--TemplateExporter.exclude_input=False]
    --embed-images
        Embed the images as base64 dataurls in the output. This flag is only useful for the HTML/WebPDF/Slides exports.
        Equivalent to: [--HTMLExporter.embed_images=True]
    --sanitize-html
        Whether the HTML in Markdown cells and cell outputs should be sanitized..
        Equivalent to: [--HTMLExporter.sanitize_html=True]
    --log-level=<Enum>
        Set the log level by value or name.
        Choices: any of [0, 10, 20, 30, 40, 50, 'DEBUG', 'INFO', 'WARN', 'ERROR', 'CRITICAL']
        Default: 30
        Equivalent to: [--Application.log_level]
    --config=<Unicode>
        Full path of a config file.
        Default: ''
        Equivalent to: [--JupyterApp.config_file]
    --to=<Unicode>
        The export format to be used, either one of the built-in formats
                ['asciidoc', 'custom', 'html', 'latex', 'markdown', 'notebook', 'pdf', 'python', 'qtpdf', 'qtpng', 'rst', 'script', 'slides', 'webpdf']
                or a dotted object name that represents the import path for an
                ``Exporter`` class
        Default: ''
        Equivalent to: [--NbConvertApp.export_format]
    --template=<Unicode>
        Name of the template to use
        Default: ''
        Equivalent to: [--TemplateExporter.template_name]
    --template-file=<Unicode>
        Name of the template file to use
        Default: None
        Equivalent to: [--TemplateExporter.template_file]
    --theme=<Unicode>
        Template specific theme(e.g. the name of a JupyterLab CSS theme distributed
        as prebuilt extension for the lab template)
        Default: 'light'
        Equivalent to: [--HTMLExporter.theme]
    --sanitize_html=<Bool>
        Whether the HTML in Markdown cells and cell outputs should be sanitized.This
        should be set to True by nbviewer or similar tools.
        Default: False
        Equivalent to: [--HTMLExporter.sanitize_html]
    --writer=<DottedObjectName>
        Writer class used to write the
                                            results of the conversion
        Default: 'FilesWriter'
        Equivalent to: [--NbConvertApp.writer_class]
    --post=<DottedOrNone>
        PostProcessor class used to write the
                                            results of the conversion
        Default: ''
        Equivalent to: [--NbConvertApp.postprocessor_class]
    --output=<Unicode>
        Overwrite base name use for output files.
                    Supports pattern replacements '{notebook_name}'.
        Default: '{notebook_name}'
        Equivalent to: [--NbConvertApp.output_base]
    --output-dir=<Unicode>
        Directory to write output(s) to. Defaults
                                      to output to the directory of each notebook. To recover
                                      previous default behaviour (outputting to the current
                                      working directory) use . as the flag value.
        Default: ''
        Equivalent to: [--FilesWriter.build_directory]
    --reveal-prefix=<Unicode>
        The URL prefix for reveal.js (version 3.x).
                This defaults to the reveal CDN, but can be any url pointing to a copy
                of reveal.js.
                For speaker notes to work, this must be a relative path to a local
                copy of reveal.js: e.g., "reveal.js".
                If a relative path is given, it must be a subdirectory of the
                current directory (from which the server is run).
                See the usage documentation
                (https://nbconvert.readthedocs.io/en/latest/usage.html#reveal-js-html-slideshow)
                for more details.
        Default: ''
        Equivalent to: [--SlidesExporter.reveal_url_prefix]
    --nbformat=<Enum>
        The nbformat version to write.
                Use this to downgrade notebooks.
        Choices: any of [1, 2, 3, 4]
        Default: 4
        Equivalent to: [--NotebookExporter.nbformat_version]
    
    Examples
    --------
    
        The simplest way to use nbconvert is
    
                > jupyter nbconvert mynotebook.ipynb --to html
    
                Options include ['asciidoc', 'custom', 'html', 'latex', 'markdown', 'notebook', 'pdf', 'python', 'qtpdf', 'qtpng', 'rst', 'script', 'slides', 'webpdf'].
    
                > jupyter nbconvert --to latex mynotebook.ipynb
    
                Both HTML and LaTeX support multiple output templates. LaTeX includes
                'base', 'article' and 'report'.  HTML includes 'basic', 'lab' and
                'classic'. You can specify the flavor of the format used.
    
                > jupyter nbconvert --to html --template lab mynotebook.ipynb
    
                You can also pipe the output to stdout, rather than a file
    
                > jupyter nbconvert mynotebook.ipynb --stdout
    
                PDF is generated via latex
    
                > jupyter nbconvert mynotebook.ipynb --to pdf
    
                You can get (and serve) a Reveal.js-powered slideshow
    
                > jupyter nbconvert myslides.ipynb --to slides --post serve
    
                Multiple notebooks can be given at the command line in a couple of
                different ways:
    
                > jupyter nbconvert notebook*.ipynb
                > jupyter nbconvert notebook1.ipynb notebook2.ipynb
    
                or you can specify the notebooks list in a config file, containing::
    
                    c.NbConvertApp.notebooks = ["my_notebook.ipynb"]
    
                > jupyter nbconvert --config mycfg.py
    
    To see all available configurables, use `--help-all`.
    



```python
from google.colab import drive
drive.mount('/content/drive')
```

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
