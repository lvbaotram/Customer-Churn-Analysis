# Telecom Customer Churn Analysis
## Introduction
The telecom industry is highly competitive, and customers have a wide selection of options to choose from. Companies rely heavily on customer retention to ensure continued growth and success. However, losing customers, also known as churn, is inevitable in any industry. As a data analyst, my job is to provide an in-depth analysis of Maven Telecom’s churn dataset and to answer the following questions:
* What are the key drivers of churn?
* Is the company losing high value customers? If so, how can they retain them?
* What is the ideal profile of a churned customer?
* What steps can Maven Telecom take to reduce churn?
## Project Strategy
I downloaded the dataset from Maven Analytics and it contained information about customer demographics, subscription plans and account records for Maven Telecom. I performed all data preparation and analysis using SQL (DBeaver), and all the SQL codes will be provided below. The visualisations and dashboard were designed with Tableau.

The main steps for this project are:
* Data Cleaning and Preparation
* Exploratory Data Analysis
* Data insights
* Customer Retention Strategies
* Data Visualization

### Data Cleaning and Preparation
There are likely some null values in this dataset because each customer has their own unique subscription preferences. Including these null values in my analysis is intentional and helps me give a more detailed and complete understanding of Maven's customer base.

I checked for duplicate values in the unique key (Customer_ID), and found none.
```
-- Total Number of customers
SELECT
COUNT(DISTINCT Customer_ID) AS customer_count
FROM
dbo.churn

-- check for duplicates in customer ID
SELECT Customer_ID,
COUNT( Customer_ID )as count
FROM dbo.churn
GROUP BY Customer_ID
HAVING count(Customer_ID) > 1;
```


![](https://github.com/user-attachments/assets/51da7dd1-f50a-4b42-b5c8-ffd25cb40bdd)

### Exploratory Analysis
There are 7043 customers in total
#### 1. How much revenue was lost to churned customers?
Maven lost 1869 customers and they accounted for 17% of Maven's total revenue. It's a pretty significant figure and we'll investigate why they are leaving later on in this project.

```
-- How much revenue did maven lose to churned customer?
SELECT Customer_Status, COUNT(Customer_ID) AS customer_count,
ROUND((SUM(Total_Revenue) * 100.0) / SUM(SUM(Total_Revenue)) OVER(), 1) AS Revenue_Percentage 
FROM dbo.churn
GROUP BY Customer_Status
```
![](https://github.com/user-attachments/assets/cbac999f-db6e-4020-b7db-16b4cff53a4a)


## Insights
## Customer Retention Strategies
