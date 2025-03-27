#  Entity Resolution for Company Records 


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
## 📄 Documentation

Full documentation is also available here:  
[https://2ly.link/25ovw](https://2ly.link/25ovw)


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



## What Makes a Company... a Company?

Is a McDonald's in Galați different from a McDonald's in Bucharest?  
What if a company splits into two entities, each responsible for different market segments or stages of the production pipeline — are those still the same company or two distinct ones?  
And what happens if a company changes its name, opens a new factory, and hires different employees — is it still the same company?

*BONUS*
These questions touch on the philosophical dilemma of identity, reminiscent of the **Ship of Theseus** paradox:  
> *If every component of a company is replaced over time, is it still the same entity?*


## ⚖️ Assumptions in This Project

In this solution, I aimed for a **balanced and practical approach**:

- If a company’s city appears in the list of known locations for another record, they may be treated as the same entity — especially if other fields (e.g., name, domain) align.
- Different locations or departments of a larger company can still be merged if the similarities are strong.
- However, if the differences across key fields are significant (e.g., industry, name, location), they are likely treated as distinct companies.

---

##  Flexibility of the System

The scoring and matching system is **fully customizable**:

- Tune how much weight is given to name, domain, location, activity, etc.
- Adjust thresholds to be stricter or more lenient
- Disable location comparison entirely if needed

This makes the solution highly **adaptable** — suitable for merging franchise records, cleaning up business databases, or defining company identity in different contexts.




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

To enrich missing `main_city`, `main_country`, and `main_country_code` values without using online geocoding services, I used a **pre-downloaded postal code dataset**.

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
- BONUS: I tried to deduplicate the data and the final dataset is stored in **`final_data.parquet`**.  
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
