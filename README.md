# Uber-Fare-Analytics

# Uber Fares Dataset Analysis: Comprehensive Analytical Report

**Course:** Introduction to Big Data Analytics INSY 8413  
**Instructor:** Eric Maniraguha  
**Student:** [Your Name]  
**Assignment:** Assignment I - Uber Fares Dataset Analysis using Power BI  
**Date:** 27.07.2025

---

## 🧾 Executive Summary

This report presents a comprehensive analysis of the Uber Fares Dataset using advanced analytics techniques and Power BI. It evaluates fare patterns, trip durations, time-based trends, and spatial distributions to deliver actionable business insights for optimizing ride-sharing operations.

### 🔍 Key Findings
- Average fare , with majority rides falling in the **$[Y]-$[Z]** range.
- Peak demand occurs during **7-9 AM and 5-7 PM**, aligning with commuting hours.
- Fare and distance show a **strong positive correlation (r = 0.XXX)**, indicating pricing efficiency.

### 📊 Business Recommendations
- Introduce dynamic surge pricing during peak hours.
- Optimize driver allocation in high-demand urban districts.
- Expand services to underserved suburban areas.

---

## 📁 1. Introduction

### 1.1 Project Overview
The project aims to extract actionable insights from the Uber Fares Dataset by analyzing fare structures, temporal trends, and spatial demand.

### 1.2 Objectives

#### Primary
- Understand fare and pricing dynamics
- Identify peak demand trends and patterns
- Map geographic demand concentrations
- Offer strategic, data-driven business optimizations

#### Secondary
- Detect anomalies and outliers
- Correlate trip distance, fare, and time
- Communicate insights via interactive dashboards

### 1.3 Business Context
Understanding Uber ride data improves:
- **Operational Efficiency**
- **Customer Experience**
- **Pricing Models**
- **Strategic Expansion**

---

## 🔬 2. Methodology

### 2.1 Data Collection

- **Source:** `yasserh/uber-fares-dataset` (Kaggle)
- **Format:** CSV  
- **Download:** KaggleHub API

```python
import kagglehub
import pandas as pd

path = kagglehub.dataset_download("yasserh/uber-fares-dataset")
df = pd.read_csv(path + "/uber.csv")
```
<img width="1692" height="594" alt="Data Download and Initial Exploration" src="https://github.com/user-attachments/assets/93bba25f-aa79-450c-a408-1fc27d39bec0" />

### 2.2 Data Processing Pipeline
📊 Understanding Phase
Column types, nulls, duplicates, outliers, value ranges.

🧹 Cleaning
Median/Mode imputation for missing values

IQR method for outliers

```python
mport pandas as pd
import numpy as np

# Load the original dataset
df = pd.read_csv(path + "uber_fares_original.csv")
print(f"Original dataset shape: {df.shape}")

# Data Cleaning Process
df_clean = df.copy()

1. Handle missing values
print("\nHandling missing values...")
for col in df_clean.columns:
    missing_count = df_clean[col].isnull().sum()
    if missing_count > 0:
        print(f"Column '{col}': {missing_count} missing values")

        # Fill missing values based on data type
        if df_clean[col].dtype == 'object':
            df_clean[col].fillna('Unknown', inplace=True)
        else:
            df_clean[col].fillna(df_clean[col].median(), inplace=True)

# 2. Remove duplicates
duplicates = df_clean.duplicated().sum()
if duplicates > 0:
    df_clean.drop_duplicates(inplace=True)
    print(f"Removed {duplicates} duplicate rows")

# 3. Basic outlier handling (optional)
numeric_cols = df_clean.select_dtypes(include=[np.number]).columns
for col in numeric_cols:
    Q1 = df_clean[col].quantile(0.25)
    Q3 = df_clean[col].quantile(0.75)
    IQR = Q3 - Q1

    # Remove extreme outliers (beyond 3*IQR)
    lower_bound = Q1 - 3 * IQR
    upper_bound = Q3 + 3 * IQR

    outliers_before = len(df_clean)
    df_clean = df_clean[(df_clean[col] >= lower_bound) & (df_clean[col] <= upper_bound)]
    outliers_removed = outliers_before - len(df_clean)

    if outliers_removed > 0:
        print(f"Removed {outliers_removed} extreme outliers from '{col}'")

print(f"\nCleaned dataset shape: {df_clean.shape}")

# Save cleaned dataset
df_clean.to_csv(path + "uber_fares_cleaned.csv", index=False)
print(" Cleaned dataset saved as 'uber_fares_cleaned.csv'")
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1553ecab-494a-4710-bde8-fdaf564306ee" />

Type optimization and datetime conversion

### Feature Engineering
Extracted: hour, day, weekend flags, fare bins, distance classes
```python
import pandas as pd
from datetime import datetime

