### Questions

## Question 1. Data Loading

Once the dataset is loaded, what's the shape of the data?

- [ ] 266,855 rows x 20 columns
- [x] 544,898 rows x 18 columns
- [ ] 544,898 rows x 20 columns
- [ ] 133,744 rows x 20 columns


## Question 2. Data Transformation

Upon filtering the dataset where the passenger count is greater than 0 _and_ the trip distance is greater than zero, how many rows are left?

- [x] 544,897 rows
- [ ] 266,855 rows
- [ ] 139,370 rows
- [ ] 266,856 rows

## Question 3. Data Transformation

Which of the following creates a new column `lpep_pickup_date` by converting `lpep_pickup_datetime` to a date?

- [ ] `data = data['lpep_pickup_datetime'].date`
- [ ] `data('lpep_pickup_date') = data['lpep_pickup_datetime'].date`
- [x] `data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt.date`
- [ ] `data['lpep_pickup_date'] = data['lpep_pickup_datetime'].dt().date()`

## Question 4. Data Transformation

What are the existing values of `VendorID` in the dataset?

- [ ] 1, 2, or 3
- [x] 1 or 2
- [ ] 1, 2, 3, 4
- [ ] 1

## Question 5. Data Transformation

How many columns need to be renamed to snake case?

- [x] 3
- [ ] 6
- [ ] 2
- [ ] 4

## Question 6. Data Exporting

Once exported, how many partitions (folders) are present in Google Cloud?

- [x] 96
- [ ] 56
- [ ] 67
- [ ] 108

## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2024/homework/hw2
* Check the link above to see the due date
  
## Solution
### DATA LOADER
```python
# import io
import pandas as pd
import requests
if 'data_loader' not in globals():
    from mage_ai.data_preparation.decorators import data_loader
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test

@data_loader
def load_data_from_api(*args, **kwargs):
    urls = [
        'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-10.csv.gz',
        'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-11.csv.gz',
        'https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2020-12.csv.gz'
    ]

    # Initialize an empty list to store dataframes
    dfs = []

    # Data types and date columns to parse
    taxi_dtypes = {
        'VendorID': pd.Int64Dtype(),
        'passenger_count': pd.Int64Dtype(),
        'trip_distance': float,
        'RatecodeID': pd.Int64Dtype(),
        'store_and_fwd_flag': str,
        'PULocationID': pd.Int64Dtype(),
        'DOLocationID': pd.Int64Dtype(),
        'payment_type': pd.Int64Dtype(),
        'fare_amount': float,
        'extra': float,
        'mta_tax': float,
        'tip_amount': float,
        'tolls_amount': float,
        'improvement_surcharge': float,
        'total_amount': float,
        'congestion_surcharge': float
    }
    parse_dates = ['lpep_pickup_datetime', 'lpep_dropoff_datetime']

    # Loop through the URLs, read the CSV files with specified dtypes and date parsing, and append to the list
    for url in urls:
        df = pd.read_csv(url, compression='gzip', dtype=taxi_dtypes, parse_dates=parse_dates)
        dfs.append(df)

    # Concatenate all dataframes in the list
    final_df = pd.concat(dfs, ignore_index=True)

    return final_df



@test
def test_output(output, *args) -> None:
    """
    Template code for testing the output of the block.
    """
    assert output is not None, 'The output is undefined'

```
### TRANSFORMER
```python
import io
import pandas as pd
import requests

if 'transformer' not in globals():
    from mage_ai.data_preparation.decorators import transformer
if 'test' not in globals():
    from mage_ai.data_preparation.decorators import test


@transformer
def transform(final_df, *args, **kwargs):
    """
    Template code for a transformer block.

    Add more parameters to this function if this block has multiple parent blocks.
    There should be one parameter for each output variable from each parent block.

    Args:
        data: The output from the upstream parent block
        args: The output from any additional upstream blocks (if applicable)

    Returns:
        Anything (e.g. data frame, dictionary, array, int, str, etc.)
    """
        # Remove rows where passenger count or trip distance is 0
    # Remove rows where passenger count or trip distance is 0
    final_df = final_df[(final_df['passenger_count'] > 0) & (final_df['trip_distance'] > 0)]
    
    # Create a new column 'lpep_pickup_date' by converting 'lpep_pickup_datetime' to date
    final_df['lpep_pickup_date'] = pd.to_datetime(final_df['lpep_pickup_datetime']).dt.date
    
    # Rename columns from Camel Case to Snake Case
    final_df.columns = [col.lower().replace('id', '_id') for col in final_df.columns]

    #how many rows are left?
    print(final_df.shape)

    #What are the existing values of VendorID in the dataset?
    print(final_df['vendor_id'].unique())

    # print(final_df.dtype())
    
    return final_df

```

```python

import pyarrow as pa
import pyarrow.parquet as pq
import os

if 'data_exporter' not in globals():
    from mage_ai.data_preparation.decorators import data_exporter


os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = "/home/src/mystic-shelter-339813-d6462b9c8c18.json"
bucket_name = "mage-zoomcamp_2024_tosin"
project_id = 'mystic-shelter-339813'

table_name = "nyc_taxi_data_green"
root_path = f'{bucket_name}/{table_name}'

@data_exporter
def export_data(final_df, *args, **kwargs):
    final_df['lpep_pickup_date'] = final_df['lpep_pickup_datetime'].dt.date
    table = pa.Table.from_pandas(final_df)
    gcs = pa.fs.GcsFileSystem()

    pq.write_to_dataset(
        table,
        root_path = root_path,
        partition_cols = ['lpep_pickup_date'],
        filesystem = gcs
    )
```
