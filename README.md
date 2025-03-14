# Healthcare Staff Distribution

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis EDA](#exploratory-data-analysis-eda)
- [Insights and Recommendations](#insights-and-recommendations)
- [Data Visualization](#data-visualization)

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

#### Comment

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

#### Comment

*Based on the data, it is recommended that a marketing campaign be launched on the West Coast, targeting the states ranked highest for each of the three selected professional categories.*

### Identifying the Most Relevant Facilities

In the final step, facilities with the highest proportion of contractor hours relative to total hours worked were identified. The top 3 categories and their 5 top states were considered.

- CNA

```sql
SELECT
   PROVNAME 
   ,STATE 
   ,CITY
   ,CEIL((SUM(Hrs_CNA_ctr) * 100.0) / (SUM(Hrs_CNA_emp) + SUM(Hrs_CNA_ctr))) || '%' AS pct_Hrs_CNA_ctr
FROM
   PBJ_Daily_Nurse_Staffing_Q1_2024
WHERE
   STATE IN ('ME', 'NH', 'VT', 'NJ', 'PA')
GROUP BY
   PROVNAME 
   ,STATE 
   ,CITY
HAVING
   CEIL((SUM(Hrs_CNA_ctr) * 100.0) / (SUM(Hrs_CNA_emp) + SUM(Hrs_CNA_ctr))) >= 60
ORDER BY pct_Hrs_CNA_ctr DESC
LIMIT 10;
```

![6](https://github.com/user-attachments/assets/a1f88fbb-aaeb-4b5e-91ab-6fcb4eeeefd1)

- LPN

```sql
SELECT
   PROVNAME 
   ,STATE 
   ,CITY
   ,CEIL((SUM(Hrs_LPN_ctr) * 100.0) / (SUM(Hrs_LPN_emp) + SUM(Hrs_LPN_ctr))) || '%' AS pct_Hrs_LPN_ctr
FROM
   PBJ_Daily_Nurse_Staffing_Q1_2024
WHERE
   STATE IN ('ME', 'NH', 'VT', 'PA', 'HI')
GROUP BY
   PROVNAME 
   ,STATE 
   ,CITY
HAVING
   CEIL((SUM(Hrs_LPN_ctr) * 100.0) / (SUM(Hrs_LPN_emp) + SUM(Hrs_LPN_ctr))) >= 60
ORDER BY pct_Hrs_LPN_ctr DESC
LIMIT 10;
```

![7](https://github.com/user-attachments/assets/bc52541d-8dd4-4c90-971e-f73bd6a738ed)

- RN

```sql
SELECT
   PROVNAME 
   ,STATE 
   ,CITY
   ,CEIL((SUM(Hrs_RN_ctr) * 100.0) / (SUM(Hrs_RN_emp) + SUM(Hrs_RN_ctr))) || '%' AS pct_Hrs_RN_ctr
FROM
   PBJ_Daily_Nurse_Staffing_Q1_2024
WHERE
   STATE IN ('NY', 'MA', 'OR', 'DE', 'ME')
GROUP BY
   PROVNAME 
   ,STATE 
   ,CITY
HAVING
   CEIL((SUM(Hrs_RN_ctr) * 100.0) / (SUM(Hrs_RN_emp) + SUM(Hrs_RN_ctr))) >= 60
ORDER BY pct_Hrs_RN_ctr DESC
LIMIT 10;
```

![8](https://github.com/user-attachments/assets/204caaa3-2e77-482c-aff3-d999843f0a95)

## Insights and Recommendations

The company generates revenue by selling an on-demand platform that connects healthcare professionals with nursing facilities, its primary clients. Analyzing total national work hours and contractor proportions highlights the CNA, LPN, and RN groups as the leaders in total hours worked. Maps ranking the top five states by contractor work hour proportions reveal a trend: most hours are concentrated in eastern states. A density map of recorded working hours shows that most cities with healthcare facilities are also concentrated in the east of the U.S. Facilities in the top states with the highest contractor hour proportions were identified, focusing on the top three categories.

#### Key insights:

​- Focus efforts on attracting professionals in the CNA, LPN, and RN categories to use the platform.
- Launch a marketing campaign targeting the eastern side of the country.
- Approach the following thirty facilities to sell platform services:

![9](https://github.com/user-attachments/assets/59fa2c64-c6c4-4abe-9954-01373f9034c1)

## Data Visualization

To present the findings, I created an interactive Tableau dashboard allowing you to select and explore the specific information you need easily. Click [here](https://public.tableau.com/app/profile/yassmin.ac/viz/TakeHomeTestClipboardHealth/CNADash) to interact with the dashboard by following the instructions below:

- Section 1:

The first section displays the total worked hours for each professional category, highlighting a discrepancy in the hours worked by the CNA, LPN, and RN categories compared to others.

![Demonstration 1](https://github.com/user-attachments/assets/bfc9e453-9482-4764-89b3-96c9df3b76c6)

- Section 2:

The second section presents a density map that highlights each city where working hours were recorded.

![Demonstration 2](https://github.com/user-attachments/assets/b4da525f-014c-4e32-8645-a90d99e400e8)

- Section 3:

The third section displays data for each professional category, with interactive tabs that allow you to switch between categories. Each tab includes two visualizations:

1. A map highlighting the five states with the highest proportion of contractor hours relative to total hours in the selected category. Hovering over each state reveals a tooltip showing the contractor percentage.

2. A table ranking the top ten facilities with the highest contractor hours relative to total hours worked, all located within the top five states for the selected category.

![Demonstration 3](https://github.com/user-attachments/assets/f92cadca-f481-4736-85d9-49c8c9039d32)


