# Telecom Customer Churn Analysis
## I. Introduction
The telecom industry is highly competitive, and customers have a wide selection of options to choose from. Companies rely heavily on customer retention to ensure continued growth and success. However, losing customers, also known as churn, is inevitable in any industry. As a data analyst, my job is to provide an in-depth analysis of Maven Telecom’s churn dataset and to answer the following questions:
* What are the key drivers of churn?
* Is the company losing high value customers? If so, how can they retain them?
* What is the ideal profile of a churned customer?
* What steps can Maven Telecom take to reduce churn?
## II. Project Strategy
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

#### 2. What's the typical tenure for churned customers?
To find out how long customers usually stay at Maven before leaving, I used a CASE statement in SQL. This CASE statement creates a new column (Tenure) and groups customers who leave the company in 12 months or less as ‘12 months,’ and so on.

I found that ~42% of churned customers spent 6 months or less at Maven before leaving.

Nearly half of the customers who left had a short time with the company, meaning Maven has a chance to improve customer retention, especially with new customers. 

```
-- Typical tenure for new customers?
SELECT
    CASE 
        WHEN Tenure_in_Months <= 6 THEN '6 months'
        WHEN Tenure_in_Months <= 12 THEN '1 Year'
        WHEN Tenure_in_Months <= 24 THEN '2 Years'
        ELSE '> 2 Years'
    END AS Tenure,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS Churn_Percentage
FROM
dbo.churn
WHERE
Customer_Status = 'Churned'
GROUP BY
    CASE 
        WHEN Tenure_in_Months <= 6 THEN '6 months'
        WHEN Tenure_in_Months <= 12 THEN '1 Year'
        WHEN Tenure_in_Months <= 24 THEN '2 Years'
        ELSE '> 2 Years'
    END
ORDER BY
Churn_Percentage DESC;
```
![](https://github.com/user-attachments/assets/297eb9c5-6a06-49d6-bbcf-000087fc3dac)

#### 3. Which cities had the highest churn rates?
Churn rate measures the percentage of customers who stop using the services of a company over a certain period of time.

For the purpose of this analysis, I only considered cities with more than 30 customers in total, because some cities had very few customers and my conclusion would have been biased towards them.

San Diego had the highest churn rate at 65%, which means that over half of their customers have left the company.
```
SELECT
    TOP 4 City,
    COUNT(Customer_ID) AS Churned,
    CEILING(COUNT(CASE WHEN Customer_Status = 'Churned' THEN Customer_ID ELSE NULL END) * 100.0 / COUNT(Customer_ID)) AS Churn_Rate
FROM
    dbo.churn
GROUP BY
    City
HAVING
    COUNT(Customer_ID)  > 30
AND
    COUNT(CASE WHEN Customer_Status = 'Churned' THEN Customer_ID ELSE NULL END) > 0
ORDER BY
    Churn_Rate DESC;
```
![](https://github.com/user-attachments/assets/efcf4ec4-1ea6-4866-8c25-020c6e221381)

#### 4. What are the general reasons for churn?
**45%** of churned customers stated ‘Competitor’ as their reason for leaving. It’s interesting to note that a significant number of customers **(17%)** left due to the **attitude** of support staff. Maven also lost about **$1.7 million** to **Competitors**, making it the most expensive type of churn.
```
SELECT 
  Churn_Category,  
  ROUND(SUM(Total_Revenue),0)AS Churned_Rev,
  CEILING((COUNT(Customer_ID) * 100.0) / SUM(COUNT(Customer_ID)) OVER()) AS Churn_Percentage
FROM 
  dbo.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY 
  Churn_Category
ORDER BY 
  Churn_Percentage DESC; 
```
![](https://github.com/user-attachments/assets/e8788639-0a14-4141-affe-df0cd99027e0)
#### 5a. Specific reasons for churn

## III. Insights
## IV. Customer Retention Strategies
