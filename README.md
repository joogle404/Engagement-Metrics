# SaaS Engagement Metrics

Tracking engagement is fundamental to understanding how users interact with a product—especially in SaaS environments where user retention and growth are critical to success. In this mini-project, I used PostgreSQL to calculate three essential metrics:

- **Daily Active Users (DAU)**
- **Monthly Active Users (MAU)**
- **User Growth Rate**

Using the `sf_events` table from Salesforce data on StrataScratch, I wrote queries to extract these metrics, which offer insight into user behavior, platform health, and growth trends. Here's the breakdown of each.


## Daily Active Users (DAU)

We are going to find average daily active users for the month of january 2021 for salesforce data. [Daily Active Users](https://platform.stratascratch.com/coding/2050-daily-active-users?code_type=1)  

### Table: `sf_events`

| Column Name | Data Type  |
|-------------|------------|
| `date`      | datetime   |
| `account_id`| varchar    |
| `user_id`   | varchar    |

### Objective

For each `account_id`, calculate the average number of **unique daily active users** during January 2021.

### Approach

1. **Filter** the dataset to only include records where `record_date` falls between **January 1 and January 30, 2021**.
2. **Group** the filtered data by `account_id` and `record_date`, then **count distinct `user_id`** to get the number of active users for each account on each day.
3. Use a **Common Table Expression (CTE)** to store the daily counts.
4. **Aggregate** the results by `account_id`, summing the daily active users and dividing by **31.0** to get the **average daily active users** per account.



### SQL Solution

```sql
-- Step 1: Create a CTE to calculate daily active users per account
WITH dau AS
(
SELECT
    account_id,
    record_date,
    COUNT(distinct user_id) AS daily_active_user -- Count unique users per day per account
FROM 
    sf_events
WHERE 
    record_date BETWEEN '2021-01-01' AND '2021-01-30' -- Filter to January 2021 (first 30 days)
GROUP BY
    account_id,
    record_date
)

-- Step 2: Aggregate the results from the CTE to calculate the average daily active users

SELECT
    account_id,
    SUM(daily_active_user) / 31.0 AS avg_dau -- Divide total by 31 to get the average
FROM 
    dau
GROUP BY 
    account_id
```
## Monthly Active Users (MAU) 

Next, we find the monthly active users for January 2021 for each account. The output should have `account_id` and the monthly count for that account. [Monthly Active Users](https://platform.stratascratch.com/coding/2051-monthly-active-users?code_type=1)

### Objective

For each `account_id`, determine the number of **unique users** who were active at least once during **January 2021**.

### Approach

1. **Filter** the dataset to include events where `record_date` falls between January 1 and January 30, 2021.
2. **Group** the data by `account_id` to calculate metrics on a per-account basis.
3. **Aggregate** the data by counting **distinct `user_id`s** for each `account_id` to get the Monthly Active Users (MAU).


---

### SQL Solution

```sql
-- Calculate Monthly Active Users (MAU) for each account from Jan 1 to Jan 30, 2021
SELECT
    account_id,
    COUNT(DISTINCT user_id) AS monthly_active_user  -- Count of unique users active in the month
FROM
    sf_events
WHERE
    record_date BETWEEN '2021-01-01' AND '2021-01-30'  -- Filter records for January 2021
GROUP BY
    account_id;  -- Group by account to get MAU per account
```

## User Growth Rate

[User Growth Rate](https://platform.stratascratch.com/coding/2052-user-growth-rate?code_type=1)

Finally, we calculate the growth rate of active users for December 2020 to January 2021 for each account. The growth rate is defined as the number of users in January 2021 divided by the number of users in December 2020.  


### Objective

For each `account_id`, calculate the growth rate of active users from December 2020 to January 2021.  

### Approach

1. **Create two Common Table Expressions (CTEs)**:  
   - The first CTE (`january_count`) filters records from January 2021 and counts the number of distinct users per `account_id`.
   - The second CTE (`december_count`) does the same for December 2020.

2. **Join** the two CTEs on `account_id` to align user counts for each account across both months.

3. **Calculate the growth rate** by dividing January's user count by December's user count for each `account_id`.

4. **Output** the `account_id` and corresponding growth rate.

```sql
-- CTE to count distinct users per account for January 2021
WITH january_count AS (
    SELECT
        account_id,
        COUNT(DISTINCT user_id) AS jan_count  -- Count of unique users in January
    FROM
        sf_events
    WHERE
        record_date BETWEEN '2021-01-01' AND '2021-01-30'  -- Filter for January
    GROUP BY
        account_id
),

-- CTE to count distinct users per account for December 2020
december_count AS (
    SELECT
        account_id,
        COUNT(DISTINCT user_id) AS dec_count  -- Count of unique users in December
    FROM
        sf_events
    WHERE
        record_date BETWEEN '2020-12-01' AND '2020-12-31'  -- Filter for December
    GROUP BY
        account_id
)

-- Final query to compute the growth rate by joining both CTEs
SELECT
    j.account_id,
    j.jan_count / d.dec_count AS growth_rate  -- Growth rate = Jan count / Dec count
FROM 
    january_count j
JOIN 
    december_count d
ON 
    j.account_id = d.account_id;

```

## Why This Matters in a SaaS Environment

Understanding **DAU**, **MAU**, and **User Growth Rate** is foundational to product analytics in any SaaS company. These metrics don’t just describe user behavior—they inform critical business decisions.

- **DAU** reflects stickiness and daily user habits.
- **MAU** highlights monthly engagement and helps detect churn risk.
- **User Growth Rate** indicates how well marketing and onboarding efforts are driving expansion.

Together, they form a core framework for evaluating product health, supporting feature prioritization, and designing user retention strategies.

Just as important as tracking these metrics is writing SQL in a way that’s **clean, reusable, and easy to maintain**. These queries are not one-off exercises—they’re tools that teams return to regularly across product reviews, performance check-ins, and executive dashboards.

That’s why each query here is structured with:
- **Common Table Expressions (CTEs)** for clarity and logical flow,
- **Descriptive naming** to ensure transparency,
- And **inline comments** to make the logic accessible to teammates or future you.

In fast-moving SaaS environments, clean analytical work is essential in enabling collaboration, reducing rework, and scaling insights across the organization.






