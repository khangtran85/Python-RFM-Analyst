# Python-RFM-Analysis
RFM Analysis is a Python-based project for analyzing customer behavior using Recency, Frequency, and Monetary metrics. It segments customers into meaningful groups for targeted marketing strategies and business insights, with scalable data processing and visualization features.
# Introduction
RFM (Recency, Frequency, Monetary) is a key component of Marketing Analysis, used to assess customer value. By segmenting customers based on these three dimensions, businesses can tailor marketing strategies and customer care to specific groups. SuperStore, a global retail company, has a vast customer base due to its expansive operations across multiple regions.

As part of its Christmas and New Year campaigns, the Marketing Department aims to run targeted marketing strategies to appreciate loyal customers and identify potential ones who can become repeat buyers. However, with the company’s growing customer base, manual segmentation, which worked in previous years, is no longer feasible due to the large data size.

To solve this challenge, the Data Analysis team has been asked to assist in implementing a customer segmentation solution using the RFM model. The Marketing Director proposed leveraging the RFM model for segmentation, which was previously handled using Excel by a smaller team. Now, with an enormous dataset, the goal is to build an automated solution using Python to efficiently perform the segmentation and enable more targeted marketing initiatives.
# Objectives
- *Segment Customers Using the RFM Model:* Implement the RFM model to categorize customers based on their Recency, Frequency, and Monetary metrics.

- *Automate the Segmentation Process:* Build a Python-based solution to process and analyze large volumes of data, automating customer segmentation.

- *Provide Insights for Targeted Campaigns:* Deliver actionable insights to the Marketing Department by identifying high-value customers and potential loyal customers for tailored marketing strategies.

- *Enhance Efficiency:* Enable SuperStore’s team to handle large datasets and perform segmentation faster and more accurately compared to manual methods.
# Processing
## Conect data from Drive
Conect data from DriveConnect to data from my Google Drive, specifically an Excel file containing the dataset for RFM analysis.
``` python
from google.colab import drive
drive.mount('/content/drive')

import pandas as pd

ecomerce_retail_source = pd.read_excel('/content/drive/MyDrive/Python/ecommerce retail.xlsx')
```
## EDA:
Examine data characteristics, including the data type of each column, and check for the presence of null, missing, or error values using the dataprep.eda library and the create_report method.
``` python
!pip install dataprep
from dataprep.eda import create_report
create_report(ecomerce_retail)
```
After analyzing the dataset, the following insights were identified:

- The dataset comprises **8 columns** and **541,909 rows**, of which approximately **3.1% of the data is missing**, amounting to 136,534 cells, and **1% of the rows are duplicates**.

- The CustomerID column contains **24.93% missing value**s, which must be resolved. Since the project focuses on customer segmentation using the RFM model, the presence of a valid CustomerID is essential for identifying customers. Missing values in this column cannot be ignored.

- The Quantity and UnitPrice columns, while containing numerical data, exhibit errors. Specifically, 2% of the rows in Quantity (around 10,624 entries) and 2 entries in UnitPrice have negative values. Moreover, 0.5% of the values in UnitPrice are zero, which is unrealistic, as both Quantity and UnitPrice should represent positive values in a real-world context.

- Certain columns are deemed redundant for the RFM model and can be excluded to streamline the analysis.

- Outliers were detected in the Quantity and UnitPrice columns. These outliers cannot be eliminated outright, as they may represent valid transactions or significantly contribute to the company’s revenue. A thorough investigation with relevant departments is required to verify their validity and impact.
## Handle data (mising data, outlier data and error data):
Handling of invalid data is based on the observations mentioned above. Additionally, the InvoiceDate column is reformatted to facilitate easier calculation of the Recency score.
``` python
ecomerce_retail = ecomerce_retail[ecomerce_retail["Quantity"] > 0]
ecomerce_retail = ecomerce_retail[ecomerce_retail["UnitPrice"] > 0]
ecomerce_retail.drop_duplicates(inplace=True)
ecomerce_retail = ecomerce_retail[ecomerce_retail["CustomerID"].notnull()]
ecomerce_retail["InvoiceDate"] = pd.to_datetime(ecomerce_retail["InvoiceDate"], format="%d/%m/%Y")
```
## Classify customers according to the RFM model:
After reviewing and refining the dataset, the next step involves classifying customers using the RFM (Recency, Frequency, Monetary) model. This classification provides valuable insights into customer behavior, enabling businesses to design targeted marketing strategies. The process begins by preparing key data columns to calculate the RFM values effectively.

**Recency** represents the time gap between the most recent transaction and a specified reference date, which is *31/12/2011* for this project. A longer gap indicates a higher risk of customer churn, suggesting the need for re-engagement strategies. Conversely, a shorter gap presents opportunities for actions like upselling or cross-selling, increasing the likelihood of converting these customers into loyal, frequent buyers.

For the Recency metric, the necessary data is contained in the DateDiff column, which is calculated based on the concept of recency — the time difference between the last transaction and a specified reference date.
``` python
end_date = pd.to_datetime("31/12/2011", format="%d/%m/%Y")
ecomerce_retail["DateDiff"] = abs((end_date - ecomerce_retail["InvoiceDate"]).dt.days)
```

