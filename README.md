# sql_marathon
30 SQL Queries per day 
# Query #1

## Remove Redundant Pairs

### Problem Statement
- For pairs of brands in the same year (e.g., apple/samsung/2020 and samsung/apple/2020):
  - If **custom1 = custom3** and **custom2 = custom4**, then keep only one pair.
- For pairs of brands in the same year:
  - If **custom1 != custom3** OR **custom2 != custom4**, then keep both pairs.
- For brands that do not have pairs in the same year, keep those rows as well.

### Table
The data should be structured in a table with the following columns:
- **BRAND1**
- **BRAND2**
- **YEAR**
- **CUSTOM1**
- **CUSTOM2**
- **CUSTOM3**
- **CUSTOM4**

![1 query](https://github.com/leon-madara/sql_marathon/assets/147078093/05df7cf3-632e-4bd6-8ab8-52154456e2f2)

### SQL Commands
The SQL commands to create the table and load the data:

```sql
CREATE TABLE tech_giant (
	brand1 VARCHAR(255),
	brand2 VARCHAR(255),
	YEAR INT,
	custom1 INT,
	custom2 INT,
	custom3 INT,
	custom4 INT
);

INSERT INTO tech_giant (brand1, brand2, YEAR, custom1, custom2, custom3, custom4
	) VALUES
	('Apple', 'Samsung', 2020, 1, 2, 1, 2),
	('Samsung', 'Apple', 2020, 5, 5, 9, 2),
	('Apple', 'Samsung', 2021, 5, 3, 5, 3),
	('Google', NULL, 2020, 5, 9, NULL, NULL),
	('OnePlus', 'Nothing', 2021, 5, 9, 6, 3);
```

### The first step is to create pairs
The SQL Code for creating pairs

```sql
SELECT *, concat(brand1, brand2, YEAR) as pair_id
	, CASE WHEN brand1 < brand2 THEN concat(brand1, brand2, YEAR)
			ELSE concat(brand2, brand1, YEAR)
	END AS pair_id
FROM tech_giant;
```

![2 query](https://github.com/leon-madara/sql_marathon/assets/147078093/4db721f6-7ee4-4e1a-8aa3-d2606c7af292)

#### Explanation
  -  The above code uses concat to concatenate three cells (brand1, brand2, and year) into one (pair_id)
  -  I then use a **CASE** statement to ensure that the pair_id are identical and unique
  -  Instead of having *AppleSamsung2020* and *SamsungApple2020*, the **CASE** statement compares the VARCHAR
  -  If they condition if **True** it gives AppleSamsung2020
  -  If **False**, it rearranges and uses brand2 first, then brand1.

### Second step is to use a common table expression (cte)

```sql
WITH CTE AS 
			(SELECT *, CASE WHEN brand1 < brand2 THEN concat(brand1, brand2, YEAR)
					ELSE concat(brand2, brand1, YEAR)
			END AS pair_id
			FROM tech_giant),
	cte_rn AS
			(SELECT *
			 , ROW_NUMBER() OVER(PARTITION BY pair_id ORDER BY pair_id) AS RNK
			 FROM CTE)
SELECT *
FROM cte_rn;
```

![3 query](https://github.com/leon-madara/sql_marathon/assets/147078093/472a1c70-a47c-49ec-a7cc-1a91b58fc06c)

#### Explanation for step 2
  -  The above code uses a cte to create a temporary table
  -  I then use a secondary cte (cte_rn) to create a row number using the **row number ()** code
  -  I then *partition* it by *pair_id*, and rank it using the same column giving us our rank

### The third step is to create a filter

```sql
WITH CTE AS 
			(SELECT *, CASE WHEN brand1 < brand2 THEN concat(brand1, brand2, YEAR)
					ELSE concat(brand2, brand1, YEAR)
			END AS pair_id
			FROM tech_giant),
	cte_rn AS
			(SELECT *
			 , ROW_NUMBER() OVER(PARTITION BY pair_id ORDER BY pair_id) AS RNK
			 FROM CTE)
SELECT *
FROM cte_rn
WHERE rnk = 1
	OR custom1 <> custom3 AND custom2 <> custom4;
```

#### Explanation for step 3
  -  The above code creates a filter using 2 conditions
  -  Condition 1 is to return values where **rnk = 1**
  -  However, if you leave it there, it will omit the record *"AppleSamsung2021"* with rnk 2
  -  The record *"AppleSamsung2021"* meets the criteria **custom1 != custom3** OR **custom2 != custom4**
  -  Thus, it must be included.
  -  To do so, I used the code ```WHERE rnk = 1	OR custom1 <> custom3 AND custom2 <> custom4``` to filter
  -  The outcome included everything.


![4 query](https://github.com/leon-madara/sql_marathon/assets/147078093/2c8aadde-e16f-4867-9633-f97492c9c383)



# Query #2
Generate a report to showcase the period of presence and absence for each employee as demonstrated in the expected output

The Input 

![amazonsql1](https://github.com/leon-madara/sql_marathon/assets/147078093/403d2719-1d2e-4145-b849-3e483a1e14e9)

The Expected Output

![amazonsql2](https://github.com/leon-madara/sql_marathon/assets/147078093/05cd4f90-b1fb-4295-8eb3-c5eeea3ce4bd)

## The first step
to create the table and insert the values

```sql
CREATE TABLE attendance (
    employee_id VARCHAR(3),
    input_date DATE,
    status VARCHAR(7)
);


INSERT INTO attendance (employee_id, input_date, status) VALUES
('A1', '2024-01-01', 'PRESENT'),
('A1', '2024-01-02', 'PRESENT'),
('A1', '2024-01-03', 'PRESENT'),
('A1', '2024-01-04', 'PRESENT'),
('A1', '2024-01-05', 'ABSENT'),
('A1', '2024-01-06', 'PRESENT'),
('A1', '2024-01-07', 'ABSENT'),
('A1', '2024-01-08', 'PRESENT'),
('A2', '2024-01-06', 'PRESENT'),
('A2', '2024-01-07', 'PRESENT'),
('A2', '2024-01-08', 'PRESENT'),
('A2', '2024-01-10', 'ABSENT');
```

## The final code

To create a window function (cte) with row number

```sql
WITH cte AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY input_date) AS rn
    FROM attendance
),
cte_present AS (
    SELECT *,
           rn - ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY input_date) AS flag
    FROM cte
    WHERE status = 'PRESENT'
),
cte_absent AS (
    SELECT *,
           rn - ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY input_date) AS flag
    FROM cte
    WHERE status = 'ABSENT'
)
SELECT employee_id,
       FIRST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date) AS FROM_DATE,
       LAST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date
           RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS TO_DATE,
       status
FROM cte_present
UNION
SELECT employee_id,
       FIRST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date) AS FROM_DATE,
       LAST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date
           RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS TO_DATE,
       status
FROM cte_absent;
```

###
1. First code

```sql
WITH cte AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY input_date) AS rn
    FROM attendance
),
```

This code uses a window function (CTE) to create a row number to rank the output

2. Second part of the code: Present Values

```sql
cte_present AS (
    SELECT *,
           rn - ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY input_date) AS flag
    FROM cte
    WHERE status = 'PRESENT'
),
```

It creates another window function (cte_present) over the newly selected one WHERE status is PRESENT

3. The third part of the code: Absent Values

```sql
cte_absent AS (
    SELECT *,
           rn - ROW_NUMBER() OVER (PARTITION BY employee_id ORDER BY input_date) AS flag
    FROM cte
    WHERE status = 'ABSENT'
)
```

It uses the first_value and last_value, including the *CLAUSE* **RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING** to 

3. It uses a **UNION** CLAUSE to merge the output from the two window functions

```sql
SELECT employee_id,
       FIRST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date) AS FROM_DATE,
       LAST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date
           RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS TO_DATE,
       status
FROM cte_present
UNION
SELECT employee_id,
       FIRST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date) AS FROM_DATE,
       LAST_VALUE(input_date) OVER (PARTITION BY employee_id, flag ORDER BY input_date
           RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS TO_DATE,
       status
FROM cte_absent
```
End






