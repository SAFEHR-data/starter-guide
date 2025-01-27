This document is an introductory walkthrough, demonstrating how to read into Python some OMOP data from a series of csv files, join on concept names and explore and visualise the data.

It uses synthetic data created in UCLH as a part of the  [Critical Care Health Informatics Collaborative (CCHIC) project](https://safehr-data.github.io/uclh-research-discovery/projects/uclh_cchic_s0/index.html).

# Installing & loading required packages
1. ### pip install pandas

2. ### pip install seaborn

3. ### pip install matplotlib



```python
import os
import requests
import pandas as pd
import datetime
import seaborn as sns
import matplotlib.pyplot as plt
```

# Downloading & reading in OMOP data
Here we will download & read in some UCLH critical care data stored in Github


```python


# Repository and file paths
repository = "SAFEHR-data/uclh-research-discovery"
source_path = "_projects/uclh_cchic_s0/data"
destination_folder = "Starter_Guide_python/data"

# Create destination folder if it doesn't exist
os.makedirs(destination_folder, exist_ok=True)

# GitHub API URL to get the contents of the folder
api_url = f"https://api.github.com/repos/{repository}/contents/{source_path}"

# Fetch the list of files from the GitHub folder
files = requests.get(api_url).json()

# Download each file
for file in files:
    file_url = file["download_url"]
    file_name = file["name"]
    file_path = os.path.join(destination_folder, file_name)

    # Download the file and save it locally
    print(f"Downloading {file_name} ******")
    file_response = requests.get(file_url)

    # Save the file to the local folder
    with open(file_path, "wb") as local_file:
        local_file.write(file_response.content)

# List all downloaded files
print("Download complete! Files in the folder:")
print(os.listdir(destination_folder))

```

    Downloading condition_occurrence.csv ******
    Downloading death.csv ******
    Downloading device_exposure.csv ******
    Downloading drug_exposure.csv ******
    Downloading measurement.csv ******
    Downloading observation.csv ******
    Downloading observation_period.csv ******
    Downloading person.csv ******
    Downloading procedure_occurrence.csv ******
    Downloading specimen.csv ******
    Downloading visit_occurrence.csv ******
    Download complete! Files in the folder:
    ['observation_period.csv', 'drug_exposure.csv', 'specimen.csv', 'death.csv', 'device_exposure.csv', 'measurement.csv', 'condition_occurrence.csv', 'visit_occurrence.csv', 'person.csv', 'observation.csv', 'procedure_occurrence.csv']


# Loading Multiple CSV Files into DataFrames
This script loads all CSV files from a specified folder into individual pandas DataFrames and stores them in a dictionary.


```python
#########Load files in Data frames w.r.t file names
# store the loaded data from each CSV file.
df = {} 
for file in os.listdir(destination_folder):
    if file.endswith(".csv"):
        # Extract the file name without extension
        file_name = os.path.splitext(file)[0]
        # Load the file into a DataFrame
        file_path = os.path.join(destination_folder, file)
        df[file_name] = pd.read_csv(file_path)
        print(f"Loaded {file_name} into a DataFrame.")
```

    Loaded observation_period into a DataFrame.
    Loaded drug_exposure into a DataFrame.
    Loaded specimen into a DataFrame.
    Loaded death into a DataFrame.
    Loaded device_exposure into a DataFrame.
    Loaded measurement into a DataFrame.
    Loaded condition_occurrence into a DataFrame.
    Loaded visit_occurrence into a DataFrame.
    Loaded person into a DataFrame.
    Loaded observation into a DataFrame.
    Loaded procedure_occurrence into a DataFrame.



```python
for name in df.keys():
    print(name)

```

    observation_period
    drug_exposure
    specimen
    death
    device_exposure
    measurement
    condition_occurrence
    visit_occurrence
    person
    observation
    procedure_occurrence


# Looking at the person table
The OMOP tables are stored as data frames within the list object & can be accessed by the table name.

Thus we can find the names of the columns in person, use df["person"].columns and df["person"].head() to preview the data and seaborn() plot some of them. Note that not all columns contain data and in that case are filled with NA.


```python
# show column names for one of the tables
df['person'].columns
```




    Index(['person_id', 'gender_concept_id', 'year_of_birth', 'month_of_birth',
           'day_of_birth', 'birth_datetime', 'race_concept_id',
           'ethnicity_concept_id', 'location_id', 'provider_id', 'care_site_id',
           'person_source_value', 'gender_source_value',
           'gender_source_concept_id', 'race_source_value',
           'race_source_concept_id', 'ethnicity_source_value',
           'ethnicity_source_concept_id'],
          dtype='object')




```python
# show column datatype for one of the tables
df['person'].dtypes
```




    person_id                        int64
    gender_concept_id                int64
    year_of_birth                    int64
    month_of_birth                   int64
    day_of_birth                     int64
    birth_datetime                  object
    race_concept_id                  int64
    ethnicity_concept_id             int64
    location_id                      int64
    provider_id                    float64
    care_site_id                   float64
    person_source_value            float64
    gender_source_value             object
    gender_source_concept_id       float64
    race_source_value              float64
    race_source_concept_id         float64
    ethnicity_source_value          object
    ethnicity_source_concept_id    float64
    dtype: object




```python
# glimpse table data
df['person'].head() 
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
      <th>person_id</th>
      <th>gender_concept_id</th>
      <th>year_of_birth</th>
      <th>month_of_birth</th>
      <th>day_of_birth</th>
      <th>birth_datetime</th>
      <th>race_concept_id</th>
      <th>ethnicity_concept_id</th>
      <th>location_id</th>
      <th>provider_id</th>
      <th>care_site_id</th>
      <th>person_source_value</th>
      <th>gender_source_value</th>
      <th>gender_source_concept_id</th>
      <th>race_source_value</th>
      <th>race_source_concept_id</th>
      <th>ethnicity_source_value</th>
      <th>ethnicity_source_concept_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2451</td>
      <td>8532</td>
      <td>1947</td>
      <td>3</td>
      <td>18</td>
      <td>1947-03-18 11:34:00.221602</td>
      <td>46285839</td>
      <td>38003564</td>
      <td>97</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>FEMALE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Not Hispanic or Latino</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2452</td>
      <td>8532</td>
      <td>1945</td>
      <td>1</td>
      <td>2</td>
      <td>1945-01-02 19:12:41.795667</td>
      <td>46285825</td>
      <td>38003564</td>
      <td>92</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>FEMALE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Not Hispanic or Latino</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2453</td>
      <td>8507</td>
      <td>1985</td>
      <td>2</td>
      <td>13</td>
      <td>1985-02-13 09:53:37.14794</td>
      <td>46286810</td>
      <td>38003563</td>
      <td>70</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>MALE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Hispanic or Latino</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2454</td>
      <td>8507</td>
      <td>1948</td>
      <td>11</td>
      <td>22</td>
      <td>1948-11-22 02:21:08.685898</td>
      <td>46286810</td>
      <td>38003563</td>
      <td>32</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>MALE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Hispanic or Latino</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2455</td>
      <td>8507</td>
      <td>1946</td>
      <td>1</td>
      <td>19</td>
      <td>1946-01-19 04:41:57.006488</td>
      <td>46286810</td>
      <td>38003563</td>
      <td>91</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>MALE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Hispanic or Latino</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### plot some columns, patient birth years by gender


```python
birth_year_gender_cnt = df['person'].groupby(['year_of_birth', 'gender_concept_id'])['person_id'].count().reset_index()

# Rename the 'person_id' column to 'count'
birth_year_gender_cnt.rename(columns={'person_id': 'count'}, inplace=True)

# Set the figure size (optional)
plt.figure(figsize=(15, 15))

# Create the plot
sns.barplot(x='year_of_birth', y='count', hue='gender_concept_id', data=birth_year_gender_cnt)

# Add title and labels
plt.title('Count of Person IDs by Year of Birth and Gender_Concept_Id')
plt.xlabel('Year of Birth')
plt.ylabel('Count of Person IDs')
plt.xticks(rotation=45)

# Show the plot
plt.show()
```


    
![png](output_13_0.png)
    


In the plot above bars are coloured by gender_concept_id which is the OMOP ID for gender, but we don't actually know which is which. We will look at resolving that by retrieving OMOP concept names in the next section.

# OMOP Processed Vocabularies
Repository for pre-processed vocabularies for working with the OMOP CDM at UCLH.

You can download these files directly or follow the local development instructions to use git to clone the files

## Downloading versions
You can download a specific tagged version using https. in this format, replacing the curly braced values:

https://github.com/SAFEHR-data/omop-vocabs-processed/raw/refs/tags/{tag}/{relative_path}

## Getting names for OMOP concept IDs


```python
import urllib.request

tag = "v4"
relative_path = "data/concept.parquet"
download_url = f"https://github.com/SAFEHR-data/omop-vocabs-processed/raw/refs/tags/{tag}/{relative_path}"
local_filename = "concept.parquet"

urllib.request.urlretrieve(download_url, local_filename)
```




    ('concept.parquet', <http.client.HTTPMessage at 0x14883f3e0>)




```python

file_path = "/Users/muhammadqummerularfeen/concept.parquet" 

# Read the parquet file into a pandas DataFrame
df_concept = pd.read_parquet(file_path)

# Display the first few rows of the DataFrame to ensure it's loaded
df_concept.head()
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
      <th>concept_id</th>
      <th>concept_name</th>
      <th>domain_id</th>
      <th>vocabulary_id</th>
      <th>concept_class_id</th>
      <th>standard_concept</th>
      <th>concept_code</th>
      <th>valid_start_date</th>
      <th>valid_end_date</th>
      <th>invalid_reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>45756805</td>
      <td>Pediatric Cardiology</td>
      <td>Provider</td>
      <td>ABMS</td>
      <td>Physician Specialty</td>
      <td>S</td>
      <td>OMOP4821938</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>45756804</td>
      <td>Pediatric Anesthesiology</td>
      <td>Provider</td>
      <td>ABMS</td>
      <td>Physician Specialty</td>
      <td>S</td>
      <td>OMOP4821939</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>45756803</td>
      <td>Pathology-Anatomic / Pathology-Clinical</td>
      <td>Provider</td>
      <td>ABMS</td>
      <td>Physician Specialty</td>
      <td>S</td>
      <td>OMOP4821940</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>45756802</td>
      <td>Pathology - Pediatric</td>
      <td>Provider</td>
      <td>ABMS</td>
      <td>Physician Specialty</td>
      <td>S</td>
      <td>OMOP4821941</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>45756801</td>
      <td>Pathology - Molecular Genetic</td>
      <td>Provider</td>
      <td>ABMS</td>
      <td>Physician Specialty</td>
      <td>S</td>
      <td>OMOP4821942</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_person_concept = pd.merge(df['person'], df_concept, 
                             left_on='gender_concept_id', 
                             right_on='concept_id', 
                             how='left')

df_person_concept.head()
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
      <th>person_id</th>
      <th>gender_concept_id</th>
      <th>year_of_birth</th>
      <th>month_of_birth</th>
      <th>day_of_birth</th>
      <th>birth_datetime</th>
      <th>race_concept_id</th>
      <th>ethnicity_concept_id</th>
      <th>location_id</th>
      <th>provider_id</th>
      <th>...</th>
      <th>concept_id</th>
      <th>concept_name</th>
      <th>domain_id</th>
      <th>vocabulary_id</th>
      <th>concept_class_id</th>
      <th>standard_concept</th>
      <th>concept_code</th>
      <th>valid_start_date</th>
      <th>valid_end_date</th>
      <th>invalid_reason</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2451</td>
      <td>8532</td>
      <td>1947</td>
      <td>3</td>
      <td>18</td>
      <td>1947-03-18 11:34:00.221602</td>
      <td>46285839</td>
      <td>38003564</td>
      <td>97</td>
      <td>NaN</td>
      <td>...</td>
      <td>8532</td>
      <td>FEMALE</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>S</td>
      <td>F</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2452</td>
      <td>8532</td>
      <td>1945</td>
      <td>1</td>
      <td>2</td>
      <td>1945-01-02 19:12:41.795667</td>
      <td>46285825</td>
      <td>38003564</td>
      <td>92</td>
      <td>NaN</td>
      <td>...</td>
      <td>8532</td>
      <td>FEMALE</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>S</td>
      <td>F</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2453</td>
      <td>8507</td>
      <td>1985</td>
      <td>2</td>
      <td>13</td>
      <td>1985-02-13 09:53:37.14794</td>
      <td>46286810</td>
      <td>38003563</td>
      <td>70</td>
      <td>NaN</td>
      <td>...</td>
      <td>8507</td>
      <td>MALE</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>S</td>
      <td>M</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2454</td>
      <td>8507</td>
      <td>1948</td>
      <td>11</td>
      <td>22</td>
      <td>1948-11-22 02:21:08.685898</td>
      <td>46286810</td>
      <td>38003563</td>
      <td>32</td>
      <td>NaN</td>
      <td>...</td>
      <td>8507</td>
      <td>MALE</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>S</td>
      <td>M</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2455</td>
      <td>8507</td>
      <td>1946</td>
      <td>1</td>
      <td>19</td>
      <td>1946-01-19 04:41:57.006488</td>
      <td>46286810</td>
      <td>38003563</td>
      <td>91</td>
      <td>NaN</td>
      <td>...</td>
      <td>8507</td>
      <td>MALE</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>Gender</td>
      <td>S</td>
      <td>M</td>
      <td>1970-01-01</td>
      <td>2099-12-31</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 28 columns</p>
</div>




```python
birth_year_gender_cnt = df_person_concept.groupby(['year_of_birth', 'concept_name'])['person_id'].count().reset_index()

# Rename the 'person_id' column to 'count'
birth_year_gender_cnt.rename(columns={'person_id': 'count'}, inplace=True)

# Set the figure size (optional)
plt.figure(figsize=(15, 15))

# Create the plot
sns.barplot(x='year_of_birth', y='count', hue='concept_name', data=birth_year_gender_cnt)

# Add title and labels
plt.title('Count of Person IDs by Year of Birth and Concept_Name (Gender) ')
plt.xlabel('Year of Birth')
plt.ylabel('Count of Person IDs')
plt.xticks(rotation=45)

# Show the plot
plt.show()
```


    
![png](output_20_0.png)
    



```python

```

# Looking at the measurement table
We can use the measurement_concept_name column (that was added by omop_join_name_all() above) to see which are the most common measurements.


```python
df['measurement'].head()
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
      <th>measurement_id</th>
      <th>person_id</th>
      <th>measurement_concept_id</th>
      <th>measurement_date</th>
      <th>measurement_datetime</th>
      <th>measurement_time</th>
      <th>measurement_type_concept_id</th>
      <th>operator_concept_id</th>
      <th>value_as_number</th>
      <th>value_as_concept_id</th>
      <th>unit_concept_id</th>
      <th>range_low</th>
      <th>range_high</th>
      <th>provider_id</th>
      <th>visit_occurrence_id</th>
      <th>visit_detail_id</th>
      <th>measurement_source_value</th>
      <th>measurement_source_concept_id</th>
      <th>unit_source_value</th>
      <th>value_source_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2448</td>
      <td>2451</td>
      <td>45757366</td>
      <td>1996-06-26</td>
      <td>1996-06-26 12:26:10.383789</td>
      <td>NaN</td>
      <td>32817</td>
      <td>4172704</td>
      <td>91</td>
      <td>NaN</td>
      <td>8555</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2451</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2449</td>
      <td>2452</td>
      <td>45773395</td>
      <td>1985-11-07</td>
      <td>1985-11-07 05:38:15.089986</td>
      <td>NaN</td>
      <td>32817</td>
      <td>4171756</td>
      <td>83</td>
      <td>NaN</td>
      <td>8496</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2452</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2450</td>
      <td>2453</td>
      <td>45763689</td>
      <td>1995-02-06</td>
      <td>1995-02-06 11:31:10.395688</td>
      <td>NaN</td>
      <td>32817</td>
      <td>4171756</td>
      <td>89</td>
      <td>NaN</td>
      <td>8496</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2453</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2451</td>
      <td>2454</td>
      <td>45773403</td>
      <td>1974-07-18</td>
      <td>1974-07-18 05:53:27.066081</td>
      <td>NaN</td>
      <td>32817</td>
      <td>4171755</td>
      <td>60</td>
      <td>NaN</td>
      <td>8648</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2454</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2452</td>
      <td>2455</td>
      <td>45763927</td>
      <td>2011-06-27</td>
      <td>2011-06-27 04:57:48.576704</td>
      <td>NaN</td>
      <td>32817</td>
      <td>4171755</td>
      <td>115</td>
      <td>NaN</td>
      <td>8547</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2455</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Join the Measurement dataframe with Concept
df_measurement_concept = pd.merge(df['measurement'], df_concept, 
                             left_on='measurement_concept_id', 
                             right_on='concept_id', 
                             how='left')

# most frequent measurement concepts
measurement_concept_cnt=df_measurement_concept.groupby(['concept_name'])['person_id'].count().reset_index().sort_values(by='person_id', ascending=False)

# rename the columns
measurement_concept_cnt.rename(columns={'person_id': 'Count', 'concept_name': 'Measurement Concept Name'}, inplace=True)

measurement_concept_cnt.head(5).reset_index()
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
      <th>index</th>
      <th>Measurement Concept Name</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>19</td>
      <td>Direct site</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>12 month target weight</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>46</td>
      <td>Prescription observable</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20</td>
      <td>Disabilities of the arm shoulder and hand outc...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32</td>
      <td>Kapandji clinical opposition and reposition te...</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



# Looking at the observation table
We can use the observation_concept_name column to see which are the most common observations.


```python
df['observation'].head()
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
      <th>observation_id</th>
      <th>person_id</th>
      <th>observation_concept_id</th>
      <th>observation_date</th>
      <th>observation_datetime</th>
      <th>observation_type_concept_id</th>
      <th>value_as_number</th>
      <th>value_as_string</th>
      <th>value_as_concept_id</th>
      <th>qualifier_concept_id</th>
      <th>unit_concept_id</th>
      <th>provider_id</th>
      <th>visit_occurrence_id</th>
      <th>visit_detail_id</th>
      <th>observation_source_value</th>
      <th>observation_source_concept_id</th>
      <th>unit_source_value</th>
      <th>qualifier_source_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2443</td>
      <td>2451</td>
      <td>723462</td>
      <td>2012-05-01</td>
      <td>2012-05-01 04:27:08.24232</td>
      <td>32816</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>725555</td>
      <td>NaN</td>
      <td>44777550</td>
      <td>NaN</td>
      <td>2451</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>710685</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2444</td>
      <td>2452</td>
      <td>706011</td>
      <td>1975-02-21</td>
      <td>1975-02-21 18:56:45.359304</td>
      <td>32868</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>715268</td>
      <td>NaN</td>
      <td>44777585</td>
      <td>NaN</td>
      <td>2452</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>715745</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2445</td>
      <td>2453</td>
      <td>723489</td>
      <td>1997-12-09</td>
      <td>1997-12-09 17:25:41.461328</td>
      <td>32809</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>710689</td>
      <td>NaN</td>
      <td>44777534</td>
      <td>NaN</td>
      <td>2453</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>713860</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2446</td>
      <td>2454</td>
      <td>704996</td>
      <td>2009-03-31</td>
      <td>2009-03-31 17:14:09.077546</td>
      <td>32840</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>715740</td>
      <td>NaN</td>
      <td>44777541</td>
      <td>NaN</td>
      <td>2454</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>706008</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2447</td>
      <td>2455</td>
      <td>725553</td>
      <td>2000-02-15</td>
      <td>2000-02-15 15:37:07.272383</td>
      <td>32849</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>706007</td>
      <td>NaN</td>
      <td>44777533</td>
      <td>NaN</td>
      <td>2455</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>723126</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Join the Measurement dataframe with Concept
df_observation_concept=pd.merge(df['observation'], df_concept, 
                             left_on='observation_concept_id', 
                             right_on='concept_id', 
                             how='left')

# most frequent measurement concepts
observation_concept_cnt=df_observation_concept.groupby(['concept_name'])['person_id'].count().reset_index().sort_values(by='person_id', ascending=False)

# rename the columns
observation_concept_cnt.rename(columns={'person_id': 'Count', 'concept_name': 'Observation Concept Name'}, inplace=True)

observation_concept_cnt.head().reset_index()

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
      <th>index</th>
      <th>Observation Concept Name</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4</td>
      <td>Assault by unspecified gases and vapours</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12</td>
      <td>City of travel [Location]</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>Advice given about 2019-nCoV (novel coronaviru...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9</td>
      <td>COVID-19 Intubation Progress note</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>32</td>
      <td>Metastasis</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



# Looking at the drug_exposure table
We can use the drug_concept_name column to see which are the most common drugs.


```python
df['drug_exposure'].head()
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
      <th>drug_exposure_id</th>
      <th>person_id</th>
      <th>drug_concept_id</th>
      <th>drug_exposure_start_date</th>
      <th>drug_exposure_start_datetime</th>
      <th>drug_exposure_end_date</th>
      <th>drug_exposure_end_datetime</th>
      <th>verbatim_end_date</th>
      <th>drug_type_concept_id</th>
      <th>stop_reason</th>
      <th>...</th>
      <th>sig</th>
      <th>route_concept_id</th>
      <th>lot_number</th>
      <th>provider_id</th>
      <th>visit_occurrence_id</th>
      <th>visit_detail_id</th>
      <th>drug_source_value</th>
      <th>drug_source_concept_id</th>
      <th>route_source_value</th>
      <th>dose_unit_source_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2443</td>
      <td>2451</td>
      <td>44818407</td>
      <td>2004-06-11</td>
      <td>2004-06-11 17:28:54.399408</td>
      <td>2009-07-12</td>
      <td>2009-07-12 02:31:47.97656</td>
      <td>NaN</td>
      <td>38000180</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>4156705</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2451</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21183732</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2444</td>
      <td>2452</td>
      <td>44818490</td>
      <td>1981-12-14</td>
      <td>1981-12-14 08:28:09.102107</td>
      <td>1990-01-08</td>
      <td>1990-01-08 15:33:59.305522</td>
      <td>NaN</td>
      <td>43542358</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>4167540</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2452</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21179371</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2445</td>
      <td>2453</td>
      <td>44818425</td>
      <td>1994-10-31</td>
      <td>1994-10-31 17:15:46.193519</td>
      <td>1996-06-07</td>
      <td>1996-06-07 13:17:53.172747</td>
      <td>NaN</td>
      <td>32426</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>40492305</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2453</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21217483</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2446</td>
      <td>2454</td>
      <td>44818479</td>
      <td>1961-03-11</td>
      <td>1961-03-11 03:31:53.289461</td>
      <td>1987-12-02</td>
      <td>1987-12-02 00:24:09.811676</td>
      <td>NaN</td>
      <td>38000177</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>40490898</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2454</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21182589</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2447</td>
      <td>2455</td>
      <td>44818391</td>
      <td>2003-04-13</td>
      <td>2003-04-13 23:45:25.852554</td>
      <td>2007-04-10</td>
      <td>2007-04-10 10:36:16.431806</td>
      <td>NaN</td>
      <td>38000175</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>600817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2455</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>21217484</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 23 columns</p>
</div>




```python
# Join the Drug Exposure dataframe with Concept
df_drug_exposure_concept = pd.merge(df['drug_exposure'], df_concept, 
                             left_on='drug_concept_id', 
                             right_on='concept_id', 
                             how='left')

# most frequent  Drug Exposure concepts
drugexposure_concept_cnt=df_drug_exposure_concept.groupby(['concept_name'])['person_id'].count().reset_index().sort_values(by='person_id', ascending=False)

# rename the columns
drugexposure_concept_cnt.rename(columns={'person_id': 'Count', 'concept_name': 'Measurement Concept Name'}, inplace=True)

drugexposure_concept_cnt.head(5).reset_index()
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
      <th>index</th>
      <th>Measurement Concept Name</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>17</td>
      <td>Lotus corniculatus flower volatile oil</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>51</td>
      <td>thyroid (USP) 81.25 MG Oral Tablet [Nature-Thr...</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>32</td>
      <td>hydrocortisone 0.5 MG/ML / lidocaine hydrochlo...</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>58</td>
      <td>zinc sulfate 125 MG Effervescent Oral Tablet</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>55</td>
      <td>toltrazuril</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



# Looking at the visit_occurrence table

The visit_occurrence table contains times and attributes of visits. Other tables (e.g. measurement & observation) have a visit_occurrence_id column that can be used to establish the visit that they were associated with. Visits have a start & end date, in these synthetic data the interval between them can be substantial.


```python
df['visit_occurrence'].head()
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
      <th>visit_occurrence_id</th>
      <th>person_id</th>
      <th>visit_concept_id</th>
      <th>visit_start_date</th>
      <th>visit_start_datetime</th>
      <th>visit_end_date</th>
      <th>visit_end_datetime</th>
      <th>visit_type_concept_id</th>
      <th>provider_id</th>
      <th>care_site_id</th>
      <th>visit_source_value</th>
      <th>visit_source_concept_id</th>
      <th>admitting_source_concept_id</th>
      <th>admitting_source_value</th>
      <th>discharge_to_concept_id</th>
      <th>discharge_to_source_value</th>
      <th>preceding_visit_occurrence_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2451</td>
      <td>2451</td>
      <td>9203</td>
      <td>1992-07-23</td>
      <td>1992-07-23 21:20:23.786384</td>
      <td>2015-09-09</td>
      <td>2015-09-09 19:12:40.927984</td>
      <td>32817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9203</td>
      <td>8882</td>
      <td>Adult Living Care Facility</td>
      <td>8716</td>
      <td>Independent Clinic</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2452</td>
      <td>2452</td>
      <td>9203</td>
      <td>1949-01-14</td>
      <td>1949-01-14 22:11:41.996225</td>
      <td>1991-01-29</td>
      <td>1991-01-29 22:07:58.329809</td>
      <td>32817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9201</td>
      <td>8536</td>
      <td>Home</td>
      <td>8615</td>
      <td>Assisted Living Facility</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2453</td>
      <td>2453</td>
      <td>9203</td>
      <td>1994-02-10</td>
      <td>1994-02-10 23:38:17.407702</td>
      <td>1998-04-24</td>
      <td>1998-04-24 13:05:43.724848</td>
      <td>32817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9203</td>
      <td>8761</td>
      <td>Rural Health Clinic</td>
      <td>8977</td>
      <td>Public Health Clinic</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2454</td>
      <td>2454</td>
      <td>9203</td>
      <td>1959-07-06</td>
      <td>1959-07-06 05:58:55.830104</td>
      <td>2013-01-24</td>
      <td>2013-01-24 07:58:42.746816</td>
      <td>32817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9201</td>
      <td>8970</td>
      <td>Inpatient Long-term Care</td>
      <td>8870</td>
      <td>Emergency Room - Hospital</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2455</td>
      <td>2455</td>
      <td>9201</td>
      <td>1973-11-01</td>
      <td>1973-11-01 03:55:41.889028</td>
      <td>2013-11-22</td>
      <td>2013-11-22 17:51:41.922618</td>
      <td>32817</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9203</td>
      <td>8546</td>
      <td>Hospice</td>
      <td>8536</td>
      <td>Home</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['visit_occurrence']['visit_start_date'] = pd.to_datetime(df['visit_occurrence']['visit_start_date'], errors='coerce')
df['visit_occurrence']['visit_end_date'] = pd.to_datetime(df['visit_occurrence']['visit_end_date'], errors='coerce')

```


```python
df['visit_occurrence'][['visit_occurrence_id','person_id','visit_concept_id','visit_type_concept_id']].head()
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
      <th>visit_occurrence_id</th>
      <th>person_id</th>
      <th>visit_concept_id</th>
      <th>visit_type_concept_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2451</td>
      <td>2451</td>
      <td>9203</td>
      <td>32817</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2452</td>
      <td>2452</td>
      <td>9203</td>
      <td>32817</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2453</td>
      <td>2453</td>
      <td>9203</td>
      <td>32817</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2454</td>
      <td>2454</td>
      <td>9203</td>
      <td>32817</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2455</td>
      <td>2455</td>
      <td>9201</td>
      <td>32817</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['visit_occurrence']['visit_start_year'] =df['visit_occurrence']['visit_start_date'].dt.to_period('Y')
df['visit_occurrence']['visit_end_year'] =df['visit_occurrence']['visit_end_date'].dt.to_period('Y')
```


```python
# Join the Visit Occurance dataframe with Concept
df_visit_concept = pd.merge(df['visit_occurrence'], df_concept, 
                             left_on='visit_concept_id', 
                             right_on='concept_id', 
                             how='left')

# most frequent Visit Occurance concepts
visit_occurance_concept_cnt = df_visit_concept.groupby(['concept_name', 'visit_start_year'])['person_id'].count().reset_index().sort_values(by='visit_start_year', ascending=False)


# rename the columns
visit_occurance_concept_cnt.rename(columns={'person_id': 'Count', 'concept_name': 'Visit Concept Name','visit_start_year': 'Year'}, inplace=True)

visit_occurance_concept_cnt.head()
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
      <th>Visit Concept Name</th>
      <th>Year</th>
      <th>Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>72</th>
      <td>Inpatient Visit</td>
      <td>2023</td>
      <td>3</td>
    </tr>
    <tr>
      <th>71</th>
      <td>Inpatient Visit</td>
      <td>2021</td>
      <td>1</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Emergency Room Visit</td>
      <td>2021</td>
      <td>1</td>
    </tr>
    <tr>
      <th>70</th>
      <td>Inpatient Visit</td>
      <td>2019</td>
      <td>1</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Emergency Room Visit</td>
      <td>2019</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Plot using seaborn
plt.figure(figsize=(20, 6))

sns.barplot(x='Year', y='Count', hue='Visit Concept Name', data=visit_occurance_concept_cnt)

# Add titles and labels
plt.title('Visit Count by Year and Visit Concept Name')
plt.xlabel('Year')
plt.ylabel('Number of Visits')
plt.xticks(rotation=45)

# Show the plot
plt.show()
```


    
![png](output_38_0.png)
    


# Joining person data to other tables

The OMOP common data model is person centred. Most tables have a person_id column that can be used to relate these data to other attributes of the patient. Here we show how we can join the measurement and person tables to see if there is any gender difference in measurements. A similar approach could be used to join to other tables including observation & drug_exposure.


```python
joined_mp = pd.merge(df['person'], df['measurement'], 
                             on='person_id', 
                             how='inner')
```


```python
freq_top_measures = (
    joined_mp
    .groupby(['measurement_concept_id', 'gender_concept_id'])
    .size()
    .reset_index(name='n')
    .sort_values(by='n', ascending=False)
    .query('n > 1')
)


```


```python
joined_data = (
    joined_mp
    .merge(df_concept, left_on='measurement_concept_id', right_on='concept_id', how='left')
    .rename(columns={'concept_name_x': 'measurement_concept_name'})
    .merge(df_concept, left_on='gender_concept_id', right_on='concept_id', how='left')
    .rename(columns={'concept_name_y': 'gender_concept_name'})
)
```


```python
freq_top_measures = (
    joined_data
    .groupby(['concept_name_x', 'gender_concept_name'])
    .size()
    .reset_index(name='n')
    .sort_values(by='n', ascending=False)
    .query('n > 1')
)
freq_top_measures.rename(columns={'concept_name_x': 'measurement_concept_name'}, inplace=True)

```


```python
freq_top_measures.head()
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
      <th>measurement_concept_name</th>
      <th>gender_concept_name</th>
      <th>n</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>28</th>
      <td>Disabilities of the arm shoulder and hand outc...</td>
      <td>MALE</td>
      <td>3</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Direct site</td>
      <td>MALE</td>
      <td>3</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Urine orotic acid:creatinine ratio measurement</td>
      <td>FEMALE</td>
      <td>2</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Detection of lymphocytes positive for CD8 antigen</td>
      <td>MALE</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>12 month target weight</td>
      <td>MALE</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Set up the FacetGrid
g = sns.FacetGrid(
    freq_top_measures,
    col="gender_concept_name",
    sharey=True,
    height=6,
    aspect=1
)

g.map_dataframe(
    sns.barplot,
    x="n",
    y="measurement_concept_name",
    hue="measurement_concept_name",
    palette="muted"
)

# Customize appearance
g.set_titles(col_template="{col_name}")  # Add titles for each gender
g.set_axis_labels("Count (n)", "Measurement Concept Name")  # Add axis labels

# Adjust layout
plt.subplots_adjust(top=0.9)
g.fig.suptitle("Top Measurements by Gender", fontsize=16)  # Set Graph Title

# Show plot
plt.show()

```


    
![png](output_46_0.png)
    


Note that we use left_join here because we only want to join on person information for rows occurring in the measurement table which is the left hand argument of the join. Also note that in this example we end up with one row per patient because the synthetic measurement table only has one row per patient. Usually we would expect multiple measurements per patient that would result in multiple rows per patient in the joined table.

# Differences between these synthetic data and real patient data
These particular synthetic data are useful to demonstrate the reading in and manipulation of OMOP data but there are some major differences between them and real patient data.

person, measurement, observation & drug_exposure tables are all same length (100 rows), in real data one would expect many more measurements, observations & drug exposures than patients
Related to 1, in these data there are a single measurement, observation and drug_exposure per patient. In reality one would expect many tens or hundreds of these other values per patient.
