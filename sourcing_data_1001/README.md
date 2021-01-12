# Introduction

This folder supports a tutorial article entitled: [_Sourcing Federal Data: Higher Education Data: 
A step-by-step guide to building three-year data panel from federal higher education data sources_](https://towardsdatascience.com/how-to-source-federal-data-higher-education-data-675f5edb9813)

The accompanying notebook provides the above tutorial's code in a ready-to-run format.

# Tutorial Summary

This guide puts together a three-year data panel (also known as longitudinal data) using public data available from the US Department of Education. Use the techniques in this article to prepare data that you can analyze on your own. Tell me in the comments what other federal data sources I should build similar guides for next.

# Jupyter Rendering Fail

Sometimes Jupyter does not render on GitHub. If that is the case for you, here is the code in Markdown (which will be missing the output). 

```Python
# STEP ONE
# ========

import requests
import zipfile
import io
import pandas as pd
# We use the ipeds prefix because the data comes from the:
# [I]ntegrated [P]ostsecondary [E]ducation [D]ata [S]ystem
ipeds_locs = 'https://nces.ed.gov/ipeds/datacenter/data/'
ipeds_fils = 'HD{}_Data_Stata.zip'
ipeds_dict = 'HD{}_Dict.zip'
years = [2015,2016,2017]

# STEPS TWO THROUGH FOUR
# ======================

# STEP TWO:
# =========
for yr in years:
    print('GETTING FILES FROM {}'.format(yr))
    rdata = requests.get(ipeds_locs + ipeds_fils.format(yr))
    rdict = requests.get(ipeds_locs + ipeds_dict.format(yr))
    rdata_zip = zipfile.ZipFile(io.BytesIO(rdata.content))
    rdict_zip = zipfile.ZipFile(io.BytesIO(rdict.content))
    
    print('Extracting {} files from zip archive:'.format(yr))
    rdata_zip.printdir()
    rdict_zip.printdir()
    rdata_zip.extractall()
    rdict_zip.extractall()
    
    print('Saving zip archive to disk.')
    open(ipeds_fils.format(yr), 'wb').write(rdata.content)
    open(ipeds_dict.format(yr), 'wb').write(rdict.content)
    
    
    # STEP THREE
    # ==========
    print('Replacing Code Values with Code Labels.')
    
    # Extract frequencies tab the data dictionary (hdYYYY.xlsx)
    freqs = pd.read_excel('hd{}.xlsx'.format(yr),
                          sheet_name='Frequencies')
    # Put institutional data into a data frame (df)
    df = pd.read_csv('hd{}_data_stata.csv'.format(yr), 
                     encoding='ISO-8859-1')    
    
    # Get list of categorical variable names
    cat_colms = set(freqs['varname'])
    
    # Remove fips code to prevent its modification
    cat_colms.remove('FIPS')
     
    # Loop through categorical columns
    for col in cat_colms:
        # Get map keys (code values)
        code_values = freqs[freqs['varname'] == col]['codevalue']
        # Convert map keys to int where appropriate
        code_values = [int(i) if str(i).isdigit() 
                       else i for i in code_values]
        # Get map value (ValueLabels)
        code_labels = freqs[freqs['varname'] == col]['valuelabel']
        var_map = dict(zip(code_values, code_labels)) 
        # Apply mapping dictionary to categorical column
        df[col] = df[col].map(var_map)
    
    
    # STEP FOUR
    # =========
    
    # Create time index for panel specification
    df['year'] = yr
    
    print('Writing hd{}_data_stata.csv as csv, pkl'.format(yr))
    df.columns = [i.lower() for i in df.columns]
    df.to_csv('hd{}_data_stata.csv'.format(yr))
    df.to_pickle('hd{}_data_stata.pkl'.format(yr))
    print('Done!', end='\n\n')
    

# STEP FIVE
# =========

all_data = {}
for yr in years:
    all_data[yr] = pd.read_csv('hd{}_data_stata.csv'.format(yr))
    
df = pd.concat(all_data).sort_values(['unitid',
                                      'year']).set_index(['unitid',
                                                          'year'])
                                                          
                                                          
# DISPLAY RESULTS - DISPLAY FIRST THREE INSTITUTIONS
# ==================================================

df[['instnm','city','stabbr','control','locale']].head(n=9)


# DISPLAY RESULTS - DISPLAY A SPECIFIC INSTITUTION (By Name)
# ==========================================================

df[df['instnm'] == 'University of Wisconsin-Madison']


# DISPLAY RESULTS - DISPLAY A SPECIFIC INSTITUTION (By Unitid)
# ============================================================

df.loc[174066]

```
