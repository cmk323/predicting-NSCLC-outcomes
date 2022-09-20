# predicting-NSCLC-outcomes
Using simulated clinical and genomic data to build and evaluate a predictive model of one-year survival after diagnosis with NSCLC.

## Background
Lung cancer is the leading cause of cancer-related deaths worldwide. In 2018, the United States is estimated to have had approximately 234,030 new diagnoses of lung cancer, with an estimated 154,050 deaths. As lung cancer is a highly heterogeneous disease with variable survival outcomes, there has been wide interest in developing prognostic models that integrate varying sources of patient information (e.g. demographic, clinical, genomic, imaging) to predict patient survival following diagnosis. An eventual goal for these models is to deploy them as part of decision support systems to facilitate improved decision making in routine clinical practice.

## Data Cleansing & Preprocessing
Several steps were taken to clean the data and preprocess it into a format which can be "understood" by a machine learning model. During an intitial exploration of the clinical data, several categorical columns stood out as those which may potentially contain inconsistencies in data entry, such as misspellings. For example, the **Stage** column contained one row in which the grade "IB" was entered using a different convention, "1B". Furthermore, two misspellings of "Right Upper Lobe" as "Righ Upper Lobe" in the **Primary.Site** column resulted in two separate categories being created for the same category. Fixing these errors helped improve data quality. Similarly, a discrepancy was identified in the **Grade** column, for which 96 rows contained values of "9". However, the data description notes that grade should range in value from 1-4. Because 96 rows occupies over 50% of the 190 row dataset, it was determined that this entire column should be dropped as the meaning of these "9" values cannot be deduced without consulting with individuals involved in data collection and entry.

Another important step in data cleansing was handling missing data. A substantial number of rows were identified with missing data for either the **N**, **M**, or **Tumor.Size** columns. Because the TNM System and tumor size intuitively seem like important prognostic factors, it would be illogical to drop these columns. Instead, all rows were preserved via mean imputation, in which `SimpleImputer()` was used to estimate the missing values using the mean. Mean imputation was used in this scenario due to time constraints. However, model accuracy could potentially be improved by using the TNM System to predict missing values of **T**, **N**, and **M**. Another potential strategy for estimating values of missing data in the **N**, **M**, or **Tumor.Size** columns is the implementation of `KNNImputer()`, which completes missing values using k-Nearest Neighbors.

In regards to preprocessing the data prior to building the model, one important step was to convert all categorical (string) data to numerical data, whether in the form of integers or Boolean flags. In order to accomplish this, the categories in **Stage** and **T** were simplified from a letter-number (e.x. "1a") ranking system to a solely numerical system (e.x. "1"). Additionally, one hot encoding was implemented to convert the categorical data in the **Histology** column as well as the at-risk genes of interest in the genomic data into new categorical columns with Boolean flags. For example, the three histological categories were divided into three new columns and appended to the dataframe, with a Boolean flag marking whether or not each histological feature was observed in a given patient. Similarly, three at-risk genes identified during exploratory data analysis were converted into separate categorical columns, with Boolean flags marking whether or not each patient was found via tumor sequencing to possess the gene variant of interest.

## Exploratory Data Analysis
Through exploratory data analysis of both the clinical and genomic data, a total of 16 features were selected to be used for the predictive model due to their correlation with survival time and cancer outcome, thus making them important prognostic factors.

The selected features are as follows:
- **Survival Months:** the followup time in months
- **Age:** the patient's age (in years) at diagnosis
- **Number of Primary Tumors**
- **T:** the size and extent of the primary tumor
- **N:** the number of nearby lymph nodes to which the cancer has spread
- **M:** Whether or not the cancer has metastasized, or spread from the primary tumor to other parts of the body
- **Radiation:** whether or not the patient has been exposed to radiation treatment protocols
- **Stage:** stage at diagnosis
- **Tumor Size:** the size of the primary tumor
- **Both Lungs:** whether the cancer is present in only one or both of the patient's lungs
- **Adenocarcinoma:** whether or not the patient possesses histological features of adenocarcinoma
- **Large-cell carcinoma:** whether or not the patient possesses histological features of Large-cell carcinoma
- **Squamous cell carcinoma:** whether or not the patient possesses histological features of Squamous cell carcinoma
- **TP53_Col1:** whether or not the patient possesses this TP53 gene variant 
- **KRAS_Col1:** whether or not the patient possesses this KRAS gene variant
- **CDKN2A:** whether or not the patient possesses this CDKN2A gene variant

In regards to the gene variants of interest, genomic analysis revealed that mutations in TP53, KRAS, and CDKN2A genes are not only the most prevalent gene mutations in the genomic dataset but are also correlated with poor prognosis. The prognostic influence of these gene variants was determined via examining the survival rate of patients with these gene variants, as those patients with mutations in TP53, KRAS, and CDKN2A were found to have survival rates of 26%, 24%, and 6.7%, respectively. The cancer-causing effects of mutations in these specific genes were confirmed via a quick literature search on the topic, thus validating their use as categorical features in the predictive model.

In regards to feature engineering, the **Both Lungs** feature was engineered from the **Primary.Site** data due to the fact that there was no obvious correlation observed between primary site and cancer outcome with the exception of patients with cancer in both lungs. While the sample size for this patient population was quite small at only 5 rows, it logically follows that patients with cancer in both lungs likely have substantially poorer prognosis than those with cancer only in one portion of one lung. Thus, this feature was engineered to make more effective use of the data in the column. Because most cancers are not caused by a single mutated gene but rather a combination of gene variants, model accuracy could potentially be improved by engineering gene network features. Such gene network features were not investigated due to time constraints.
