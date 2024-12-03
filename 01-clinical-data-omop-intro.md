# Intro to clinical data at UCLH & OMOP

<!-- 
  for comments that won't appear online 
  - current links for OMOP CDM are 5.3 as that's the version we are currently using
-->

This page provides a brief introduction and getting started guide for clinical data at UCLH. 

### OMOP outline

UCLH clinical data is, by default, provided for research in a format called the [OMOP Common Data Model (CDM)](https://ohdsi.github.io/CommonDataModel/), sometimes just OMOP for short.

The beauty of the CDM is that it allows for data from different locations to be combined. As OMOP is used by a large and growing number of researchers globally this opens up potential for collaboration and to contribute data to network studies. In addition there are analytical tools that run directly from CDM data including ones provided by [OHDSI](https://ohdsi.github.io/Hades/), the organisation that manages OMOP, and the [Darwin EU project](https://github.com/darwin-eu/). 

OMOP consists of two parts :

1. **a data structure** defining required tables and columns.
1. **a vocabulary** offering standardised IDs and names for nearly everything that can happen in a hospital. OMOP includes many other vocabularies (e.g. [SNOMED](https://www.snomed.org/what-is-snomed-ct), [LOINC](https://loinc.org/) etc.) by assigning a unique OMOP concept id to each of their IDs.

The first part could be thought of as a filing cabinet with sections where data can go, the second part as a dictionary, allowing data elements to be looked up and standardised. This [2024 paper describes the OMOP vocabs](https://pmc.ncbi.nlm.nih.gov/articles/PMC10873827/).    


### Clinical data from UCLH

UCLH clinical data are provided as a series of tables in the OMOP format. These can be provided in either parquet or csv format, or already uploaded into a database.    

Here you can find details of the [tables and columns making up an OMOP instance](https://ohdsi.github.io/CommonDataModel/cdm53.html#person).


### A simple OMOP example

Here we will work through a reduced OMOP example to introduce you to how you can use the data.

We will start by considering four of the OMOP tables and selected fields/columns in each :

1. `Person`
1. `Measurement`
1. `Observation`
1. `Drug_exposure`

#### `Person` and `Measurement` tables.   

`Person` has one row per patient and a column called `person_id` that can link rows in most of the other tables to an individual. It also stores attributes of the individual including birth date, gender and ethnicity.    
    
`Measurement` has one row per measurement conducted on the patient and has columns storing the ID of the patient, the identity of the measurement, when it was conducted and the value recorded.    

OMOP Table  | Selected Table Columns
------------- | -------------
Person  | `person_id`  `year_of_birth`  `gender_concept_id`  `gender_source_value`
Measurement  |  `person_id`  `measurement_id`  `measurement_concept_id`  `measurement_date`  `value_as_number`  `value_as_concept_id`  `measurement_source_value`

#### OMOP concept IDs and names

Any column named `*concept_id` contains OMOP concept IDs, integer values that are defined in the OMOP vocabulary where a corresponding name is stored for each ID.   

There are different ways of looking up the concept names from IDs. One way is to use the [omopcept R package](https://github.com/SAFEHR-data/omopcept). `omopcept` provides a function [omop_join_name_all()](https://github.com/SAFEHR-data/omopcept/blob/f1a484623103bcc88cc91649c631b1694c1724bb/R/omop_join_name.R#L157) that will add `concept_name` columns for all OMOP `concept_id` columns in a table or list of tables. For example for the slimmed down `Person` table described above it would add a `gender_concept_name` column. These are the concepts for male and female.

`gender_concept_id`  | `gender_concept_name`
------------- | -------------
8532  | FEMALE  
8507  | MALE 
    
OMOP concepts can also be looked up in [Athena](https://athena.ohdsi.org) an online tool provided by OHDSI, but this manual process would take a long time for more than a few concepts and is less reproducible.

#### `concept_id` is the unique OMOP ID `concept_code` is the ID in one of the source vocabularies e.g. SNOMED or LOINC

The OMOP vocabularies have a `concept_code` field that contains the identifier in the source vocabulary, but most often you will want to use **concept_id** which is the unique OMOP ID.

#### Beware of using `*source_value` columns

You may notice that there are columns named `*_source_value` in both the slimmed `Person` and `Measurement` tables. These store the values as recorded in the source data before it was mapped to OMOP (the values stored in EPIC in the case of UCLH). You may be tempted to use source value columns in your analyses but this is not recommended. The benefit of using OMOP is that you can combine your data/analysis with other sites because of the standardisation. If you use `*_source_value` columns in your analysis you lose the benefit of standardisation. Using source values makes it unlikely that you'll be able to use data from another site in your analysis. 


#### Joining patient identifiers and attributes onto other data (e.g. measurements)

To look at patient attributes associated with measurements (or other omop tables) you can join the `person` table onto the `measurement` table using `person_id`.
In R, code like this could be used to do the join :

```
library(dplyr)
mp <- Measurement |> 
      left_join(Person, by="person_id")
```

Be slightly careful that this creates a table that has multiple rows per patient ID.

#### Measurement values

Measurements are stored in a question-answer format. The question is represented by `measurement_concept_id` & `measurement_concept_name`. Answers are represented in `value_as_number` for numeric values & `value_as_concept_id` for values that can be represented by another `concept_id`.

### `Drug_exposure` and `Observation` tables

The `Drug_exposure` and `Observation` tables can be treated similarly to the `Measurement` table. These are some of the most useful fields.

OMOP Table  | Selected Table Columns
------------- | -------------
Person  | `person_id`  `year_of_birth`  `gender_concept_id`  `gender_source_value`
Drug_exposure | `person_id`  `drug_exposure_id`  `drug_concept_id`  `drug_exposure_start_date`  `drug_exposure_end_date`  `quantity` `drug_source_value`
Observation | `person_id`  `observation_id`  `observation_concept_id`  `observation_date`  `value_as_number`  `value_as_concept_id` `value_as_string` `observation_source_value`

Note that `Observation` has an additional column `value_as_string` that is not present in `Measurement`.

### OMOP `Standard` concepts

For any clinical event OHDSI defines a single **Standard** `concept_id`. Whilst clinical events may be represented by a range of vocabularies (e.g. SNOMED, LOINC, ICD10) only one will be `Standard`. For example the Standard vocabulary for conditions is SNOMED and for drugs is RxNorm or RxNorm Extension.  Non-standard concepts can be included in `source*` fields but should not be present in `*concept_id` fields.

### Next OMOP steps

This has been a brief introduction to OMOP at UCLH. YOu can explore the links below to learn more. Also we will will be providing more detailed documentation soon.

### Useful links (repeated from the text above)

1.   [OMOP tables and columns](https://ohdsi.github.io/CommonDataModel/cdm53.html/#person)

1.   [The Book Of OHDSI](https://ohdsi.github.io/TheBookOfOhdsi/) A useful comprehensive community resource describing all things OHDSI & OMOP. A little dated now (from 2021).

1.   [Athena](https://athena.ohdsi.org) - online OMOP concept lookup provided by OHDSI

1.   [OMOP CDM](https://ohdsi.github.io/CommonDataModel/) - Observational Medical Outcomes Partnership (OMOP) Common Data Model (CDM) is an open community data standard, designed to standardize the structure and content of observational data and to enable efficient analyses. UCLH using OMOP since 2022 to provide data for research

1.   [OHDSI](https://www.ohdsi.org/) - Pronounced Odyssey. Observational Health Data Sciences and Informatics program. Maintains the OMOP Common Data Model

1.   [omopcept](https://github.com/SAFEHR-data/omopcept) - an R package for querying and visualising **omop** con**cept**s (with fewer cons!). Developed by Andy South at UCLH.

1.   [SNOMED CT](https://digital.nhs.uk/services/terminology-and-classifications/snomed-ct) - A structured clinical vocabulary. All NHS healthcare providers in England must use SNOMED CT for capturing clinical terms within electronic patient record systems. OMOP has a representation of SNOMED concepts.

1.   [2024 paper describing OMOP vocabs](https://pmc.ncbi.nlm.nih.gov/articles/PMC10873827/)

1.   [OHDSI community forums](https://forums.ohdsi.org/) where you can browse & ask community questions