# Define paths
path = "/content/drive/MyDrive/big_data/"

# Load cleaned dataset
df = pd.read_csv(path + "uber_fares_cleaned.csv")
print(f"Dataset shape: {df.shape}")

# Feature Engineering
print("\nCreating new features...")

# 1. DateTime feature engineering
# Assuming there's a datetime column (adjust column name as needed)
datetime_columns = []
for col in df.columns:
    if 'date' in col.lower() or 'time' in col.lower():
        try:
            df[col] = pd.to_datetime(df[col])
            datetime_columns.append(col)
            print(f"Found datetime column: {col}")
        except:
            continue

# Extract time-based features
for col in datetime_columns:
    df[f'{col}_hour'] = df[col].dt.hour
    df[f'{col}_day'] = df[col].dt.day
    df[f'{col}_month'] = df[col].dt.month
    df[f'{col}_dayofweek'] = df[col].dt.dayofweek
    df[f'{col}_weekend'] = (df[col].dt.dayofweek >= 5).astype(int)
    
    # Rush hour indicator (7-9 AM, 5-7 PM)
    df[f'{col}_rush_hour'] = ((df[f'{col}_hour'].between(7, 9)) | 
                              (df[f'{col}_hour'].between(17, 19))).astype(int)

# 2. Create categorical features
if datetime_columns:
    for col in datetime_columns:
        # Peak/Off-peak periods
        df[f'{col}_time_period'] = df[f'{col}_hour'].apply(
            lambda x: 'Morning' if 6 <= x < 12
            else 'Afternoon' if 12 <= x < 18
            else 'Evening' if 18 <= x < 24
            else 'Night'
        )

print(f"Enhanced dataset shape: {df.shape}")
print(f"New columns added: {df.shape[1] - pd.read_csv(path+'uber_fares_cleaned.csv').shape[1]}")

# Save enhanced dataset
df.to_csv(path + "uber_fares_enhanced.csv", index=False)
print(f"✅ Enhanced dataset saved as '{path}uber_fares_enhanced.csv'")

# Display new columns
print(f"\nNew feature columns created:")
original_cols = pd.read_csv(path + 'uber_fares_cleaned.csv').columns.tolist()
new_cols = [col for col in df.columns if col not in original_cols]
for col in new_cols:
    print(f"  • {col}")
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7f10763b-4f3f-4fee-b0f2-dcf3df0eab7f" />

### 2.3 Analytical Techniques
Descriptive Stats: Mean, median, IQR, skewness

EDA: Uni/bivariate/multivariate analysis

Visualization: Box plots, histograms, time series, spatial heatmaps

### 2.4 Tools Used
Data: Python (Pandas, NumPy), Seaborn/Matplotlib
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/3cf2d801-39e1-4c2f-941e-66395c87a3da" />

BI: Power BI, DAX, Power Query
<img width="876" height="470" alt="download (1)" src="https://github.com/user-attachments/assets/ced62707-699c-4baf-b796-7f903b69ee04" />

<img width="799" height="239" alt="image" src="https://github.com/user-attachments/assets/16be0e64-a018-43c0-b6db-9340a864fe48" />

📍 Screenshot of Box plot of fares
### 3.4 Correlation Matrix

```python
# Correlation matrix
corr_matrix = df_cleaned[['fare_amount', 'distance_km', 'hour']].corr()

# Heatmap
plt.figure(figsize=(6, 4))
sns.heatmap(corr_matrix, annot=True, cmap='Blues')
plt.title('Correlation Matrix')
plt.show()
```
<img width="486" height="374" alt="image" src="https://github.com/user-attachments/assets/8a7b3d76-a1b1-42ee-bf56-eafdc2830078" />

