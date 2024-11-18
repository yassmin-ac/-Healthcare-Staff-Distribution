# Healthcare Staff Distribution

## Table of Contents

## Project Overview

This study reviews the distribution of U.S. healthcare staffing in Q1 2024, benchmarking work hours by role to generate insights and identify regional sales opportunities for a company that connects healthcare contractors with nursing homes in need of on-demand staff.

The following sections outline the steps taken during the analysis, including SQL queries and Tableau visualizations. You can also explore the findings interactively through the dashboard linked [here](https://public.tableau.com/app/profile/yassmin.ac/viz/TakeHomeTestClipboardHealth/CNADash).

![GIF - Healthcare Staff](https://github.com/user-attachments/assets/26098c0a-10a2-470c-bc0b-c9ba3fd9f569)

## Data Sources

This project uses data provided by the [Centers for Medicare & Medicaid Services (CMS)](https://data.cms.gov/), a federal agency within the United States Department of Health and Human Services (HHS) that inspects and reports on nursing homes nationwide.

The dataset explored is named ['Payroll-Based Journal (PBJ) Daily Nurse Staffing.csv'](https://data.cms.gov/quality-of-care/payroll-based-journal-daily-nurse-staffing/data) and contains information on registered healthcare providers.

Over one million rows of data were reviewed, focusing on specific data points such as:

- Provider name (nursing home facility name)
- City
- State
- Work date
- Worked hours by professional category, which includes the following categories:
  - RN (Registered Nurses)
  - RNDON (Director of Nursing)
  - RNadmin (RN with Administrative Duties)
  - LPN (Licensed Practical Nursing)
  - LPNadmin (LPN with Administrative Duties)
  - CNA (Certified Nursing Assistant)
  - NAtrn (Nurse Aide in Training)
  - MedAide (Certified Medication Aide)​

Each professional category is classified into two groups: those directly employed by nursing facilities (employees) and those hired on demand (contractors). The dataset includes the total hours for each of these employment types.

## Tools

- MS Excel - Data Cleaning
- SQLite / DBeaver - Data Analysis
- Tableau - Creating Data Visualization and Reports

## Data Cleaning and Preparation

### Getting Familiar with the Dataset

- I created an Excel spreadsheet that catalogs each column name with sample values, highlighting key information I found potentially useful for the analysis.
- I researched each professional category abbreviation through the dataset’s dictionary to better understand each role.
- I explored the responsibilities and hierarchy of each category, looking for patterns and correlations in staffing numbers to deepen my insights.

### Preparing the Data

- I uploaded the PBJ dataset into Tableau, adjusted the data types, and organized Tableau’s data browser into folders and hierarchies for better accessibility.
- I created a separate dataset that includes state names and a country column to enable the map functionality in Tableau. After uploading it to Tableau, I defined its primary key to establish a correlation with the PBJ dataset.
- I wrote the following script to log the PBJ dataset into SQL:

```sql
CREATE TABLE PBJ_Daily_Nurse_Staffing_Q1_2024 (
   PROVNUM INTEGER,
   PROVNAME TEXT,
   STATE TEXT,
   COUNTY_NAME TEXT,
   COUNTY_FIPS INTEGER,
   CY_Qtr TEXT,
   WorkDate TEXT,
   MDScensus INTEGER,
   Hrs_RNDON INTEGER,
   Hrs_RNDON_emp INTEGER,
   Hrs_RNDON_ctr INTEGER,
   Hrs_RNadmin INTEGER,
   Hrs_RNadmin_emp INTEGER,
   Hrs_RNadmin_ctr INTEGER,
   Hrs_RN INTEGER,
   Hrs_RN_emp INTEGER,
   Hrs_RN_ctr INTEGER,
   Hrs_LPNadmin INTEGER,
   Hrs_LPNadmin_emp INTEGER,
   Hrs_LPNadmin_ctr INTEGER,
   Hrs_LPN INTEGER,
   Hrs_LPN_emp INTEGER,
   Hrs_LPN_ctr INTEGER,
   Hrs_CNA INTEGER,
   Hrs_CNA_emp INTEGER,
   Hrs_CNA_ctr INTEGER,
   Hrs_NAtrn INTEGER,
   Hrs_NAtrn_emp INTEGER,
   Hrs_NAtrn_ctr INTEGER,
   Hrs_MedAide INTEGER,
   Hrs_MedAide_emp INTEGER,
   Hrs_MedAide_ctr INTEGER
);
```

## Exploratory Data Analysis EDA

### Defining Marketing Priorities to Achieve Business Goals​

This study was developed to help a company determine where to focus its marketing campaign efforts. The company generates revenue by selling its platform to nursing facilities, which are its primary clients. While targeting these facilities is essential, a strategic approach should also consider healthcare professionals who use the platform to connect with these facilities. These users are crucial, as they help spread awareness and enhance the platform's relevance.

### Assessing the Impact of Professional Categories Nationwide

The first step in the exploratory data analysis was to rank professional categories across the United States by volume. Registered work hours by professional category were used to estimate the number of professionals in each group. The analysis of the query results below revealed that the CNA, LPN, and RN categories collectively account for 89% of the total hours across all categories.

```sql
SELECT 
   CEIL(SUM(Hrs_CNA)) total_CNA
   ,CEIL(SUM(Hrs_LPN)) total_LPN
   ,CEIL(SUM(Hrs_RN)) total_RN
   ,CEIL(SUM(Hrs_RNadmin)) total_RNadmin
   ,CEIL(SUM(Hrs_MedAide)) total_MedAide
   ,CEIL(SUM(Hrs_LPNadmin)) total_LPNadmin
   ,CEIL(SUM(Hrs_RNDON)) AS total_RNDON
   ,CEIL(SUM(Hrs_NAtrn)) total_NAtrn
FROM PBJ_Daily_Nurse_Staffing_Q1_2024;
```

![1](https://github.com/user-attachments/assets/bb8f97e9-2a51-4443-aa34-d8f7d72ed0b7)

![1](https://github.com/user-attachments/assets/ed8ac124-ad92-45eb-8777-73a32f6468b6)

Nursing homes typically employ a combination of full-time staff and contractors, with varying ratios across facilities. As healthcare professionals utilize the platform to find contract work in nursing homes that require on-demand staffing, they should be prioritized as key marketing targets. By calculating the ratio of contractors to total working hours for each category—considering both contractors and employees within each professional category—the following results were obtained:

```sql
SELECT 
   CEIL((SUM(Hrs_CNA_ctr) * 100.0) / (SUM(Hrs_CNA_emp) + SUM(Hrs_CNA_ctr))) || '%' AS pct_Hrs_CNA_ctr
   ,CEIL((SUM(Hrs_LPN_ctr) * 100.0) / (SUM(Hrs_LPN_emp) + SUM(Hrs_LPN_ctr))) || '%' AS pct_Hrs_LPN_ctr
   ,CEIL((SUM(Hrs_RN_ctr) * 100.0) / (SUM(Hrs_RN_emp) + SUM(Hrs_RN_ctr))) || '%' AS pct_Hrs_RN_ctr
   ,CEIL((SUM(Hrs_RNadmin_ctr) * 100.0) / (SUM(Hrs_RNadmin_emp) + SUM(Hrs_RNadmin_ctr))) || '%' AS pct_Hrs_RNadmin_ctr
   ,CEIL((SUM(Hrs_MedAide_ctr) * 100.0) / (SUM(Hrs_MedAide_emp) + SUM(Hrs_MedAide_ctr))) || '%' AS pct_Hrs_MedAide_ctr
   ,CEIL((SUM(Hrs_LPNadmin_ctr) * 100.0) / (SUM(Hrs_LPNadmin_emp) + SUM(Hrs_LPNadmin_ctr))) || '%' AS pct_Hrs_LPNadmin_ctr
   ,CEIL((SUM(Hrs_RNDON_ctr) * 100.0) / (SUM(Hrs_RNDON_emp) + SUM(Hrs_RNDON_ctr))) || '%' AS pct_Hrs_RNDON_ctr
   ,CEIL((SUM(Hrs_NAtrn_ctr) * 100.0) / (SUM(Hrs_NAtrn_emp) + SUM(Hrs_NAtrn_ctr))) || '%' AS pct_Hrs_NAtrn_ctr
FROM PBJ_Daily_Nurse_Staffing_Q1_2024;
```

![2](https://github.com/user-attachments/assets/5bc147d2-f2ab-49bb-a5a1-551b7741a0bc)

#### Insight

*The analysis of the total national work hours and the contractor proportions in each category suggests focusing marketing campaigns on the CNA, LPN, and RN groups.​*

### Top 3 Professional Categories: A Geographic Breakdown

Next, the study concentrated on the CNA, LPN, and RN categories, ranking the five states with the highest proportion of contractor work hours relative to total hours.

- CNA

```sql
SELECT 
   STATE
   ,CEIL((SUM(Hrs_CNA_ctr) * 100.0) / (SUM(Hrs_CNA_emp) + SUM(Hrs_CNA_ctr))) || '%' AS pct_Hrs_CNA_ctr
FROM
   PBJ_Daily_Nurse_Staffing_Q1_2024
GROUP BY
   STATE
ORDER BY 
   CEIL((SUM(Hrs_CNA_ctr) * 100.0) / (SUM(Hrs_CNA_emp) + SUM(Hrs_CNA_ctr))) DESC
   LIMIT 5;
```

![3](https://github.com/user-attachments/assets/32fcdd24-daec-474a-87cb-e4b492494541)

![2](https://github.com/user-attachments/assets/214954ea-f7f6-4a3d-ab42-2a3f30b9489e)

- LPN

```sql
SELECT 
   STATE
   ,CEIL((SUM(Hrs_LPN_ctr) * 100.0) / (SUM(Hrs_LPN_emp) + SUM(Hrs_LPN_ctr))) || '%' AS pct_Hrs_LPN_ctr
FROM
   PBJ_Daily_Nurse_Staffing_Q1_2024
GROUP BY
   STATE
ORDER BY 
   CEIL((SUM(Hrs_LPN_ctr) * 100.0) / (SUM(Hrs_LPN_emp) + SUM(Hrs_LPN_ctr))) DESC
   LIMIT 5;
```

![4](https://github.com/user-attachments/assets/de28ca6d-754b-4a1e-85a9-8c8a509abe66)

![3](https://github.com/user-attachments/assets/899bbac4-5334-47b1-ae39-3156b50406cc)

- RN

```sql
SELECT 
   STATE
   ,CEIL((SUM(Hrs_RN_ctr) * 100.0) / (SUM(Hrs_RN_emp) + SUM(Hrs_RN_ctr))) || '%' AS pct_Hrs_RN_ctr
FROM
   PBJ_Daily_Nurse_Staffing_Q1_2024
GROUP BY
   STATE
ORDER BY 
   CEIL((SUM(Hrs_RN_ctr) * 100.0) / (SUM(Hrs_RN_emp) + SUM(Hrs_RN_ctr))) DESC
   LIMIT 5;
```

![5](https://github.com/user-attachments/assets/149601b8-0be5-4ee7-bbad-606115b91000)

![4](https://github.com/user-attachments/assets/4232725c-c265-4370-906e-0f5c303e04cb)

The maps revealed a trend: most hours were registered in states on the country's eastern side. To further explore this pattern, a density map was created to highlight each city where hours were recorded, which shows that most of the cities with registered healthcare facilities are also located on the eastern side of the country.

![5](https://github.com/user-attachments/assets/83bac6eb-6b05-447e-af24-b3ccaa4408c8)

#### Insight

*Based on the data, it is recommended that a marketing campaign be launched on the West Coast, targeting the states ranked highest for each of the three selected professional categories.*


