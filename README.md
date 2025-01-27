### Instagram Influencer’s account analysis using SQL


#### 🤔 Problem Statement:

Analyze Instagram activity data of a tech influencer to uncover trends, evaluate account growth, and identify impactful content strategies. The goal is to provide actionable insights to enhance engagement and audience reach.

#### ✅ Task:
As a data analyst, my task was to analyze datasets and answer key business questions, and present actionable insights business stakeholders.

#### 🔧 Tool Used: 
MySQL Workbench<br>

### 🛢️ Queries to address

1. How many unique post types are found in the 'fact_content' table?

```
SELECT COUNT(DISTINCT post_type) unique_post_types
FROM fact_content;
```

2. What are the highest and lowest recorded impressions for each post type?
 ``` 
SELECT post_type,
       MAX(impressions) highest_recorded_impressions,
       MIN(impressions) lowest_recorded_impressions
FROM fact_content
GROUP BY post_type
ORDER BY post_type; 
 ```

3. Filter all the posts that were published on a weekend in the month of 
March and April and export them to a separate csv file.
 ``` 
SELECT d.date,post_category,post_type,
       video_duration,carousel_item_count,impressions,
       reach,shares,follows,likes,comments,saves
FROM fact_content c
JOIN dim_dates d ON c.date=d.date
WHERE weekday_or_weekend="weekend" AND month_name IN ("March","April"); 
 ```

4. Create a report to get the statistics for the account. The final output 
includes the following fields: <br>
• month_name <br>
• total_profile_visits <br>
• total_new_followers<br>

```
SELECT month_name,
       SUM(profile_visits) AS total_profile_visits,
       SUM(new_followers) AS total_new_followers
FROM fact_account a 
JOIN dim_dates d ON a.date=d.date
GROUP BY month_name;
```

5. Write a CTE that calculates the total number of 'likes’ for each 
'post_category' during the month of 'July' and subsequently, arrange the 
'post_category' values in descending order according to their total likes.

```
WITH total_likes_by_post_category AS(
SELECT post_category,
       SUM(likes) AS total_likes
FROM fact_content c 
JOIN dim_dates d ON c.date=d.date
WHERE month_name="July"
GROUP BY post_category,month_name)
SELECT post_category,
       total_likes
FROM total_likes_by_post_category
ORDER BY total_likes DESC;
```





6. Create a report that displays the unique post_category names alongside 
their respective counts for each month. The output should have three 
columns:  <br>
• month_name <br>
• post_category_names  <br>
• post_category_count <br>
Example:  
• 'April', 'Earphone,Laptop,Mobile,Other Gadgets,Smartwatch', '5' <br>
• 'February', 'Earphone,Laptop,Mobile,Smartwatch', '4'<br>

```
SELECT month_name,
       post_category_names,
       post_category_count 
FROM (
SELECT month_name,
       MONTH(d.date) AS month_order,
       GROUP_CONCAT(DISTINCT post_category) AS post_category_names,
       LENGTH(GROUP_CONCAT(DISTINCT post_category))-
       LENGTH(REPLACE(GROUP_CONCAT(DISTINCT post_category ),",",""))+1 AS post_category_count
FROM fact_content c 
JOIN dim_dates d ON c.date=d.date
GROUP BY month_name,month_order) x
ORDER BY month_order;
```


7. What is the percentage breakdown of total reach by post type?  The final 
output includes the following fields: <br>
• post_type <br>
• total_reach <br>
• reach_percentage <br>

```
WITH total_reach_by_post_type AS(
SELECT post_type,
       SUM(reach) AS total_reach
FROM fact_content
GROUP BY post_type)
SELECT post_type,
       total_reach,
       ROUND(total_reach*100/SUM(total_reach) OVER(),2) AS reach_percentage
FROM total_reach_by_post_type;
```


8. Create a report that includes the quarter, total comments, and total 
saves recorded for each post category. Assign the following quarter 
groupings: 
(January, February, March) → “Q1” <br>
(April, May, June) → “Q2” <br>
(July, August, September) → “Q3” <br>
The final output columns should consist of: <br>
• post_category <br>
• quarter <br>
• total_comments <br>
• total_saves<br>

```
SELECT post_category,
       CASE WHEN month_name IN ("January", "February", "March") THEN "Q1"
			WHEN month_name IN ("April", "May", "June") THEN "Q2"
			WHEN month_name IN ("July", "August", "September") THEN "Q3"
	   END AS quarter,
	   SUM(comments) total_comments,
	   SUM(saves) total_saves
FROM fact_content c 
JOIN dim_dates d ON c.date=d.date
GROUP BY post_category,quarter
ORDER BY post_category,quarter;
```


9. List the top three dates in each month with the highest number of new 
followers. The final output should include the following columns: <br>
• month <br>
• date <br>
• new_followers<br>

```
WITH new_followers_by_month AS (
SELECT month_name,d.date,month(d.date) AS month_order,
       SUM(new_followers) AS new_followers,
       DENSE_RANK() OVER(PARTITION BY month_name ORDER BY SUM(new_followers) DESC) AS drnk
FROM fact_account a 
JOIN dim_dates d ON a.date=d.date
GROUP BY month_name,d.date,month_order)
SELECT month_name,
       date,
       new_followers
FROM new_followers_by_month
WHERE drnk<=3
ORDER BY month_order;
```


10.  Create a stored procedure that takes the 'Week_no' as input and 
generates a report displaying the total shares for each 'Post_type'. The 
output of the procedure should consist of two columns: <br>
• post_type <br>
• total_shares<br>

```
CREATE PROCEDURE weekly_post_shares_report(week_number VARCHAR(255))
SELECT post_type,
       SUM(shares) AS total_shares
FROM fact_content c 
JOIN dim_dates d ON d.date=c.date
WHERE week_no=week_number
GROUP BY post_type;

CALL weekly_post_shares_report("W3");
```


Project Presentation Link: https://youtu.be/hDHycouCZEU

LinkedIn Post Link: https://shorturl.at/OetVy
