# Introduction

📊 Dive into the data job market! Focusing on data job roles, this project explores 💰 top-paying jobs, 🔥 in-demand skills, and 📈 where high demand meets high salary in data analytics.

# Background

Driven by a quest to navigate the data analyst job market more effectivelly, this project was born from a desire to pinpoint top-paid and in-demand skills, streamlining others work to find optimal job.

The questions I wanted to answer through my sql queries were:
1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with high salaries?
5. What are the most optimal skills to learn?

# Tools I used

For my deep dive into the data analyst job market, I harnessed the power of several key tools:

 - SQL: The backbone of my analysis, allowing me to query the database and unearth critical insights.
 - PostgreSQL: The chosen database management system, ideal for handling the job posting data.
 - Visual Studio Code: My go-to work database management and executing SQL queries.
 - Git & Github: Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.

   # The Analysis
   
 Each query for this project aimed at investigating specific aspects of the data analyst jobs market. Here is how I approached each questions:

 1. Top Paying Data Analyst Jobs

To identify the highest-paying roles, I filtered data analyst position by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field.

```SELECT 
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM  
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id    
WHERE 
    job_title = 'Data Analyst'  AND 
    job_location = 'Anywhere' AND 
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC   
LIMIT 10

2. Skills for Top Paying Jobs

To understand what skills are required for top-paying jobs, I joined the job postings with skills data, providing insights into what employers value for high-compensation roles.

WITH top_paying_jobs AS (
    SELECT 
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM  
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id    
    WHERE 
        job_title = 'Data Analyst'  AND 
        job_location = 'Anywhere' AND 
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC   
    LIMIT 10
)
SELECT 
top_paying_jobs.*,
skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC

3. In-Demand Skills for Data Analysts

This query helped to identify the skills most frequently requested in job postings, directing focus to areas with high demand.

SELECT 
skills,
COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst' AND job_work_from_home = True
GROUP BY skills
ORDER BY demand_count DESC
LIMIT 5

Here's the breakdown of the most demanded skills for data analyst in 2024

| Rank | Skill       | Demand Count |
|------|-------------|--------------|
| 1    | SQL         | 7,292        |
| 2    | Excel       | 4,611        |
| 3    | Python      | 4,331        |
| 4    | Tableau     | 3,745        |
| 5    | Power BI    | 3,080        |

Key Insights:
SQL dominates with nearly twice as many job postings requiring it compared to the second skill (Excel)

Core technical skills (SQL, Python) and visualization tools (Tableau, Power BI) make up the entire top 5

Spreadsheet skills remain crucial despite being "basic" (Excel appears in 61% of jobs that mention SQL)

The Python vs. Excel gap is relatively small (only 280 job postings difference)

This data suggests that for remote Data Analyst roles in 2024, candidates need strong:

Database querying skills (SQL)

Programming fundamentals (Python)

Data visualization competency (Tableau/Power BI)

Foundational data manipulation abilities (Excel)

4. Skills Based on Salary

Exploring the average salaries associated with different skills revealed which skills are the highest paying.

SELECT 
skills,
ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst' AND job_work_from_home = True
AND salary_year_avg IS NOT NULL
GROUP BY skills
ORDER BY avg_salary DESC
LIMIT 25

Rank	Skill	           Avg Salary ($)
1	    PySpark	           208,172
2	    Elasticsearch	   189,477
3	    Databricks	   184,912
4	    Golang	           178,864
5	    Snowflake	   175,760
6	    Pandas	           174,933
7	    NumPy	           173,869
8	    Kafka	           172,795
9	    Airflow	           172,500
10	    Scala	           171,999
11	    Jenkins	           171,115
12	    Kubernetes	   170,647
13	    TensorFlow	   170,136
14	    Cassandra	   169,650
15	    Spark	           169,533
16	    Linux	           169,396
17	    NoSQL	           168,255
18	    AWS	           167,527
19	    R	           166,880
20	    Java	           166,722
21	    Machine Learning  166,348
22	    C++	           165,959
23	    BigQuery	   165,939
24	    Azure	           165,890
25	    PostgreSQL	   165,633

Key Insights:
PySpark leads at $208K, showing premium value for big data processing skills

Cloud/data engineering tools dominate (Databricks, Snowflake, Kafka, Airflow)

Python ecosystem remains strong (Pandas, NumPy, TensorFlow appear in top 15)

Surprise entry: Golang ranks #4, suggesting demand for multi-language analysts

$165K+ baseline: All top 25 skills command salaries above $165K


5. Most Optimal Skills to Learn

Combining insights from demand and salary data this query aimed to pinpoint skills that are both in high demand and have high salaries, offering a strategic focus for skill development.

WITH skills_demand AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE 
        job_title_short = 'Data Analyst' 
        AND job_work_from_home = True
        AND salary_year_avg IS NOT NULL
    GROUP BY 
        skills_dim.skill_id
), 
average_salary AS (
    SELECT 
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    WHERE 
        job_title_short = 'Data Analyst' 
        AND job_work_from_home = True
        AND salary_year_avg IS NOT NULL
    GROUP BY 
        skills_job_dim.skill_id
)
SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM skills_demand
INNER JOIN average_salary 
    ON skills_demand.skill_id = average_salary.skill_id
WHERE demand_count > 10    
ORDER BY avg_salary DESC,
demand_count DESC
LIMIT 25

Rank  Skill               Demand Count  Avg Salary ($)
-----  ------------------  ------------  --------------
1      PySpark            11            208,172
2      Databricks         14            184,912
3      Snowflake          20            175,760
4      Pandas             15            174,933
5      Kafka              12            172,795
6      Airflow            16            172,500
7      Scala              13            171,999
8      Kubernetes         11            170,647
9      TensorFlow         18            170,136
10     Spark              13            169,533
11     Linux              12            169,396
12     AWS                34            167,527
13     R                  27            166,880
14     Java               17            166,722
15     Machine Learning   33            166,348
16     BigQuery           22            165,939
17     Azure              35            165,890
18     PostgreSQL         23            165,633
19     Python             78            165,321
20     Tableau            49            164,534
21     Power BI           46            163,025
22     SQL                142           162,387
23     Excel              58            161,832
24     Looker             21            161,635
25     Redshift           18            160,832

Key Insights:

SQL has highest demand (142 postings) while maintaining $162K average

Python ecosystem dominates (Pandas, Spark, TensorFlow all appear)

PySpark tops both salary ($208K) and demand among high-paying skills

Cloud platforms show strong presence (AWS, Azure, Snowflake, Databricks)

$160K+ baseline: All listed skills pay above $160K with >10 job postings

# What I Learned

Throughout this adventure, I have turocharged my SQL toolkit with some serious firepower:

- Complex Query Crafting: Mastered art of advanced SQL, merging tables like a pro and weilding.
- Data Aggregation: Got cozy with GROUP BY and aggregate functions like COUNT() and AVG().
- Analytical Wizardry: Leveled up my real-world puzzle-solving-skills.

# Conclusion

## Insights

From the analysis, several general insights emerged:

1. Top-Paying Data Analyst jobs: The highest-paying jobs for data anlalyst that allow remote work offer a wide range of salaries the highest at $650,000!

2. Skills for Top-paying Jobs: High-paying data analyst jobs require advanced proficiency in SQL, suggesting it's a critical skill for earning top salary.

3. Most In-Demand Skills: SQL is also the most demanded skills in data analyst job market, thus making it essential for job seekers.

4. Skills With Higher Salaries: Specialized skills such as SVN and solidity, are associated with the highest average salaries.

5. Optimal Skills for Job Market Value: SQL leads in demand and offer for a higher average salary, positioning it as one of the most optimal skills for data analysts to learn to maximize their market value.

# Closing Thoughts

This project enhanced my SQL skills and provided valuable insights into the data analyst job market. The finding from the analysis serve as a guide to priortizing skill development and job search efforts. Aspiring data analysts can better position themselves in a comptitve job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continous learning and adaptation to emerging trends in the field of data analytics.










    
