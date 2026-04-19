# Sales Data Pipeline: Azure Data Factory (ADF) Data Flow

## Project Overview
This project implements a robust ETL (Extract, Transform, Load) pipeline using Azure Data Factory to clean and standardize "dirty" sales records. The pipeline handles common data quality issues such as mixed date formats, inconsistent casing, negative values in financial columns, and missing identifiers.

---

## 1. Environment Setup
The pipeline is hosted within an Azure Data Factory instance. We utilized a **Mapping Data Flow** with a debug cluster to allow for real-time validation of our transformation logic.

<img width="1917" height="927" alt="Screenshot 2026-04-19 175642" src="https://github.com/user-attachments/assets/31d544c6-3c6f-40f7-8498-4f6a68dca6aa" />


---

## 2. Source Configuration & Schema Fixes
The raw data is ingested from a CSV file. One critical step was manually overriding the **Source Projection** to treat all incoming columns as `string`. This prevents ADF from incorrectly guessing data types (Type Mismatch) before the cleaning logic can run.

* **Source Format:** Delimited Text (CSV)
* **Key Column Renames:** `O_ID` to `order_id`, `O_Date` to `order_date`, etc.

---

## 3. Transformation Logic (The "Code")
The heart of the pipeline is a **Derived Column** transformation. We used the ADF Expression Language to implement the following logic:

### Date Standardization
Handles mixed formats like `YYYY-MM-DD` and `DD/MM/YYYY` into a unified `Date` type.
```text
coalesce(toDate(toString(order_date), 'yyyy-MM-dd'), toDate(toString(order_date), 'dd/MM/yyyy'))
```

### Financial Data Correction
Uses absolute values to correct erroneous negative signs in `quantity` and `unit_price`, ensuring `total_amount` is calculated accurately.
```text
abs(toDecimal(unit_price, 10, 2))
```

### Text Normalization
Cleans `customer_id` and fills missing `category` values with a "General" tag.
```text
iif(isNull(category) || category == 'Unknown', 'General', toString(category))
```

<img width="1919" height="932" alt="Screenshot 2026-04-19 175705" src="https://github.com/user-attachments/assets/a5f5ac60-d253-493d-9539-19941e5bbee5" />


---

## 4. Data Validation (Filter)
To ensure downstream systems only receive high-quality data, a **Filter Transformation** was added to drop any records where critical fields (like `order_date` or `order_id`) remained null after processing.

<img width="1919" height="929" alt="Screenshot 2026-04-19 175722" src="https://github.com/user-attachments/assets/c0d5055f-0ccf-4f23-a9cd-b52dd7a1c208" />


---

## 5. Final Results & Data Preview
By utilizing the **Data Preview** tab in ADF, we confirmed that:
- Mixed dates are now properly formatted.
- Negative prices are now positive.
- "Unknown" categories are standardized.

<img width="1919" height="927" alt="Screenshot 2026-04-19 175737" src="https://github.com/user-attachments/assets/eb020844-3f94-40eb-9d46-2ea1507e45e1" />


---

## How to Run
1.  Open **Azure Data Factory Studio**.
2.  Navigate to the **Author** tab and select the `SalesDataFlow` pipeline.
3.  Click **Trigger Now** to process the latest `sales.csv` file.
4.  Monitor the status via the **Monitor** tab.

---

### Technical Stack
* **Cloud Provider:** Microsoft Azure
* **Service:** Azure Data Factory (V2)
* **Feature:** Mapping Data Flows
* **Language:** ADF Expression Language (Expression Builder)
