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
```-- Total Number of customers
SELECT
COUNT(DISTINCT Customer_ID) AS customer_count
FROM
dbo.churn```
![q1](https://github.com/user-attachments/assets/a0749dac-2097-4663-bc27-5959adb589a8)

## Insights
## Customer Retention Strategies
