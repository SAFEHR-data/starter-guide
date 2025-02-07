# Outline of how drugs are coded in UCLH, standardised to OMOP and can be classified using ATC

author: Andy South
date: "2024-12-03"


<!-- *.md is generated from `*.Rmd` in /dynamic-docs/, to update edit `*.Rmd`, re-knit, copy `*.md` & the `*_files` folder to root, delete YAML header so it displays better in Github, don't delete `.html` because that will delete folder and images used by the md. We can automate this process later. -->


This document introduces how drug data are stored in UCLH, how they are standardised to OMOP and how they can be classified into generic groupings (e.g. ANTIBACTERIALS FOR SYSTEMIC USE) using [ATC](https://www.nlm.nih.gov/research/umls/rxnorm/sourcereleasedocs/atc.html) (Anatomical Therapeutic Chemical Classification System) a WHO drug classification incorporated within OMOP.  



## Installing & loading required packages

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
library(knitr) #for kable
```

## Reading in OMOP drug data 

Here we will read in some example drug data stored in this repository. These data are a subset of rows and columns from the OMOP `drug_exposure` table with 1 row per unique drug and only those columns that are useful here.



``` r
#TODO copy drugs_omop_unique.csv from omop_analyses & look at it

datafolder <- here("dynamic-docs/03-drugs-uclh-omop/data")

#using omopcept to read in a table, could use readr
drugs_unique <- omop_cdm_table_read("drugs_omop_unique",path = datafolder, filetype = "csv")

# names() can show us names of the tables read in
names(drugs_unique)
```

```
##  [1] "drug_concept_id"          "drug_concept_name"       
##  [3] "drug_type_concept_id"     "drug_type_concept_name"  
##  [5] "quantity"                 "route_concept_id"        
##  [7] "route_concept_name"       "drug_source_value"       
##  [9] "drug_source_concept_id"   "drug_source_concept_name"
## [11] "route_source_value"       "dose_unit_source_value"
```

``` r
# join on vocabulary_id for concept_id & maybe drug_source_concept_id (expect all to be dm+d)

drugs_unique <- drugs_unique |> 
  #remove concept_name temporarily due to issue in omopcept
  select(-drug_concept_name) |> 
  omopcept::omop_join_name(namefull="drug_concept_id",columns=c("concept_name","domain_id","vocabulary_id","concept_class_id"))



#this is how file created, may be better to move join_names bit into here & make the data file closer to
#the drug_exposure table
# drugs_unique <- omde |> 
#   #group_by(drug_concept_id) |> 
#   #slice_head(n=1) #|> 
#   #select(drug_concept_id) |> 
#   #count replaces above 3 lines & gives num rows
#   count(drug_concept_id, sort=TRUE) |> 
#   omopcept::omop_join_name_all(columns=c("concept_name","domain_id","vocabulary_id","concept_class_id"))

# #to use in documentation
# write_csv(drugs_unique, "drugs_omop_unique.csv")
```


These data show us that drugs exported to OMOP from UCLH are in the vocabularies either `RxNorm` or `RxNormExtension` & concept_class_id of mostly either `Clinical Drug` or `Quant Clinical Drug`.


``` r
#note kable is a knitr function to display tables in a markdown file
count(drugs_unique, vocabulary_id, sort=TRUE) |> kable()
```



|vocabulary_id    |    n|
|:----------------|----:|
|RxNorm Extension | 1005|
|RxNorm           |  736|
|None             |    1|

``` r
count(drugs_unique, concept_class_id, sort=TRUE)  |> kable()
```



|concept_class_id    |    n|
|:-------------------|----:|
|Clinical Drug       | 1164|
|Quant Clinical Drug |  475|
|Marketed Product    |   85|
|Clinical Drug Form  |   14|
|Branded Drug Form   |    1|
|Clinical Drug Comp  |    1|
|Clinical Pack       |    1|
|Undefined           |    1|

These data show us that drug records come from different places and have different administration routes.


``` r
count(drugs_unique, drug_type_concept_name, sort=TRUE) |> kable()
```



|drug_type_concept_name    |    n|
|:-------------------------|----:|
|EHR administration record | 1135|
|EHR order                 |  472|
|EHR prescription          |  133|
|Patient self-report       |    2|

``` r
#filtering the top 10
count(drugs_unique, route_concept_name, sort=TRUE) |> filter(row_number()<11) |>  kable()
```



|route_concept_name  |   n|
|:-------------------|---:|
|Oral                | 819|
|Intravenous         | 306|
|Subcutaneous        | 143|
|Topical             |  95|
|No matching concept |  70|
|Respiratory trac    |  59|
|Ophthalmic          |  51|
|Intramuscula        |  43|
|Transdermal         |  33|
|Rectal              |  24|

## drug standardisation process at UCLH

Drug records at UCLH are stored in our Electronic Health Record (EPIC) with as a `MedicationKey` value. Our OMOP extraction system translates this to an identifier in the NHS dictionary of medicines and devices (**dm+d**). `dm+d` is included in OMOP so there are values of OMOP concept_id for each dm+d. However because dm+d is not a standard vocabulary in OMOP we need to translate it once more to get to a standard OMOP id that can potentially be used in collaborative studies. The standard vocabularies for drugs in OMOP are **RxNorm** and **RxNormExtension**. Thus the `drug_concept_id` in UCLH OMOP extracts will either be in `RxNorm` or `RxNormExtension`.

Note that the OMOP vocabularies have a dated representation of `dm+d` codes so some `dm+d` codes don't map to an OMOP or `RxNorm` equivalent.

## drug hierarchies

Different drug vocabularies each have a hierarchy that allows them to represent and query drugs at various levels of granularity. 

For example at increasing levels of granularity a vocabulary could represent, say, paracetomol as
1. a generic active ingredient
1. the ingredient in a particular form such as a tablet
1. a box of tablets of a defined size
1. a commercially available product of a defined size and manufacturer


## Anatomical Therapeutic Chemical Classification System (ATC) for drug classification

[ATC](https://www.nlm.nih.gov/research/umls/rxnorm/sourcereleasedocs/atc.html) is a WHO drug classification incorporated within OMOP. ATC has five different levels that classify according to the organ or system on which the drug acts plus therapeutic, pharmacological, and chemical properties of the drug. 	
	
ATC Level | Description | Number of codes | Example code | Example name
--|-----|---|-----|-----------------
1 |anatomical main group|14|A|Alimentary tract and metabolism
2 |therapeutic subgroup|94|A10|Drugs used in diabetes
3 |pharmacological subgroup|267|A10B|Blood glucose lowering drugs, excl. insulins
4 |chemical subgroup|889|A10BA|Biguanides
5 |chemical substance|5067|A10BA02|metformin

The OMOP vocabularies contain ATC and it can be used to classify and query drugs.
	
The [omopcept package](https://github.com/SAFEHR-data/omopcept) developed at UCLH, and installed above, has a function for creating a drug classification lookup table using ATC.	

Here we create the drug lookup and view the top rows.
	

``` r
drug_lookup <- omopcept::omop_drug_lookup_create(drugs_unique)

drug_lookup |> head(6) |> kable()  
```



|drug_concept_name                                                          | drug_concept_id|drug_concept_class_id |ATC_level |ATC_concept_name                                   |ATC_code | ATC_concept_id|
|:--------------------------------------------------------------------------|---------------:|:---------------------|:---------|:--------------------------------------------------|:--------|--------------:|
|10 ML immunoglobulin G 200 MG/ML Injectable Solution [Hizentra] by Behring |        36926058|Marketed Product      |1         |ANTIINFECTIVES FOR SYSTEMIC USE                    |J        |       21602795|
|100 ML Immunoglobulin G 100 MG/ML Injectable Solution                      |        44135078|Quant Clinical Drug   |1         |ANTIINFECTIVES FOR SYSTEMIC USE                    |J        |       21602795|
|sunitinib 12.5 MG Oral Capsule                                             |         1336541|Clinical Drug         |5         |sunitinib; oral                                    |L01EX01  |         947855|
|osimertinib 80 MG Oral Tablet                                              |        35605535|Clinical Drug         |5         |osimertinib; oral                                  |L01EB04  |         947878|
|ixazomib 3 MG Oral Capsule                                                 |        35606236|Clinical Drug         |5         |ixazomib; oral                                     |L01XG03  |         947997|
|canagliflozin 100 MG Oral Tablet                                           |        43526467|Clinical Drug         |4         |Sodium-glucose co-transporter 2 (SGLT2) inhibitors |A10BK    |        1123627|
	
We can select a single drug concept (`amoxicillin 500 MG Oral Capsule`) and see which ATC classes it appears in.	


``` r
drug_lookup |> 
  filter(drug_concept_name == "amoxicillin 500 MG Oral Capsule") |> 
  kable()  
```



|drug_concept_name               | drug_concept_id|drug_concept_class_id |ATC_level |ATC_concept_name                        |ATC_code | ATC_concept_id|
|:-------------------------------|---------------:|:---------------------|:---------|:---------------------------------------|:--------|--------------:|
|amoxicillin 500 MG Oral Capsule |        19073187|Clinical Drug         |1         |ANTIINFECTIVES FOR SYSTEMIC USE         |J        |       21602795|
|amoxicillin 500 MG Oral Capsule |        19073187|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE         |J01      |       21602796|
|amoxicillin 500 MG Oral Capsule |        19073187|Clinical Drug         |3         |BETA-LACTAM ANTIBACTERIALS, PENICILLINS |J01C     |       21602818|
|amoxicillin 500 MG Oral Capsule |        19073187|Clinical Drug         |4         |Penicillins with extended spectrum      |J01CA    |       21602819|
|amoxicillin 500 MG Oral Capsule |        19073187|Clinical Drug         |5         |amoxicillin; systemic                   |J01CA04  |       21602823|

Alternatively we can filter all drugs that appear in a particular ATC level and class (in this case "ANTIBACTERIALS FOR SYSTEMIC USE" from ATC level 2). 


``` r
antibacterials <- drug_lookup |> 
  filter(ATC_concept_name == "ANTIBACTERIALS FOR SYSTEMIC USE") 

nrow(antibacterials)
```

```
## [1] 69
```

``` r
#display top 10
antibacterials |> 
  filter(row_number()<11) |>  kable()
```



|drug_concept_name                                                | drug_concept_id|drug_concept_class_id |ATC_level |ATC_concept_name                |ATC_code | ATC_concept_id|
|:----------------------------------------------------------------|---------------:|:---------------------|:---------|:-------------------------------|:--------|--------------:|
|nitrofurantoin 5 MG/ML Oral Suspension                           |          920300|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|amoxicillin 500 MG / clavulanate 125 MG Oral Tablet              |         1713694|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|amoxicillin 50 MG/ML / clavulanate 12.5 MG/ML Oral Suspension    |         1759879|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|sulfamethoxazole 40 MG/ML / trimethoprim 8 MG/ML Oral Suspension |         1836449|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|floxacillin 250 MG Oral Capsule                                  |        19054940|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|floxacillin 500 MG Oral Capsule                                  |        19054961|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|azithromycin 250 MG Oral Tablet                                  |        19073777|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|cephalexin 250 MG Oral Tablet                                    |        19075035|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|Colistin 1000000 UNT Inhalant Powder                             |        21105909|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|
|cefiderocol 1000 MG Injectable Solution                          |        35885672|Clinical Drug         |2         |ANTIBACTERIALS FOR SYSTEMIC USE |J01      |       21602796|


TO BE CONTINUED ...
DRAFT MATERIAL BELOW
TODO 
is this true : Note that the `dm+d` codes are more granular than `RxNorm` so the mapping can go from 1 to many.

add link to drug_exposure table description :
https://ohdsi.github.io/CommonDataModel/cdm54.html#drug_exposure

check issue in omopcept when concept_name not included in join columns

This is where uclh drugs are mapped in omop_es
https://github.com/uclh-criu/omop_es/blob/3c36642b3a24944da7feb0fd214088241cf1990b/mapping/UCLH/epic_common/medication_component_common.R

Comments there : 

Maps from EPIC MedicationKey to OMOP concept_id and, if applicable, a standard drug_concept_id via the following stages:

EPIC MedKey -> dm+d -> dm+d concept (OMOP Non-Std) -> RxNORM **Ext** OMOP (OMOP Std)

Note that this is a 1-M-M mapping process

For this reason a score is assigned that ranks full over partial mapping and devices over drugs

The dm+d concept_id (omop?) should be in drug_source_concept_id

also route_concept_id & route_source_concept_id should be useful