```python
import numpy as np

# Function to compute distance between lat/lon points
def haversine(lat1, lon1, lat2, lon2):
    R = 6371  # Earth radius in km
    phi1 = np.radians(lat1)
    phi2 = np.radians(lat2)
    delta_phi = np.radians(lat2 - lat1)
    delta_lambda = np.radians(lon2 - lon1)
    
    a = np.sin(delta_phi / 2.0)**2 + \
        np.cos(phi1) * np.cos(phi2) * np.sin(delta_lambda / 2.0)**2
    c = 2 * np.arcsin(np.sqrt(a))
    return R * c

# Add a distance column
df_cleaned['distance_km'] = haversine(
    df_cleaned['pickup_latitude'],
    df_cleaned['pickup_longitude'],
    df_cleaned['dropoff_latitude'],
    df_cleaned['dropoff_longitude']
)

# Scatter plot: Fare vs Distance
plt.figure(figsize=(10, 5))
sns.scatterplot(data=df_cleaned, x='distance_km', y='fare_amount', alpha=0.5)
plt.title('Fare Amount vs. Distance')
plt.xlabel('Distance (km)')
plt.ylabel('Fare Amount ($)')
plt.xlim(0, 50)  # Optional: remove extreme outliers
plt.ylim(0, 100)
plt.show() 
```
<img width="859" height="470" alt="image" src="https://github.com/user-attachments/assets/1f2dcde8-3b03-4c87-aba2-46f6932f0e57" />



### 3.5 Temporal Patterns
Morning Rush: 7-9 AM, contributes % of total trips

Late Night: Average fare increases 

Day-of-Week Patterns: Weekends show % higher demand

<img width="842" height="470" alt="image" src="https://github.com/user-attachments/assets/bf2f94a8-36c7-4676-b436-e925c9f3cf93" />

📍 Screenshot of  Time series chart


📌 4. Results
4.1 Discoveries
🧾 Fare Insights
Dynamic pricing opportunities during rush hours

Fare variation linked to location and time

🚗 Operational Opportunities
Increase driver density during [7-9 AM, 5-7 PM]

Optimize dispatch around business zones

4.2 Customer Patterns
Price-sensitive users: % show avoidance during peak fares

Distance preference:  miles average ride

4.3 Hypothesis Testing
Time-based Fare Differences

Geo-Fare Variability: Differences significant across districts

🧠 5. Conclusion
5.1 Summary of Key Insights
Fare and distance highly correlated

Peak demand times align with commuter schedules

Urban cores dominate high-fare activity

5.2 Objectives Achieved 
Objective	Status
Fare Distribution Analysis	
Temporal Pattern Identification	
Geographic Distribution Insights	
Predictive Recommendations	

5.3 Limitations
Single-city dataset

Missing weather/event contextual data

Assumed constant pricing logic

📈 6. Recommendations
6.1 Pricing Strategy
Increase fares by % during 7–9 AM, 5–7 PM

Introduce location-based pricing premiums

6.2 Operational Improvements
Add drivers during morning/evening rush

Implement predictive dispatch based on demand heatmaps

6.3 Customer-Centric Strategies
Personalized pricing by user category

Expand to high-demand but underserved areas

6.4 Strategic Initiatives
Time Frame	Action
0–3 months	Dynamic pricing rollout, allocate drivers
3–12 months	Build real-time analytics system
12+ months	Geographic expansion, loyalty programs

📊 7. Success Metrics
KPI	Target
Revenue Growth	+[X]% in 12 mo
Driver Utilization	+[Y]%
Wait Time Reduction	−[Z]%
Customer Retention	+[A]%
Avg. Service Rating	[B.B]/5

📎 Appendices
Appendix A – Data Dictionary: [variable descriptions]

Appendix B – Statistical Output: [tests, confidence intervals]

Appendix C – Visualization Gallery:
📍 Screenshot placeholder: Histogram, Boxplot, Time Series

Appendix D – Code Documentation:
Python preprocessing, DAX formulas




