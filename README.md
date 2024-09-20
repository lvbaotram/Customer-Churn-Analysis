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

The **key churn indicators** are therefore:
* **Contract**: 89% of churned customers were on the month-to-month contract
* **Premium Tech Support**: 77% of churners did not have premium tech support
* **Internet Type**: 66% of churners used Fiber Optic internet
* **Offer**: 56% of churners did not have any promotional offers, while 23% had Offer E.
#### 6. Are high value customers at risk of churning?
I defined high value customers based on these factors and grouped them into 3 risk levels (High, Medium, Low):

* Tenure: This is a measure of loyalty, so I only considered customers that have been with the company for at least 9 months.

* Monthly Charge: If the customer’s total monthly charge is in the top 50th percentile.

* Referrals: customers who refer other customers to the business.

High-value customers with 3–4 churn indicators are High Risk, while Medium Risk customers have 2 and Low Risk customers have only 1. For instance, a high-value customer at high risk of churning may use fiber optic, have a month-to-month contract, and no promotional offers or premium tech support.
```
-- Are high value customers at risk?
SELECT 
    CASE 
        WHEN (num_conditions >= 3) THEN 'High Risk'
        WHEN num_conditions = 2 THEN 'Medium Risk'
        ELSE 'Low Risk'
    END AS risk_level,
    COUNT(Customer_ID) AS num_customers,
    ROUND(COUNT(Customer_ID) *100.0 / SUM(COUNT(Customer_ID)) OVER(),1) AS cust_percentage,
    num_conditions  
FROM 
    (
    SELECT 
        Customer_ID,
        SUM(CASE WHEN Offer = 'Offer E' OR Offer = 'None' THEN 1 ELSE 0 END)+
        SUM(CASE WHEN Contract = 'Month-to-Month' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Premium_Tech_Support = 'No' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Internet_Type = 'Fiber Optic' THEN 1 ELSE 0 END) AS num_conditions
    FROM 
        dbo.churn
    WHERE 
        Monthly_Charge > 70.05 
        AND Customer_Status = 'Stayed'
        AND Number_of_Referrals > 0
        AND Tenure_in_Months > 9
    GROUP BY 
        Customer_ID
    HAVING 
        SUM(CASE WHEN Offer = 'Offer E' OR Offer = 'None' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Contract = 'Month-to-Month' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Premium_Tech_Support = 'No' THEN 1 ELSE 0 END) +
        SUM(CASE WHEN Internet_Type = 'Fiber Optic' THEN 1 ELSE 0 END) >= 1
    ) AS subquery
GROUP BY 
    CASE 
        WHEN (num_conditions >= 3) THEN 'High Risk'
        WHEN num_conditions = 2 THEN 'Medium Risk'
        ELSE 'Low Risk'
    END, num_conditions; 
```
![Q12](https://github.com/user-attachments/assets/f6b8de1b-c9e0-41b7-8451-b6af8eeee772)
#### 7. Building the ideal churn customer profile
I built a simple churn profile using the key churn indicators I discussed in previous sections, and the churn demographic results below:

* 32% of churned customers are above 60 years old
* ~50% of churned customers are Female
* 64% of churned customers are Single
* 94% of churned customers have no dependents in their household; it’s possible they have more flexibility and fewer commitments, making it easier for them to switch providers or cancel their subscription.
  
You can view the sql codes for the demographic results here.
## III. Insights
* Maven has 1869 churned customers and 20% of them are high-value customers.
* 42% of churned customers only stayed for 6 months or less.
* The top 3 reasons for churn are competitors made better offers, competitors had better devices and attitude of support staff.
* Maven lost ~$1.7 million to competitors, making it the most expensive type of churn
* The key indicators of churn are Month-to-Month contract , No Premium Tech Support, Fiber Optic internet, No promotional offer and Offer E.
* 70% of customers who churned to competitors used Fiber Optic
* High value customers are churning at a rate of 23%
* Based on the key churn indicators, out of 1250 high-value customers remaining, 77% are at high risk of churning.
## IV. Customer Retention Strategies
* Loyalty Programs: Since the top reason for churn is ‘competitors making better offers’ , and more than half of churned customers did not have any promotional offers, Maven could implement different loyalty programs to retain their customers. For instance, they could reward customers on long-term contracts with discounted rates, free upgrade, or additional features.
* Improve Customer Support: Invest in training and development of support staff to ensure they provide excellent customer service. This could include regular coaching and feedback sessions, as well as incentives for staff who receive positive customer feedback.
* Make better devices: Evaluate the features, performance and pricing of your devices to ensure they are in line with market standards and demand.
* Premium Tech Support: Since customers who did not have access to premium tech support were more likely to churn, Maven should consider offering this service to all customers.
* Improve Fiber Optic Service: Invest in improving your Fiber Optic offerings like faster speeds, more stable connections, and better customer support for Fiber Optic customers.
* Engage High-Value Customers: Prioritise engaging these customers to prevent them from leaving. Provide personalised offers, send targeted communications, and provide premium tech support to ensure these customers remain satisfied with their service.
* After-Sales Service: Schedule regular check-ins with customers to ensure they are still satisfied with their service. These check-ins could be in the form of surveys, phone calls, or email communications.
Thanks for reading!