**Frequency** measures how often a customer interacts with the company through transactions. A higher frequency often correlates with stronger brand loyalty and responsiveness to marketing campaigns, making these customers ideal candidates for retention efforts. Frequency serves as a direct measure of the strength of the relationship between the customer and the business.

As for Frequency, it represents the total number of transactions that occurred within the designated period, serving as a measure of purchasing frequency. This is derived from the existing InvoiceNo column in the dataset, which uniquely identifies transaction order numbers.

**Monetary** quantifies the total spending of a customer within a defined period, *from their first purchase to 31/12/2011* in this case. This metric helps identify high-value customers, as those who spend more are likely to have greater trust in the brand. Such customers are more inclined to make significant purchases, underscoring their importance to the company’s long-term success.

The essential data column required for Monetary is TotalInvoiceValue, aptly named to reflect the total value of invoices that customers have spent during the analyzed time period. This value is derived as the product of two variables: Quantity and UnitPrice.
``` python
ecomerce_retail["TotalInvoiceValue"] = ecomerce_retail["Quantity"] * ecomerce_retail["UnitPrice"]
```
Next is the process of aggregating data by each customer ID.
``` python
DF1 = ecomerce_retail.groupby("CustomerID", as_index=False).agg({"InvoiceNo": "nunique", "TotalInvoiceValue": "sum", "DateDiff": "min"})
```
Use the rank method with the pct=True parameter in the pandas.DataFrame library to rank the values based on their percentiles, ranging from 0 to 1. For the Recency value, rank them in ascending order (instead of assigning the largest value a rank of 1, it will receive the highest rank in ascending order).

When multiplied by 100, the ranking range will be from 0 to 100.
``` python
DF1["Recency"] = DF1["DateDiff"].rank(pct=True, ascending=False) * 100
DF1["Frequency"] = DF1["InvoiceNo"].rank(pct=True) * 100
DF1["Monetary"] = DF1["TotalInvoiceValue"].rank(pct=True) * 100
```
The next step is to convert the ranking score from 0 to 100 into a scale with 5 discrete values ranging from 1 to 5.

Next, these scores are aggregated into a single RFM score for classification. For example, a customer with Recency = 1, Frequency = 2, and Monetary = 1 will have an RFM score of 121.
``` python
DF2 = DF1.copy()
for col in ["Recency", "Frequency", "Monetary"]:
  condlist1 = [
      (DF2[col] < 20),
      ((DF2[col] >= 20) & (DF2[col] < 40)),
      ((DF2[col] >= 40) & (DF2[col] < 60)),
      ((DF2[col] >= 60) & (DF2[col] < 80)),
      (DF2[col] >= 80)
  ]
  choicelist1 = ["1", "2", "3", "4", "5"]
  DF2[col + "Score"] = np.select(condlist1, choicelist1)
DF2["RFMScore"] = DF2["RecencyScore"] + DF2["FrequencyScore"] + DF2["MonetaryScore"]
DF2["RFMScore"] = DF2["RFMScore"].astype(int)
```
The final step is customer segmentation. There are 11 groups based on the RFM model, which are: "Champions", "Loyal", "Potential Loyalist", "New Customers", "Promising", "Need Attention", "About To Sleep", "At Risk", "Cannot Lose Them", "Hibernating Customers", and "Lost Customers".

Each group consists of different combinations of RFM scores, and within each group, there is a certain level of similarity in terms of RFM characteristics. For example, the "About To Sleep" group includes customers who haven’t made a purchase for a while (indicated by a low Recency score, around 3), have made fewer transactions in the past (Frequency score below 3), and have low monetary value (Monetary score around 1 or 2).
``` python
DF3 = DF2.copy()
condlist2 = [
    (DF3["RFMScore"].isin([555, 554, 544, 545, 454, 455, 445])),
    (DF3["RFMScore"].isin([543, 444, 435, 355, 354, 345, 344, 335])),
    (DF3["RFMScore"].isin([553, 551, 552, 541, 542, 533, 532, 531, 452, 451, 442, 441, 431, 453, 433, 432, 423, 353, 352, 351, 342, 341, 333, 323])),
    (DF3["RFMScore"].isin([512, 511, 422, 421, 412, 411, 311])),
    (DF3["RFMScore"].isin([525, 524, 523, 522, 521, 515, 514, 513, 425, 424, 413, 414, 415, 315, 314, 313])),
    (DF3["RFMScore"].isin([535, 534, 443, 434, 343, 334, 325, 324])),
    (DF3["RFMScore"].isin([331, 321, 312, 221, 213, 231, 241, 251])),
    (DF3["RFMScore"].isin([255, 254, 245, 244, 253, 252, 243, 242, 235, 234, 225, 224, 153, 152, 145, 143, 142, 135, 134, 133, 125, 124])),
    (DF3["RFMScore"].isin([155, 154, 144, 214, 215, 115, 114, 113])),
    (DF3["RFMScore"].isin([332, 322, 233, 232, 223, 222, 132, 123, 122, 212, 211])),
    (DF3["RFMScore"].isin([111, 112, 121, 131, 141, 151]))
]
choicelist2 = ["Champions", "Loyal", "Potential Loyalist", "New Customers", "Promising", "Need Attention", "About To Sleep", "At Risk", "Cannot Lose Them", "Hibernating Customers", "Lost Customers"]
DF3["CustomerType"] = np.select(condlist2, choicelist2)
```
## Visualization of the RFM model

# Recomendation
