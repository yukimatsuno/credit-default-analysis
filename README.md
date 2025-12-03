# Credit Default Analysis

This project explores and models credit default risk using the **Give Me Some Credit** dataset.  
It covers the whole workflow from **automated data ingestion** to **data cleaning, exploratory data analysis (EDA), and feature engineering**.

The repository is designed both as a **data science** study project and as a **data-engineering–oriented pipeline** example.

---

## Part 1 – Data Ingestion (Batch Pipeline)

### Goal

Automatically collect new CSV files on a schedule, append them to a master dataset, and upload the consolidated data into a **Cloud SQL (MySQL on GCP)** instance so that multiple data scientists can access the same source of truth.

### File locations

- Data directory: `data/`
- Raw input folder (watched by the cron job): `data/input_folder/`
- Consolidated dataset (created by the job): `data/sample_data.csv`
- Original snapshot of the dataset: `data/sample_data_original.csv`
- Cron scripts & notebooks: `notebooks/`
  - `cron-upload-to-gcp.py`
  - `cronjob.txt`
  - `basic visualization.ipynb`

### Proposed flow

1. A **cron job** runs every day at **2:00 AM** on a VM.
2. It checks for new CSV files in `../data/input_folder/`.
3. New incoming files are dropped into this directory by an upstream process.
4. The python script `cron-upload-to-gcp.py`:
   - Reads all CSV files in `../data/input_folder/`.
   - Appends the rows to the existing master dataframe.
   - Saves the updated dataframe as `sample_data.csv` in `../data/`.
   - Uploads the contents of `sample_data.csv` into a **Cloud SQL (MySQL)** table on GCP.
   - Deletes all files in `../data/input_folder/` (including the temporary `sample_data.csv`).
   - Recreates a clean `../data/input_folder/` directory for the next run.

### Cron setup

The crontab entry is documented in `notebooks/cronjob.txt`.

Steps to set it up:

1. Open Terminal on the VM.
2. Run:

   ```bash
   crontab -e

---

 ## Part 2 – Understanding the Data (EDA & Cleaning)

 All EDA and cleaning logic is implemented in:

 - `notebooks/basic visualization.ipynb`

 ### How to run locally

 1. Start Jupyter:

    ```bash
    jupyter notebook
    ```

 2. Open: `notebooks/basic visualization.ipynb`  
 3. Run the cells step by step.

 ### Main steps in the notebook

 1. **Import raw CSV**

    - Load the credit data from CSV files.
    - Drop technical columns such as `Unnamed: 0` that do not contain information.

 2. **Initial exploration**

    - Inspect columns, shape, data types, and non-null counts.
    - Build a simple data dictionary to understand each feature.
    - Check for duplicated rows.

 3. **Column name cleanup**

    - Rename columns containing symbols (e.g. `-`) to avoid issues when referring to them in code.

 4. **Missing values & outliers**

    The dataset includes missing values and “encoded outliers” such as 96 or 98.  
    The notebook applies column-specific cleaning rules:

    - `MonthlyIncome`  
      - Fill `NaN` with the median.  
      - Replace extreme outliers with the median.

    - `NumberOfDependents`  
      - Fill `NaN` with the median.  
      - Replace extreme outliers.

    - `Age`  
      - Replace ages below 22 with 22.

    - `RevolvingUtilizationOfUnsecuredLines`  
      - Cap values above 1 at 1.

    - `NumberOfTime30to59DaysPastDueNotWorse`  
      - Replace special values 96 and 98 with the median.

    - `DebtRatio`  
      - Cap values above 1 at 1.

    - `NumberOfOpenCreditLinesAndLoans`  
      - Cap values above 20 at 20.

    - `NumberOfTimes90DaysLate`  
      - Replace 96 and 98 with the median.

    - `NumberRealEstateLoansOrLines`  
      - Replace outliers with the median.

    - `NumberOfTime60to89DaysPastDueNotWorse`  
      - Replace 96 and 98 with the median.

 5. **Class imbalance**

    The target variable is:

    - `SeriousDlqin2yrs` — whether the person experienced a 90-days-past-due delinquency or worse within 2 years.

    The dataset is highly imbalanced:

    - ~93%: No delinquency (`0`)  
    - ~7%: Delinquency (`1`)

    The notebook explores rebalancing techniques and visualization.

 6. **Visualizations & correlations**

    - Correlation heatmaps
    - Feature distributions (Yes vs No)
    - Visualizations without outliers

 ### Key findings (EDA summary)

 - Higher revolving utilization is associated with a higher delinquency rate.  
 - Younger customers tend to show higher delinquency.  
 - If `NumberOfTimes90DaysLate > 1`, the chance of delinquency rises sharply.  
 - More real-estate–backed loans tends to correlate with lower delinquency.

 ---
