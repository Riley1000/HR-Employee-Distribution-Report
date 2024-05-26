# HR Employee Distribution Report

## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results and Findings](#results-and-findings)
- [Limitations](#limitations)

### Project Overview

A project detailing employee distribution in a company under various categories to seek insights and find answers to a few questions asked.

![Uploading hr dashboard.PNG…]()

![hr dashboard](https://github.com/Riley1000/HR-Employee-Distribution-Report/assets/110679126/8a14f9f6-4771-4df6-be13-3442528b9564)

### Data Sources 

The primary dataset used fot this analysis is the "Human_Resource.csv" file. It contains information about employees spaning various categories from the hire_date to the termdate.

### Tools

- SQL (MySQL server) - Data cleaning and analysis
- PowerBI - Data report and viz

 ### Data Cleaning and Preparation
 
1. A database was created in MySQL and the data imported using the "table import wizard"
2. Table formatting and data type changes

### Exploratory Data Analysis

The dataset was explored, deductions made and a few questionss asked to understand the best way to generate unbiased insights and give correct answers to the questions asked by HR. 

### Data Analysis

Analysis was done in the MySQL server and the most interesting part of the analysis was the use of subqueries to generate insights into the data
``` SQL
ALTER TABLE hr
CHANGE COLUMN ï»¿id emp_id VARCHAR(20) NULL;

DESCRIBE hr;

SELECT birthdate FROM hr;

SET Sql_safe_updates = 0;

UPDATE hr
SET birthdate = CASE
WHEN birthdate LIKE '%/%' THEN date_format(str_to_date(birthdate, '%m/%d/%Y'),'%Y-%m-%d')
WHEN birthdate LIKE '%-%' THEN date_format(str_to_date(birthdate, '%m-%d-%Y'),'%Y-%m-%d')
ELSE NULL
END;

ALTER TABLE hr
MODIFY COLUMN birthdate DATE;

SELECT birthdate FROM hr;

UPDATE hr
SET hire_date = CASE
WHEN hire_date LIKE '%/%' THEN date_format(str_to_date(hire_date, '%m/%d/%Y'),'%Y-%m-%d')
WHEN hire_date LIKE '%-%' THEN date_format(str_to_date(hire_date, '%m-%d-%Y'),'%Y-%m-%d')
ELSE NULL
END;

ALTER TABLE hr
MODIFY COLUMN hire_date DATE;


UPDATE hr
SET termdate = IF(termdate IS NOT NULL AND termdate != '', date(str_to_date(termdate, '%Y-%m-%d %H:%i:%s UTC')), '0000-00-00')
WHERE true;

ALTER TABLE hr
MODIFY COLUMN termdate DATE;

ALTER TABLE hr ADD COLUMN age INT;

UPDATE hr
SET age = TIMESTAMPDIFF(YEAR, birthdate, CURDATE());

SELECT
	min(age) AS youngest,
    max(age) AS oldest
FROM hr;

SELECT * FROM hr;

/* THE CODES BELOW ARE WRITTEN TO SPECIFICALLY ANSWER THE QUESTIONS(1-11) ABOVE */

/* 1 */
SELECT gender, count(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY gender;

/* 2 */
SELECT race, count(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY race
ORDER BY count(*) DESC;

/* 3 */
SELECT
	min(age) AS youngest,
    max(age) AS oldest
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00';

SELECT 
	CASE
	WHEN age >= 18 AND age <= 24 THEN '18-24'
	WHEN age >= 25 AND age <= 34 THEN '25-34'
	WHEN age >= 35 AND age <= 44 THEN '35-44'
	WHEN age >= 45 AND age <= 54 THEN '45-54'
    WHEN age >= 55 AND age <= 64 THEN '55-64'
	ELSE '65+'
	END AS age_group,
count(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY age_group
ORDER BY age_group;

SELECT 
	CASE
	WHEN age >= 18 AND age <= 24 THEN '18-24'
	WHEN age >= 25 AND age <= 34 THEN '25-34'
	WHEN age >= 35 AND age <= 44 THEN '35-44'
	WHEN age >= 45 AND age <= 54 THEN '45-54'
    WHEN age >= 55 AND age <= 64 THEN '55-64'
	ELSE '65+'
	END AS age_group, gender,
count(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY age_group, gender
ORDER BY age_group, gender;

/* 4 */
SELECT location, count(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY location;

/* 5 */
SELECT
	round(avg(datediff(termdate, hire_date))/365,0) AS avg_length_employment
FROM hr
WHERE termdate <= curdate() AND termdate <> '0000-00-00' AND age >= 18;

/* 6 */
SELECT department, gender, count(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY department, gender
ORDER BY department;

/*7 */
SELECT jobtitle, count(*) AS count
FROM hr
WHERE age >= 18 AND termdate ='0000-00-00'
GROUP BY jobtitle
ORDER BY jobtitle DESC;

/*8 */
SELECT department,
	total_count,
    terminated_count,
    terminated_count/total_count AS termination_rate
FROM (
	SELECT department,
    count(*) AS total_count,
    sum(CASE WHEN termdate <> '0000-00-00' AND termdate <= curdate() THEN 1 ELSE 0 END) AS terminated_count
FROM hr 
WHERE age >= 18
GROUP BY department
) AS subquery
ORDER BY termination_rate DESC;

/* 9 */
SELECT location_state, count(*) AS count
FROM hr 
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY location_state
ORDER BY count DESC;

/* 10 */
SELECT 
	year,
    hires,
    terminations,
    hires - terminations AS net_change,
    round((hires - terminations)/hires * 100, 2) AS net_change_percent
FROM (
	SELECT
		YEAR(hire_date) AS year,
        count(*) AS hires,
        SUM(CASE WHEN termdate <> '0000-00-00' AND termdate <= curdate() THEN 1 ELSE 0 END) AS terminations
        FROM hr 
        WHERE age >= 18
        GROUP BY year(hire_date)
	) AS subquery
ORDER BY YEAR ASC;

/* 11 */
SELECT department, round(avg(datediff(termdate, hire_date)/365),0) AS avg_tenure
FROM hr 
WHERE termdate <= curdate() AND termdate <> '0000-00-00' AND age >= 18
GROUP BY department;
```

### Results and Findings
The Following questions where answered and forms the results and findings from the analysis. 
1. What is the gender breakdown of employees in the company?

2. What is the race/ethnicity breakdown of employees in the company?

3. What is the age distribution of employees in the company?

4. How many employees work at headquarters versus remote locations?

5. What is the average length of employment for employees who have been terminated?

6. How does the gender distribution vary across departments and job titles?

7. What is the distribution of job titles across the company?

8. Which department has the highest turnover rate?

9. What is the distribution of employees across locations by state?

10. How has the company's employee count changed over time based on hire and term dates?

11. What is the tenure distribution for each department?

### Limitations

- Some records had negative ages and they were excluded during querying (967 records) and only ages 18 years and avove were used.
- Some termdates were far into the future (e.g 2067) and they were excluded during the analysis (1599 records). The only termdates used were those  equal to theh current date. 
