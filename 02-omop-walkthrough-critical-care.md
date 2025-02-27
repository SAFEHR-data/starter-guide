<!-- 
  *.md is generated from `*.Rmd` in /dynamic-docs/, this is updated in a github action,
  you can regenerate this by running the "Find and knit Rmd files" step from the github action.
-->

This document is an introductory walkthrough, demonstrating how to read
into R some OMOP data from a series of csv files, join on concept names
and explore and visualise the data.

It uses synthetic data created in UCLH as a part of the [Critical Care
Health Informatics Collaborative (CCHIC)
project](https://safehr-data.github.io/uclh-research-discovery/projects/uclh_cchic_s0/index.html).

## Installing & loading required packages

If any of the `library($PACKAGE)` packages aren’t installed, you can
install them by running `install.packages("$PACKAGE")`, replacing
`$PACKAGE` with the package name.

``` r
# install omopcept from Github if not installed
if (!requireNamespace("omopcept", quietly=TRUE)) 
{
  if (!requireNamespace("remotes", quietly=TRUE)) install.packages("remotes")
  
  remotes::install_github("SAFEHR-data/omopcept")
}

library(readr)
library(dplyr)
library(here)
library(gh)
library(omopcept)
library(ggplot2)
library(stringr)
library(lubridate)
```

## Downloading & reading in OMOP data

Here we will download & read in some UCLH critical care data stored in
Github (if they are not already present from a previous download).

``` r
repo <- "SAFEHR-data/uclh-research-discovery"
path <- "_projects/uclh_cchic_s0/data"
destdata <- here("dynamic-docs/02-omop-walkthrough-critical-care/data")

# only download if not already present
if (! file.exists(file.path(destdata,"person.csv")))
{
  # Make GitHub API request to list contents of given path
  response <- gh::gh(glue::glue("/repos/{repo}/contents/{path}"))

  # Download all files to the destination dir
  purrr::walk(response, ~ download.file(.x$download_url, destfile = file.path(destdata, .x$name)))

  list.files(destdata)
}
```

## Reading in the OMOP data

We now have the OMOP data as a series of csv files in a folder.

The [omopcept package](https://github.com/SAFEHR-data/omopcept)
developed at UCLH, and installed above, has a function for reading in
OMOP tables to a single list object.

You could alternatively use
[CDMConnector::cdm_from_files](https://darwin-eu.github.io/CDMConnector/reference/cdm_from_files.html)
to access OMOP data from files like this and
[CDMConnector::cdm_from_con](https://darwin-eu.github.io/CDMConnector/reference/cdm_from_con.html)
if your OMOP data are in a database. We aim to add documentation about
that here soon.

``` r
omop <- omopcept::omop_cdm_read(destdata, filetype="csv")

# names() can show us names of the tables read in
names(omop)
```

    ##  [1] "condition_occurrence" "death"                "device_exposure"     
    ##  [4] "drug_exposure"        "measurement"          "observation"         
    ##  [7] "observation_period"   "person"               "procedure_occurrence"
    ## [10] "specimen"             "visit_occurrence"

## Looking at the `person` table

The OMOP tables are stored as data frames within the list object & can
be accessed by the table name.

Thus we can find the names of the columns in `person`, use `glimpse()`
to preview the data and `ggplot()` plot some of them. Note that not all
columns contain data and in that case are filled with `NA`.

``` r
# names() can also show column names for one of the tables
names(omop$person)
```

    ##  [1] "person_id"                   "gender_concept_id"          
    ##  [3] "year_of_birth"               "month_of_birth"             
    ##  [5] "day_of_birth"                "birth_datetime"             
    ##  [7] "race_concept_id"             "ethnicity_concept_id"       
    ##  [9] "location_id"                 "provider_id"                
    ## [11] "care_site_id"                "person_source_value"        
    ## [13] "gender_source_value"         "gender_source_concept_id"   
    ## [15] "race_source_value"           "race_source_concept_id"     
    ## [17] "ethnicity_source_value"      "ethnicity_source_concept_id"

``` r
# glimpse table data
glimpse(omop$person)
```

    ## Rows: 100
    ## Columns: 18
    ## $ person_id                   <int> 2451, 2452, 2453, 2454, 2455, 2456, 2457, …
    ## $ gender_concept_id           <int> 8532, 8532, 8507, 8507, 8507, 8507, 8507, …
    ## $ year_of_birth               <int> 1947, 1945, 1985, 1948, 1946, 1973, 1979, …
    ## $ month_of_birth              <int> 3, 1, 2, 11, 1, 11, 12, 10, 4, 3, 4, 8, 5,…
    ## $ day_of_birth                <int> 18, 2, 13, 22, 19, 18, 6, 16, 17, 3, 18, 1…
    ## $ birth_datetime              <dttm> 1947-03-18 11:34:00, 1945-01-02 19:12:41,…
    ## $ race_concept_id             <int> 46285839, 46285825, 46286810, 46286810, 46…
    ## $ ethnicity_concept_id        <int> 38003564, 38003564, 38003563, 38003563, 38…
    ## $ location_id                 <int> 97, 92, 70, 32, 91, 93, 1, 40, 30, 16, 26,…
    ## $ provider_id                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ care_site_id                <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ person_source_value         <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ gender_source_value         <chr> "FEMALE", "FEMALE", "MALE", "MALE", "MALE"…
    ## $ gender_source_concept_id    <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ race_source_value           <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ race_source_concept_id      <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ ethnicity_source_value      <chr> "Not Hispanic or Latino", "Not Hispanic or…
    ## $ ethnicity_source_concept_id <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
# plot some columns, patient birth years by gender
ggplot(omop$person, aes(x=year_of_birth, fill = as.factor(gender_concept_id))) +
  geom_bar() +
  theme_minimal()
```

![](/home/runner/work/starter-guide/starter-guide/02-omop-walkthrough-critical-care_files/figure-gfm/explore-person-1.png)<!-- -->

In the plot above bars are coloured by `gender_concept_id` which is the
OMOP ID for gender, but we don’t actually know which is which. We will
look at resolving that by retrieving OMOP concept names in the next
section.

## Getting names for OMOP concept IDs

To get the names for these and other concept IDs we need to use the OMOP
vocabularies that store concept IDs and names. In some cases an OMOP
database will contain the vocabularies as extra tables. In this case the
vocabularies are not provided with the csv files.

One way to add concept names is to use a function called
`omop_join_name_all()` from the [omopcept
package](https://github.com/SAFEHR-data/omopcept). It will add columns
containing concept names for all columns identified as containing
concept ids (based on the column name). It works on single tables but
will also accept a list of tables and add name columns to all of them
(it can take a good few seconds to join on all the name columns).

``` r
# join name columns onto all tables
omop_named <- omop |> omop_join_name_all()

# the names columns that have been added
names(omop_named$person) |> str_subset("name")
```

    ## [1] "gender_concept_name"    "race_concept_name"      "ethnicity_concept_name"

``` r
# now the gender name column can be used in the plot
ggplot(omop_named$person, aes(x=year_of_birth, fill=as.factor(gender_concept_name))) +
  geom_bar() +
  theme_minimal()
```

![](/home/runner/work/starter-guide/starter-guide/02-omop-walkthrough-critical-care_files/figure-gfm/omop-join-names-all-1.png)<!-- -->

## Looking at the `measurement` table

We can use the `measurement_concept_name` column (that was added by
`omop_join_name_all()` above) to see which are the most common
measurements.

``` r
glimpse(omop_named$measurement)
```

    ## Table
    ## 100 rows x 24 columns
    ## $ measurement_id                       <int32> 2448, 2449, 2450, 2451, 2452, 245…
    ## $ person_id                            <int32> 2451, 2452, 2453, 2454, 2455, 245…
    ## $ measurement_concept_id               <int32> 45757366, 45773395, 45763689, 457…
    ## $ measurement_concept_name            <string> "Age at smoking cessation", "Anio…
    ## $ measurement_date               <date32[day]> 1996-06-26, 1985-11-07, 1995-02-0…
    ## $ measurement_datetime <timestamp[us, tz=UTC]> 1996-06-26 12:26:10, 1985-11-07 0…
    ## $ measurement_time                      <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ measurement_type_concept_id          <int32> 32817, 32817, 32817, 32817, 32817…
    ## $ measurement_type_concept_name       <string> "EHR", "EHR", "EHR", "EHR", "EHR"…
    ## $ operator_concept_id                  <int32> 4172704, 4171756, 4171756, 417175…
    ## $ operator_concept_name               <string> ">", "<", "<", ">=", ">=", ">", "…
    ## $ value_as_number                      <int32> 91, 83, 89, 60, 115, 124, 34, 34,…
    ## $ value_as_concept_id                   <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ unit_concept_id                      <int32> 8555, 8496, 8496, 8648, 8547, 851…
    ## $ unit_concept_name                   <string> "second", "femtoliter platelet me…
    ## $ range_low                             <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ range_high                            <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ provider_id                           <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ visit_occurrence_id                  <int32> 2451, 2452, 2453, 2454, 2455, 245…
    ## $ visit_detail_id                       <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ measurement_source_value              <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ measurement_source_concept_id         <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ unit_source_value                     <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ value_source_value                    <bool> NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## Call `print()` for full schema details

``` r
# most frequent measurement concepts
count(omop_named$measurement, measurement_concept_name, sort=TRUE)
```

    ## Table (query)
    ## measurement_concept_name: string
    ## n: int64
    ## 
    ## * Sorted by n [desc]
    ## See $.data for the source Arrow object

## Looking at the `observation` table

We can use the `observation_concept_name` column (that was added by
`omop_join_name_all()` above) to see which are the most common
observations.

``` r
glimpse(omop_named$observation)
```

    ## Rows: 100
    ## Columns: 23
    ## $ observation_id                  <int> 2444, 2446, 2448, 2451, 2453, 2454, 24…
    ## $ person_id                       <int> 2452, 2454, 2456, 2459, 2461, 2462, 24…
    ## $ observation_concept_id          <int> 706011, 704996, 715751, 703437, 723488…
    ## $ observation_concept_name        <chr> "Metastasis", "Patient meets COVID-19 …
    ## $ observation_date                <date> 1975-02-21, 2009-03-31, 2021-10-16, 1…
    ## $ observation_datetime            <dttm> 1975-02-21 18:56:45, 2009-03-31 17:14…
    ## $ observation_type_concept_id     <int> 32868, 32840, 32813, 32818, 32830, 328…
    ## $ observation_type_concept_name   <chr> "Payer system record (secondary payer)…
    ## $ value_as_number                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ value_as_string                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ value_as_concept_id             <int> 715268, 715740, 706014, 710686, 703430…
    ## $ value_as_concept_name           <chr> "COVID-19 Intubation Procedure note", …
    ## $ qualifier_concept_id            <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ unit_concept_id                 <int> 44777585, 44777541, 44777573, 44777545…
    ## $ unit_concept_name               <chr> "million allergen-specific IgE antibod…
    ## $ provider_id                     <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ visit_occurrence_id             <int> 2452, 2454, 2456, 2459, 2461, 2462, 24…
    ## $ visit_detail_id                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ observation_source_value        <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ observation_source_concept_id   <int> 715745, 706008, 715738, 710690, 706004…
    ## $ observation_source_concept_name <chr> "Advice given about 2019-nCoV (novel c…
    ## $ unit_source_value               <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    ## $ qualifier_source_value          <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
# most frequent observation concepts
count(omop_named$observation, observation_concept_name, sort=TRUE)
```

    ## # A tibble: 55 × 2
    ##    observation_concept_name                                                    n
    ##    <chr>                                                                   <int>
    ##  1 <NA>                                                                        9
    ##  2 Assault by unspecified gases and vapours                                    4
    ##  3 City of travel [Location]                                                   4
    ##  4 Advice given about 2019-nCoV (novel coronavirus) infection by telephone     3
    ##  5 Assault by other specified gases and vapours                                3
    ##  6 COVID-19 Intubation Progress note                                           3
    ##  7 Exposure to 2019-nCoV (novel coronavirus) infection                         3
    ##  8 Family history with explicit context pertaining to sibling                  3
    ##  9 Metastasis                                                                  3
    ## 10 Retired SNOMED UK Drug extension concept, do not use, use concept indi…     3
    ## # ℹ 45 more rows

## Looking at the `drug_exposure` table

We can use the `drug_concept_name` column (that was added by
`omop_join_name_all()` above) to see which are the most common drugs.

``` r
glimpse(omop_named$drug_exposure)
```

    ## Rows: 100
    ## Columns: 27
    ## $ drug_exposure_id             <int> 2443, 2444, 2445, 2446, 2447, 2448, 2449,…
    ## $ person_id                    <int> 2451, 2452, 2453, 2454, 2455, 2456, 2457,…
    ## $ drug_concept_id              <int> 44818407, 44818490, 44818425, 44818479, 4…
    ## $ drug_concept_name            <chr> "potassium bicarbonate 25 MEQ Effervescen…
    ## $ drug_exposure_start_date     <date> 2004-06-11, 1981-12-14, 1994-10-31, 1961…
    ## $ drug_exposure_start_datetime <dttm> 2004-06-11 17:28:54, 1981-12-14 08:28:09…
    ## $ drug_exposure_end_date       <date> 2009-07-12, 1990-01-08, 1996-06-07, 1987…
    ## $ drug_exposure_end_datetime   <dttm> 2009-07-12 02:31:47, 1990-01-08 15:33:59…
    ## $ verbatim_end_date            <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ drug_type_concept_id         <int> 38000180, 43542358, 32426, 38000177, 3800…
    ## $ drug_type_concept_name       <chr> "Inpatient administration", "Physician ad…
    ## $ stop_reason                  <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ refills                      <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ quantity                     <int> 6, 6, 6, 6, 5, 6, 5, 5, 5, 6, 6, 6, 5, 5,…
    ## $ days_supply                  <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ sig                          <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ route_concept_id             <int> 4156705, 4167540, 40492305, 40490898, 600…
    ## $ route_concept_name           <chr> "Intracardiac", "Enteral", "Intrapericard…
    ## $ lot_number                   <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ provider_id                  <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ visit_occurrence_id          <int> 2451, 2452, 2453, 2454, 2455, 2456, 2457,…
    ## $ visit_detail_id              <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ drug_source_value            <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ drug_source_concept_id       <int> 21183732, 21179371, 21217483, 21182589, 2…
    ## $ drug_source_concept_name     <chr> "Noristerat 200mg/1ml solution for inject…
    ## $ route_source_value           <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ dose_unit_source_value       <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…

``` r
# most frequent drug_exposure concepts
count(omop_named$drug_exposure, drug_concept_name, sort=TRUE)
```

    ## # A tibble: 61 × 2
    ##    drug_concept_name                                                           n
    ##    <chr>                                                                   <int>
    ##  1 Lotus corniculatus flower volatile oil                                      5
    ##  2 thyroid (USP) 81.25 MG Oral Tablet [Nature-Throid]                          4
    ##  3 Amomum villosum var. xanthioides whole extract                              3
    ##  4 Glycyrrhiza uralensis whole extract                                         3
    ##  5 hydrocortisone 0.5 MG/ML / lidocaine hydrochloride 0.5 MG/ML / pramoxi…     3
    ##  6 salicylic acid 2.5 MG/ML / sodium thiosulfate 1 MG/ML Medicated Shampoo     3
    ##  7 toltrazuril                                                                 3
    ##  8 zinc sulfate 125 MG Effervescent Oral Tablet                                3
    ##  9 0.1 ML influenza A virus vaccine, A-Victoria-361-2011 (H3N2)-like viru…     2
    ## 10 Aesculus hippocastanum seed oil                                             2
    ## # ℹ 51 more rows

## Looking at the `visit_occurrence` table

The `visit_occurrence` table contains times and attributes of visits.
Other tables (e.g. `measurement` & `observation`) have a
`visit_occurrence_id` column that can be used to establish the visit
that they were associated with. Visits have a start & end date, in these
synthetic data the interval between them can be substantial.

``` r
glimpse(omop_named$visit_occurrence)
```

    ## Rows: 100
    ## Columns: 22
    ## $ visit_occurrence_id           <int> 2451, 2452, 2453, 2454, 2455, 2456, 2457…
    ## $ person_id                     <int> 2451, 2452, 2453, 2454, 2455, 2456, 2457…
    ## $ visit_concept_id              <int> 9203, 9203, 9203, 9203, 9201, 9201, 9201…
    ## $ visit_concept_name            <chr> "Emergency Room Visit", "Emergency Room …
    ## $ visit_start_date              <date> 1992-07-23, 1949-01-14, 1994-02-10, 195…
    ## $ visit_start_datetime          <dttm> 1992-07-23 21:20:23, 1949-01-14 22:11:4…
    ## $ visit_end_date                <date> 2015-09-09, 1991-01-29, 1998-04-24, 201…
    ## $ visit_end_datetime            <dttm> 2015-09-09 19:12:40, 1991-01-29 22:07:5…
    ## $ visit_type_concept_id         <int> 32817, 32817, 32817, 32817, 32817, 32817…
    ## $ visit_type_concept_name       <chr> "EHR", "EHR", "EHR", "EHR", "EHR", "EHR"…
    ## $ provider_id                   <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    ## $ care_site_id                  <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    ## $ visit_source_value            <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    ## $ visit_source_concept_id       <int> 9203, 9201, 9203, 9201, 9203, 9201, 9203…
    ## $ visit_source_concept_name     <chr> "Emergency Room Visit", "Inpatient Visit…
    ## $ admitting_source_concept_id   <int> 8882, 8536, 8761, 8970, 8546, 8905, 8968…
    ## $ admitting_source_concept_name <chr> "Adult Living Care Facility", "Home", "R…
    ## $ admitting_source_value        <chr> "Adult Living Care Facility", "Home", "R…
    ## $ discharge_to_concept_id       <int> 8716, 8615, 8977, 8870, 8536, 8905, 5814…
    ## $ discharge_to_concept_name     <chr> "Independent Clinic", "Assisted Living F…
    ## $ discharge_to_source_value     <chr> "Independent Clinic", "Assisted Living F…
    ## $ preceding_visit_occurrence_id <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …

``` r
# plot timeline of visit starts
omop_named$visit_occurrence |>
  #ggplot(aes(x=visit_start_date)) + geom_bar() +
  #replacing above with year tally otherwise bars too narrow to be reliably visible
  group_by(visit_concept_name, year = lubridate::floor_date(visit_start_date, "year")) |>
  summarize(nvisits = n()) |>
  ungroup() |> 
  ggplot(aes(x = year, y = nvisits, fill=visit_concept_name)) +
  geom_col() +
  facet_grid(vars(as.factor(visit_concept_name))) +
  theme_minimal() +
  theme(legend.position = "none")
```

![](/home/runner/work/starter-guide/starter-guide/02-omop-walkthrough-critical-care_files/figure-gfm/explore-visit_occurrence-1.png)<!-- -->

## Joining `person` data to other tables

The OMOP common data model is person centred. Most tables have a
`person_id` column that can be used to relate these data to other
attributes of the patient. Here we show how we can join the
`measurement` and `person` tables to see if there is any gender
difference in measurements. A similar approach could be used to join to
other tables including `observation` & `drug_exposure`.

``` r
joined_mp <- omop_named$measurement |> 
  left_join(omop_named$person, by="person_id")

freq_top_measures <- joined_mp |> 
  count(measurement_concept_name,gender_concept_name, sort=TRUE) |> 
  filter(n > 1) |>
  # We have to collect before plotting because the joined data with arrow
  # has lazy loading and can't be coerced into a dataframe
  # we explicitly set this to convert into dataframe before plotting
  collect()

#plot
freq_top_measures |> 
  ggplot(aes(y=measurement_concept_name, x=n, fill=as.factor(gender_concept_name))) +
    geom_col() +
    facet_wrap(vars(as.factor(gender_concept_name))) +
    theme_minimal() +
    theme(legend.position = "none")
```

![](/home/runner/work/starter-guide/starter-guide/02-omop-walkthrough-critical-care_files/figure-gfm/join-person-measurement-1.png)<!-- -->

Note that we use `left_join` here because we only want to join on
`person` information for rows occurring in the `measurement` table which
is the left hand argument of the join. Also note that in this example we
end up with one row per patient because the synthetic `measurement`
table only has one row per patient. Usually we would expect multiple
measurements per patient that would result in multiple rows per patient
in the joined table.

## Differences between these synthetic data and real patient data

These particular synthetic data are useful to demonstrate the reading in
and manipulation of OMOP data but there are some major differences
between them and real patient data.

1.  `person`, `measurement`, `observation` & `drug_exposure` tables are
    all same length (100 rows), in real data one would expect many more
    measurements, observations & drug exposures than patients
2.  Related to 1, in these data there are a single `measurement`,
    `observation` and `drug_exposure` per patient. In reality one would
    expect many tens or hundreds of these other values per patient.
