## Active User Retention

Assume you're given a table containing information on Facebook user actions. Write a query to obtain number of monthly active users (MAUs) in July 2022, including the month in numerical format "1, 2, 3".

An active user is defined as a user who has performed actions such as 'sign-in', 'like', or 'comment' in both the current month and the previous month.

### user_actions Table:

|Column Name | Type|
|-------------|-------------|
|user_id	|integer|
|event_id	|integer|
|event_type	|string ("sign-in, "like", "comment")|
|event_date	|datetime|

### user_actions Example Input:

| user_id | event_id | event_type | event_date          |
|---------|----------|------------|---------------------|
| 445     | 7765     | sign-in    | 05/31/2022 12:00:00 |
| 742     | 6458     | sign-in    | 06/03/2022 12:00:00 |
| 445     | 3634     | like       | 06/05/2022 12:00:00 |
| 742     | 1374     | comment    | 06/05/2022 12:00:00 |
| 648     | 3124     | like       | 06/18/2022 12:00:00 |

### Example Output for June 2022:
| month | monthly_active_users |
|-------|----------------------|
| 6     | 1                    |

Example
In June 2022, there was only one monthly active user (MAU) with the user_id 445.

Please note that the output provided is for June 2022 as the user_actions table only contains event dates for that month. You should adapt the solution accordingly for July 2022.

The dataset you are querying against may have different input & output - this is just an example!


## My solutions to the problem

#### Solution using GROUP BY

~~~~sql
WITH cte as (SELECT DISTINCT user_id, to_char(event_date, 'MM/YY') as date,
COUNT(*) OVER (PARTITION BY user_id)
FROM user_actions
WHERE EXTRACT(MONTH FROM event_date) in (6,7)
GROUP BY user_id, date)

SELECT 7 as month, COUNT(count) as monthly_active_users FROM cte
WHERE count = 2 AND date = '07/22'
~~~~

#### Solution using two subqueries and the AND operator

~~~~sql
SELECT 7 as month, count(DISTINCT user_id) as monthly_active_users FROM user_actions
WHERE 
(user_id, '06/22') in (SELECT user_id, to_char(event_date, 'MM/YY') FROM user_actions)
AND
(user_id, '07/22') in (SELECT user_id, to_char(event_date, 'MM/YY') FROM user_actions)
~~~~

#### Solution using two subqueries and an intersection

~~~~sql
SELECT 7 as month, count(user_id) as monthly_active_users FROM
(SELECT user_id FROM user_actions WHERE to_char(event_date, 'MM/YY') = '06/22'
INTERSECT
SELECT user_id FROM user_actions WHERE to_char(event_date, 'MM/YY') = '07/22') 
as t2
~~~~

#### Solution using self join

~~~~sql
SELECT 7 as month, COUNT(DISTINCT t1.user_id) as monthly_active_users 
FROM user_actions t1
JOIN user_actions t2 
ON t1.user_id = t2.user_id
AND to_char(t1.event_date, 'MM/YY') != to_char(t2.event_date, 'MM/YY')
AND to_char(t1.event_date, 'MM/YY') in ('06/22', '07/22')
AND to_char(t2.event_date, 'MM/YY') in ('06/22', '07/22')
~~~~
