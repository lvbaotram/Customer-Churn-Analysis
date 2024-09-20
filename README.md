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
The top 3 reasons for churn are ‘Competitor made **better offer**’, ‘Competitor had **better devices**’ and ‘**Attitude** of support person’.
```
-- why exactly did customers churn?
SELECT TOP 5
    Churn_Reason,
    Churn_Category,
    ROUND(COUNT(Customer_ID) *100 / SUM(COUNT(Customer_ID)) OVER(), 1) AS churn_percentage
FROM
    dbo.churn
WHERE
    Customer_Status = 'Churned'
GROUP BY 
Churn_Reason,
Churn_Category
ORDER BY churn_percentage DESC;
```
![q6](https://github.com/user-attachments/assets/de596305-78ca-4bbe-9917-7e138248a306)

#### 5b. What offers did churned customers have?
56% of churners did not have any promotional offer while 23% had Offer E. Offers are a great way to reward and retain your loyal customers.
```
-- What offers did churners have?
SELECT  
    Offer,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS churned
FROM
    dbo.churn
WHERE
    Customer_Status = 'Churned'
GROUP BY
Offer
ORDER BY 
churned DESC;
```
![q7](https://github.com/user-attachments/assets/5bb6d202-1b60-4c52-a53a-c1684f7b0cf9)

#### 5c. What internet type did churned have?
66% of all churned customers used Fiber Optic. While ~70% of customers who left for competitors also used Fiber Optic. Maven should review the quality and service of their Fiber Optic internet, as this could be the reason customers are leaving to competitors.
```
-- What Internet Type did churners have?
SELECT
    Internet_Type,
    COUNT(Customer_ID) AS Churned,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM
    dbo.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY
Internet_Type
ORDER BY 
Churned DESC;

-- What Internet Type did 'Competitor' churners have?
SELECT
    Internet_Type,
    Churn_Category,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM
    dbo.churn
WHERE 
    Customer_Status = 'Churned'
    AND Churn_Category = 'Competitor'
GROUP BY
Internet_Type,
Churn_Category
ORDER BY Churn_Percentage DESC;
```
![q8](https://github.com/user-attachments/assets/279d19f5-b396-448d-a0b1-2416801527b8)
![q9](https://github.com/user-attachments/assets/4f65be3d-6709-4539-9378-ae76f23c27bc)
#### 5d. Did churned have premium tech support?
77% of churned customers did not have premium tech support. It’s possible that this service could have improved their after-sales experience and reduced churn.
```
-- Did churners have premium tech support?
SELECT 
    Premium_Tech_Support,
    COUNT(Customer_ID) AS Churned,
    ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS Churn_Percentage
FROM
    dbo.churn
WHERE 
    Customer_Status = 'Churned'
GROUP BY Premium_Tech_Support
ORDER BY Churned DESC;
```
![q10](https://github.com/user-attachments/assets/15a2d5f1-4480-493e-a11b-45c9973e13a4)

#### 5e. What contract were churners on?
Almost all churned customers (89%) were on the month-to-month contract.

Customers on a month-to-month contract are more likely to churn, as they have greater flexibility to cancel or switch providers without incurring any penalty.
```
-- What contract were churners on?
SELECT 
    Contract,
    COUNT(Customer_ID) AS Churned,
    ROUND(COUNT(Customer_ID) * 100.0 / SUM(COUNT(Customer_ID)) OVER(), 1) AS Churn_Percentage
FROM 
    dbo.churn
WHERE
    Customer_Status = 'Churned'
GROUP BY
    Contract
ORDER BY 
    Churned DESC;
```
![q11](https://github.com/user-attachments/assets/07bfb585-6124-4610-9f5a-c25ecbfdf37f)

## III. Insights
## IV. Customer Retention Strategies
