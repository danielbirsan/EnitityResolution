#  Entity Resolution for Company Records — Veridion Internship Challenge

The project was completed as part of the **Veridion Data Engineer Internship challenge**, and showcases a pipeline designed to be:

> **Table of Contents**  
> - [Overview](#overview)  
> - [Key Features](#key-features)  
> - [Installation & Requirements](#installation--requirements)  
> - [Results](#results)  
> - [Next Steps](#next-steps)  

---

## Overview

This repository demonstrates a comprehensive **Entity Resolution** pipeline for **company records**. Starting with a **noisy dataset** of **33,000+ rows**, the project aims to:

1. **Clean & normalize** raw company data.  
2. **Compute similarities** among potentially matching records via a **points-based** or **weighted** approach.  
3. **Cluster** duplicate records (connected components) and **merge** them into single “golden records.”

The final output is saved to a **Parquet file** and provides a **deduplicated** set of unique companies.

---

## Key Features

- **Preprocessing & Normalization**  
  - Lowercases text columns, strips whitespace, standardizes phone, email, location fields, etc.
- **Offline & Fast**  
  - Uses **blocking** on `main_country` to reduce comparisons.  
  - Employs **RapidFuzz** for fuzzy name matching.  
  - No external API calls (e.g., no geocoding services), ensuring speed and privacy.
- **Points-Based Similarity**  
  - Weighted scoring for:
    - **Company Name** (fuzzy, up to 2 points)  
    - **Revenue** ±10% (1 point)  
    - **Employee Count** ±10% (1 point)  
    - **Website Domain** exact (2 points)  
    - **Social Media** (0.5 points each match), etc.
    - **Country/City/Region different** (-2 points each), etc.
- **Connected Components**  
  - Pairwise “duplicates” are converted into graph edges.  
  - Each **connected component** merges into a final “golden record.”
- **Final Output**  
  - Writes to **`final_data.parquet`** or a user-specified name, drastically reducing the original row count.


## Installation & Requirements

1. **Python 3+**  
2. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```
   or individually:
   ```bash
   pip install pandas numpy pycountry tldextract spacy rapidfuzz networkx pyarrow
   ```
3. **spaCy Model** (if using text-based NER):
   ```bash
   python -m spacy download en_core_web_sm
   ```

## Postal Code Lookup

To enrich missing `main_city`, `main_country`, and `main_country_code` values without using online geocoding services, we use a **pre-downloaded postal code dataset**.

 Download the dataset from this Google Drive folder:

 [📁 allCountriesCSV (Google Drive)](https://drive.google.com/drive/folders/1mN47iWtoVVqBUNuUiFeq7UQ65yAv-fps)

It includes columns like:

- `POSTAL_CODE`
- `CITY`
- `COUNTRY` (ISO Alpha-2)
- `LATITUDE`, `LONGITUDE`, etc.

```python
postal_df = pd.read_csv("allCountriesCSV.csv", dtype=str)
postcode_to_city = dict(zip(postal_df["POSTAL_CODE"], postal_df["CITY"]))
postcode_to_country = dict(zip(postal_df["POSTAL_CODE"], postal_df["COUNTRY"]))
```

Used this mapping offline to:
- Fill in missing cities/countries
- Avoid any network/API calls
- Enhance both preprocessing and similarity scoring

---

## Results

- From **33,000+** initial rows, the pipeline merges duplicates into a much smaller set of **unique “golden records.”** (19779 connected components (clusters) )
- The final dataset is stored in **`final_data.parquet`**.  
- You can print or visualize any multi-record cluster before merging to verify correctness.

---

## Next Steps

1. **Tune Threshold & Points**  
   - Adjust the points for each feature (e.g., 1 vs. 2 points for domain match).  
   - Evaluate how it impacts precision vs. recall.

2. **Improve Scalability**  
   - For large data, rely on Dask or Spark for parallel blocking & pairwise comparisons.

3. **Advanced Matching**  
   - Incorporate advanced **token-based** or **embedding-based** name matching.  
   - Explore **locality-sensitive hashing (LSH)** if you have extremely large sets.
4. **Enriching data**  
   - Use LLM to extract data from the descriptions
   - Value more the activity columns and the location columns


