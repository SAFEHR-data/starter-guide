<!-- *.md is generated from `*.Rmd` in /dynamic-docs/, to update edit `*.Rmd`, re-knit, copy `*.md` & the `*_files` folder to root, delete YAML header so it displays better in Github, don't delete `.html` because that will delete folder and images used by the md. We can automate this process later. -->

This document introduces how drug data are stored in UCLH, how they are
standardised to OMOP and how they can be classified into generic
groupings (e.g. ANTIBACTERIALS FOR SYSTEMIC USE) using
[ATC](https://www.nlm.nih.gov/research/umls/rxnorm/sourcereleasedocs/atc.html)
(Anatomical Therapeutic Chemical Classification System), a WHO drug
classification incorporated within OMOP.

## Installing & loading required packages

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
    library(DiagrammeR)

## Reading in OMOP drug data

Here we will read in some example drug data stored in this repository.
These data are a subset of rows and columns from the OMOP
`drug_exposure` table with 1 row per unique drug and only those columns
that are useful here.

    #TODO copy drugs_omop_unique.csv from omop_analyses & look at it

    datafolder <- here("dynamic-docs/03-drugs-uclh-omop/data")

    #using omopcept to read in a table, could use readr
    drugs_unique <- omop_cdm_table_read("drugs_omop_unique",path = datafolder, filetype = "csv")

    # names() can show us names of the tables read in
    names(drugs_unique)

    ##  [1] "drug_concept_id"          "drug_concept_name"       
    ##  [3] "drug_type_concept_id"     "drug_type_concept_name"  
    ##  [5] "quantity"                 "route_concept_id"        
    ##  [7] "route_concept_name"       "drug_source_value"       
    ##  [9] "drug_source_concept_id"   "drug_source_concept_name"
    ## [11] "route_source_value"       "dose_unit_source_value"

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

These data show us that drugs exported to OMOP from UCLH are in the
vocabularies either `RxNorm` or `RxNormExtension` & concept\_class\_id
of mostly either `Clinical Drug` or `Quant Clinical Drug`.

    #note kable is a knitr function to display tables in a markdown file
    count(drugs_unique, vocabulary_id, sort=TRUE) |> kable()

<table>
<thead>
<tr class="header">
<th style="text-align: left;">vocabulary_id</th>
<th style="text-align: right;">n</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">RxNorm Extension</td>
<td style="text-align: right;">1005</td>
</tr>
<tr class="even">
<td style="text-align: left;">RxNorm</td>
<td style="text-align: right;">736</td>
</tr>
<tr class="odd">
<td style="text-align: left;">None</td>
<td style="text-align: right;">1</td>
</tr>
</tbody>
</table>

    count(drugs_unique, concept_class_id, sort=TRUE)  |> kable()

<table>
<thead>
<tr class="header">
<th style="text-align: left;">concept_class_id</th>
<th style="text-align: right;">n</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: right;">1164</td>
</tr>
<tr class="even">
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: right;">475</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Marketed Product</td>
<td style="text-align: right;">85</td>
</tr>
<tr class="even">
<td style="text-align: left;">Clinical Drug Form</td>
<td style="text-align: right;">14</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Branded Drug Form</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">Clinical Drug Comp</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Clinical Pack</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">Undefined</td>
<td style="text-align: right;">1</td>
</tr>
</tbody>
</table>

These data show us that drug records come from different places and have
different administration routes.

    count(drugs_unique, drug_type_concept_name, sort=TRUE) |> kable()

<table>
<thead>
<tr class="header">
<th style="text-align: left;">drug_type_concept_name</th>
<th style="text-align: right;">n</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">EHR administration record</td>
<td style="text-align: right;">1135</td>
</tr>
<tr class="even">
<td style="text-align: left;">EHR order</td>
<td style="text-align: right;">472</td>
</tr>
<tr class="odd">
<td style="text-align: left;">EHR prescription</td>
<td style="text-align: right;">133</td>
</tr>
<tr class="even">
<td style="text-align: left;">Patient self-report</td>
<td style="text-align: right;">2</td>
</tr>
</tbody>
</table>

    #filtering the top 10
    count(drugs_unique, route_concept_name, sort=TRUE) |> filter(row_number()<11) |>  kable()

<table>
<thead>
<tr class="header">
<th style="text-align: left;">route_concept_name</th>
<th style="text-align: right;">n</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Oral</td>
<td style="text-align: right;">819</td>
</tr>
<tr class="even">
<td style="text-align: left;">Intravenous</td>
<td style="text-align: right;">306</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Subcutaneous</td>
<td style="text-align: right;">143</td>
</tr>
<tr class="even">
<td style="text-align: left;">Topical</td>
<td style="text-align: right;">95</td>
</tr>
<tr class="odd">
<td style="text-align: left;">No matching concept</td>
<td style="text-align: right;">70</td>
</tr>
<tr class="even">
<td style="text-align: left;">Respiratory trac</td>
<td style="text-align: right;">59</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Ophthalmic</td>
<td style="text-align: right;">51</td>
</tr>
<tr class="even">
<td style="text-align: left;">Intramuscula</td>
<td style="text-align: right;">43</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Transdermal</td>
<td style="text-align: right;">33</td>
</tr>
<tr class="even">
<td style="text-align: left;">Rectal</td>
<td style="text-align: right;">24</td>
</tr>
</tbody>
</table>

## drug standardisation process at UCLH

    graph TD;
      A[EPIC_ID] --> B[dm+d code];
      B --> C[OMOP concept_id dm+d];
      C --> D[OMOP concept_id RxNorm & Extension];

Drug records at UCLH are stored in our Electronic Health Record (EPIC)
with as a `MedicationKey` value. Our OMOP extraction system translates
this to an identifier in the NHS dictionary of medicines and devices
(**dm+d**). `dm+d` is included in OMOP so there are values of OMOP
concept\_id for each dm+d. However because dm+d is not a standard
vocabulary in OMOP we need to translate it once more to get to a
standard OMOP id that can potentially be used in collaborative studies.
The standard vocabularies for drugs in OMOP are **RxNorm** and
**RxNormExtension**. Thus the `drug_concept_id` in UCLH OMOP extracts
will either be in `RxNorm` or `RxNormExtension`.

Note that the OMOP vocabularies have a dated representation of `dm+d`
codes so some `dm+d` codes don’t map to an OMOP or `RxNorm` equivalent.

TODO add example with names & codes

<table>
<colgroup>
<col style="width: 27%" />
<col style="width: 37%" />
<col style="width: 23%" />
<col style="width: 11%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">omop_field_name</th>
<th style="text-align: left;">vocabulary</th>
<th style="text-align: left;">in_diagram_above</th>
<th style="text-align: left;">example</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">drug_concept_id</td>
<td style="text-align: left;">OMOP RxNorm/Extension ID</td>
<td style="text-align: left;">4</td>
<td style="text-align: left;">?</td>
</tr>
<tr class="even">
<td style="text-align: left;">drug_concept_name</td>
<td style="text-align: left;">OMOP RxNorm/Extension Name</td>
<td style="text-align: left;">4b</td>
<td style="text-align: left;">?</td>
</tr>
<tr class="odd">
<td style="text-align: left;">drug_source_concept</td>
<td style="text-align: left;">OMOP dm+d ID</td>
<td style="text-align: left;">3</td>
<td style="text-align: left;">?</td>
</tr>
<tr class="even">
<td style="text-align: left;">drug_source_value</td>
<td style="text-align: left;">dm+d ID</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">/</td>
</tr>
</tbody>
</table>

## drug hierarchies

Different drug vocabularies each have a hierarchy that allows them to
represent and query drugs at various levels of granularity.

For example at increasing levels of granularity a vocabulary could
represent, say, paracetomol as 1. a generic active ingredient 1. the
ingredient in a particular form such as a tablet 1. a box of tablets of
a defined size 1. a commercially available product of a defined size and
manufacturer

## Anatomical Therapeutic Chemical Classification System (ATC) for drug classification

[ATC](https://www.nlm.nih.gov/research/umls/rxnorm/sourcereleasedocs/atc.html)
is a WHO drug classification incorporated within OMOP. ATC has five
different levels that classify according to the organ or system on which
the drug acts plus therapeutic, pharmacological, and chemical properties
of the drug.

<table>
<colgroup>
<col style="width: 6%" />
<col style="width: 15%" />
<col style="width: 9%" />
<col style="width: 15%" />
<col style="width: 53%" />
</colgroup>
<thead>
<tr class="header">
<th>ATC Level</th>
<th>Description</th>
<th>Number of codes</th>
<th>Example code</th>
<th>Example name</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>anatomical main group</td>
<td>14</td>
<td>A</td>
<td>Alimentary tract and metabolism</td>
</tr>
<tr class="even">
<td>2</td>
<td>therapeutic subgroup</td>
<td>94</td>
<td>A10</td>
<td>Drugs used in diabetes</td>
</tr>
<tr class="odd">
<td>3</td>
<td>pharmacological subgroup</td>
<td>267</td>
<td>A10B</td>
<td>Blood glucose lowering drugs, excl. insulins</td>
</tr>
<tr class="even">
<td>4</td>
<td>chemical subgroup</td>
<td>889</td>
<td>A10BA</td>
<td>Biguanides</td>
</tr>
<tr class="odd">
<td>5</td>
<td>chemical substance</td>
<td>5067</td>
<td>A10BA02</td>
<td>metformin</td>
</tr>
</tbody>
</table>

The OMOP vocabularies contain ATC and it can be used to classify and
query drugs.

The [omopcept package](https://github.com/SAFEHR-data/omopcept)
developed at UCLH, and installed above, has a function for creating a
drug classification lookup table using ATC.

Here we create the drug lookup and view the top rows.

    drug_lookup <- omopcept::omop_drug_lookup_create(drugs_unique)

    drug_lookup |> arrange(drug_concept_name, ATC_level) |> head(6)  |> kable()

<table>
<colgroup>
<col style="width: 30%" />
<col style="width: 9%" />
<col style="width: 13%" />
<col style="width: 6%" />
<col style="width: 25%" />
<col style="width: 5%" />
<col style="width: 9%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">drug_concept_name</th>
<th style="text-align: right;">drug_concept_id</th>
<th style="text-align: left;">drug_concept_class_id</th>
<th style="text-align: left;">ATC_level</th>
<th style="text-align: left;">ATC_concept_name</th>
<th style="text-align: left;">ATC_code</th>
<th style="text-align: right;">ATC_concept_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">0.1 ML aflibercept 40 MG/ML Injectable
Solution</td>
<td style="text-align: right;">41404616</td>
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: left;">1</td>
<td style="text-align: left;">ANTINEOPLASTIC AND IMMUNOMODULATING
AGENTS</td>
<td style="text-align: left;">L</td>
<td style="text-align: right;">21601386</td>
</tr>
<tr class="even">
<td style="text-align: left;">0.1 ML aflibercept 40 MG/ML Injectable
Solution</td>
<td style="text-align: right;">41404616</td>
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTINEOPLASTIC AGENTS</td>
<td style="text-align: left;">L01</td>
<td style="text-align: right;">21601387</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0.1 ML aflibercept 40 MG/ML Injectable
Solution</td>
<td style="text-align: right;">41404616</td>
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: left;">3</td>
<td style="text-align: left;">OTHER ANTINEOPLASTIC AGENTS</td>
<td style="text-align: left;">L01X</td>
<td style="text-align: right;">21603746</td>
</tr>
<tr class="even">
<td style="text-align: left;">0.1 ML aflibercept 40 MG/ML Injectable
Solution</td>
<td style="text-align: right;">41404616</td>
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: left;">4</td>
<td style="text-align: left;">Other antineoplastic agents</td>
<td style="text-align: left;">L01XX</td>
<td style="text-align: right;">21603783</td>
</tr>
<tr class="odd">
<td style="text-align: left;">0.1 ML aflibercept 40 MG/ML Injectable
Solution</td>
<td style="text-align: right;">41404616</td>
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: left;">5</td>
<td style="text-align: left;">aflibercept; ophthalmic, parenteral</td>
<td style="text-align: left;">L01XX44</td>
<td style="text-align: right;">43534816</td>
</tr>
<tr class="even">
<td style="text-align: left;">0.2 ML Dalteparin 12500 UNT/ML Injectable
Solution</td>
<td style="text-align: right;">43864220</td>
<td style="text-align: left;">Quant Clinical Drug</td>
<td style="text-align: left;">1</td>
<td style="text-align: left;">BLOOD AND BLOOD FORMING ORGANS</td>
<td style="text-align: left;">B</td>
<td style="text-align: right;">21600959</td>
</tr>
</tbody>
</table>

We can select a single drug concept (`amoxicillin 500 MG Oral Capsule`)
and see which ATC classes it appears in.

    drug_lookup |> 
      filter(drug_concept_name == "amoxicillin 500 MG Oral Capsule") |> 
      kable()  

<table>
<colgroup>
<col style="width: 22%" />
<col style="width: 11%" />
<col style="width: 15%" />
<col style="width: 6%" />
<col style="width: 27%" />
<col style="width: 6%" />
<col style="width: 10%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">drug_concept_name</th>
<th style="text-align: right;">drug_concept_id</th>
<th style="text-align: left;">drug_concept_class_id</th>
<th style="text-align: left;">ATC_level</th>
<th style="text-align: left;">ATC_concept_name</th>
<th style="text-align: left;">ATC_code</th>
<th style="text-align: right;">ATC_concept_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">amoxicillin 500 MG Oral Capsule</td>
<td style="text-align: right;">19073187</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="even">
<td style="text-align: left;">amoxicillin 500 MG Oral Capsule</td>
<td style="text-align: right;">19073187</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">3</td>
<td style="text-align: left;">BETA-LACTAM ANTIBACTERIALS,
PENICILLINS</td>
<td style="text-align: left;">J01C</td>
<td style="text-align: right;">21602818</td>
</tr>
<tr class="odd">
<td style="text-align: left;">amoxicillin 500 MG Oral Capsule</td>
<td style="text-align: right;">19073187</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">5</td>
<td style="text-align: left;">amoxicillin; systemic</td>
<td style="text-align: left;">J01CA04</td>
<td style="text-align: right;">21602823</td>
</tr>
<tr class="even">
<td style="text-align: left;">amoxicillin 500 MG Oral Capsule</td>
<td style="text-align: right;">19073187</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">1</td>
<td style="text-align: left;">ANTIINFECTIVES FOR SYSTEMIC USE</td>
<td style="text-align: left;">J</td>
<td style="text-align: right;">21602795</td>
</tr>
<tr class="odd">
<td style="text-align: left;">amoxicillin 500 MG Oral Capsule</td>
<td style="text-align: right;">19073187</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">4</td>
<td style="text-align: left;">Penicillins with extended spectrum</td>
<td style="text-align: left;">J01CA</td>
<td style="text-align: right;">21602819</td>
</tr>
</tbody>
</table>

Alternatively we can filter all drugs that appear in a particular ATC
level and class (in this case “ANTIBACTERIALS FOR SYSTEMIC USE” from ATC
level 2).

    antibacterials <- drug_lookup |> 
      filter(ATC_concept_name == "ANTIBACTERIALS FOR SYSTEMIC USE") 

    nrow(antibacterials)

    ## [1] 69

    #display top 10
    antibacterials |> 
      filter(row_number()<11) |>  kable()

<table>
<colgroup>
<col style="width: 37%" />
<col style="width: 9%" />
<col style="width: 13%" />
<col style="width: 6%" />
<col style="width: 19%" />
<col style="width: 5%" />
<col style="width: 9%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">drug_concept_name</th>
<th style="text-align: right;">drug_concept_id</th>
<th style="text-align: left;">drug_concept_class_id</th>
<th style="text-align: left;">ATC_level</th>
<th style="text-align: left;">ATC_concept_name</th>
<th style="text-align: left;">ATC_code</th>
<th style="text-align: right;">ATC_concept_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">amoxicillin 50 MG/ML Oral Suspension</td>
<td style="text-align: right;">1713370</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="even">
<td style="text-align: left;">azithromycin 500 MG Oral Tablet</td>
<td style="text-align: right;">1734134</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="odd">
<td style="text-align: left;">levofloxacin 250 MG Oral Tablet</td>
<td style="text-align: right;">1742254</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="even">
<td style="text-align: left;">erythromycin 50 MG/ML Oral Suspension</td>
<td style="text-align: right;">1747354</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="odd">
<td style="text-align: left;">tetracycline hydrochloride 250 MG Oral
Tablet</td>
<td style="text-align: right;">1836973</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="even">
<td style="text-align: left;">amoxicillin 250 MG Oral Capsule</td>
<td style="text-align: right;">19073183</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="odd">
<td style="text-align: left;">cephalexin 250 MG Oral Capsule</td>
<td style="text-align: right;">19075032</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="even">
<td style="text-align: left;">ciprofloxacin 50 MG/ML Oral
Suspension</td>
<td style="text-align: right;">19075379</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="odd">
<td style="text-align: left;">amoxicillin 80 MG/ML / clavulanate 11.4
MG/ML Oral Suspension</td>
<td style="text-align: right;">19123605</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
<tr class="even">
<td style="text-align: left;">Colistin 1000000 UNT Injectable
Solution</td>
<td style="text-align: right;">36883959</td>
<td style="text-align: left;">Clinical Drug</td>
<td style="text-align: left;">2</td>
<td style="text-align: left;">ANTIBACTERIALS FOR SYSTEMIC USE</td>
<td style="text-align: left;">J01</td>
<td style="text-align: right;">21602796</td>
</tr>
</tbody>
</table>

TO BE CONTINUED … DRAFT MATERIAL BELOW TODO

~ look at drug\_source\_concept\_name for drugs that have a drug\_concept\_id of 0  
(add to other docs about what \*concept\_id of 0 means)

is this true : Note that the `dm+d` codes are more granular than
`RxNorm` so the mapping can go from 1 to many.

add link to drug\_exposure table description :
<https://ohdsi.github.io/CommonDataModel/cdm54.html#drug_exposure>

check issue in omopcept when concept\_name not included in join columns

This is where uclh drugs are mapped in omop\_es
<https://github.com/uclh-criu/omop_es/blob/3c36642b3a24944da7feb0fd214088241cf1990b/mapping/UCLH/epic_common/medication_component_common.R>

Comments there :

Maps from EPIC MedicationKey to OMOP concept\_id and, if applicable, a
standard drug\_concept\_id via the following stages:

EPIC MedKey -&gt; dm+d -&gt; dm+d concept (OMOP Non-Std) -&gt; RxNORM
**Ext** OMOP (OMOP Std)

Note that this is a 1-M-M mapping process

For this reason a score is assigned that ranks full over partial mapping
and devices over drugs

The dm+d concept\_id (omop?) should be in drug\_source\_concept\_id

also route\_concept\_id & route\_source\_concept\_id should be useful
