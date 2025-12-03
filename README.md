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
