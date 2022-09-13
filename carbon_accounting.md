## Table of Contents
- [Libraries](#Libraries)
- [Data Cleaning](#data-cleaning)
- [Figure 1](#Figure_1)
- [Table 1](#Table_1)
- [Figure 2](#Figure_2)
- [Table 2](#Table_2)

## Libraries
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from pandas.plotting import parallel_coordinates
from sklearn.preprocessing import MinMaxScaler, StandardScaler
import seaborn as sns
```


## Data Cleaning
```python
# Strip % 
carbon['downstream_percent_total_pcf'] = (carbon['downstream_percent_total_pcf'].str.strip('%'))
carbon['upstream_percent_total_pcf'] = (carbon['upstream_percent_total_pcf'].str.strip('%'))
carbon['operations_percent_total_pcf'] = (carbon['operations_percent_total_pcf'].str.strip('%'))

# Replace NA
carbon['upstream_percent_total_pcf'] = carbon['upstream_percent_total_pcf'].replace('N/a (product with insufficient stage-level data)', np.NaN)
carbon['operations_percent_total_pcf'] = carbon['operations_percent_total_pcf'].replace('N/a (product with insufficient stage-level data)', np.NaN)
carbon['downstream_percent_total_pcf'] = carbon['downstream_percent_total_pcf'].replace('N/a (product with insufficient stage-level data)', np.NaN)

# Remove " " from industry_group
carbon['industry_group'] = carbon['industry_group'].str.strip('"')

# Remove " " from company name
carbon['company'] = carbon['company'].str.strip('"')
```

## Figure_1
```python
carbon_all = carbon[['industry_group','carbon_footprint_pcf']]
carbon_all_groupby = carbon_all.groupby('industry_group')['carbon_footprint_pcf'].sum().reset_index()
carbon_all_sorted = carbon_all_groupby.sort_values(by='carbon_footprint_pcf',ascending=False)

number_companies = carbon[['industry_group', 'company']]
number_companies_groupby = number_companies.groupby('industry_group')['company'].count().reset_index()
number_companies_count = number_companies_groupby.sort_values(by='company',ascending=False)
carbon_all_merged = carbon_all_sorted.merge(number_companies_count, on='industry_group',how='left')
```

## Table_1
```python
polluting_company = carbon[['year','industry_group', 'company','carbon_footprint_pcf']]
polluting_electrical = polluting_company[polluting_company['industry_group'] == 'Electrical Equipment and Machinery']
polluting_electrical_sorted = polluting_electrical.sort_values(by='carbon_footprint_pcf', ascending=False)
```

## Figure_2
```python
carbon_ci_2013 = carbon_ci_groupby[carbon_ci_groupby['year'] == 2013]
carbon_ci_2014 = carbon_ci_groupby[carbon_ci_groupby['year'] == 2014].reset_index(drop=True)
carbon_ci_2015 = carbon_ci_groupby[carbon_ci_groupby['year'] == 2015].reset_index(drop=True)
carbon_ci_2016 = carbon_ci_groupby[carbon_ci_groupby['year'] == 2016].reset_index(drop=True)
carbon_ci_2017 = carbon_ci_groupby[carbon_ci_groupby['year'] == 2017].reset_index(drop=True)

def ci_scaler(data):
    scaled = MinMaxScaler().fit_transform(data[['carbon_intensity']])
    df = pd.DataFrame(scaled)
    concat = pd.concat([data, df], axis=1)
    dropcol = concat.drop('carbon_intensity', axis='columns')
    dropcol.columns = ['year', 'industry_group', 'carbon_intensity']
    dropyear = dropcol.drop(['year'], axis=1)
    return dropyear

carbon_ci_2013_concat = ci_scaler(carbon_ci_2013)
carbon_ci_2014_concat = ci_scaler(carbon_ci_2014)
carbon_ci_2015_concat = ci_scaler(carbon_ci_2015)
carbon_ci_2016_concat = ci_scaler(carbon_ci_2016)
carbon_ci_2017_concat = ci_scaler(carbon_ci_2017)

carbon_ci_2013_concat.columns = ['industry_group', '2013']
carbon_ci_2014_concat.columns = ['industry_group', '2014']
carbon_ci_2015_concat.columns = ['industry_group', '2015']
carbon_ci_2016_concat.columns = ['industry_group', '2016']
carbon_ci_2017_concat.columns = ['industry_group', '2017']

ci_output1 = pd.merge(carbon_ci_2013_concat, carbon_ci_2014_concat, on='industry_group',how='outer')
ci_output2 = pd.merge(ci_output1, carbon_ci_2015_concat, on='industry_group',how='outer')
ci_output3 = pd.merge(ci_output2, carbon_ci_2016_concat, on='industry_group',how='outer')
ci_output4 = pd.merge(ci_output3, carbon_ci_2017_concat, on='industry_group',how='outer')

pd.plotting.parallel_coordinates(ci_output4, 'industry_group')
plt.legend(bbox_to_anchor=(1.02, 1), loc='upper left', borderaxespad=0)
plt.show()
```

## Table_2
```python
carbon_breakdown_hardware = carbon[['company', 'carbon_footprint_pcf', 'upstream_percent_total_pcf','downstream_percent_total_pcf' ,'operations_percent_total_pcf']]
carbon_breakdown_hardware = carbon_breakdown_hardware[carbon_breakdown_hardware['company'] == 'Technology Hardware & Equipment']
carbon_breakdown_hardware_sorted = carbon_breakdown_hardware.sort_values(ascending=False, by='carbon_footprint_pcf')
```