# Snowflake Tutorial Assignment — README

This document summarizes the steps executed and outputs produced while completing the Snowflake tutorial assignment using **SnowSQL v1.5.1**.

---

## 1. SnowSQL Login and Connection

Connected using a config file:

```
snowsql --config E:\Snowflake\.snowsql\config -c my_connection
```

Verified the session with:

```sql
SELECT CURRENT_USER()      AS current_user,
       CURRENT_ROLE()      AS current_role,
       CURRENT_WAREHOUSE() AS current_warehouse,
       CURRENT_DATABASE()  AS current_database,
       CURRENT_SCHEMA()    AS current_schema;
```

**Output:**

| CURRENT_USER | CURRENT_ROLE | CURRENT_WAREHOUSE | CURRENT_DATABASE      | CURRENT_SCHEMA |
|---|---|---|---|---|
| PR58 | SYSADMIN | COMPUTE_WH | SNOWFLAKE_SAMPLE_DATA | NULL |

Connection confirmed successfully — 1 row returned in 0.149s.

---

## 2. Creation of Snowflake Objects

Switched context into project-specific objects:

```sql
CREATE OR REPLACE DATABASE TUTORIAL_DB;
CREATE OR REPLACE SCHEMA TUTORIAL_DB.TUTORIAL_SCHEMA;
CREATE OR REPLACE WAREHOUSE TUTORIAL_WH
  WAREHOUSE_SIZE = 'XSMALL'
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE;

USE WAREHOUSE TUTORIAL_WH;
USE DATABASE TUTORIAL_DB;
USE SCHEMA TUTORIAL_SCHEMA;
```

Created table and stage, then inserted sample records:

```sql
CREATE OR REPLACE TABLE EMPLOYEES (
    EMP_ID      INT,
    EMP_NAME    STRING,
    DEPARTMENT  STRING,
    SALARY      NUMBER(10,2),
    JOIN_DATE   DATE
);

CREATE OR REPLACE STAGE TUTORIAL_STAGE
  FILE_FORMAT = (TYPE = 'CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);

INSERT INTO EMPLOYEES (EMP_ID, EMP_NAME, DEPARTMENT, SALARY, JOIN_DATE) VALUES
(1, 'Arun Kumar',   'IT',      55000.00, '2022-01-15'),
(2, 'Priya Sharma',  'HR',      48000.00, '2021-11-01'),
(3, 'Rahul Verma',   'Finance', 62000.00, '2020-06-20'),
(4, 'Sneha Reddy',   'IT',      58000.00, '2023-03-10'),
(5, 'Karthik Iyer',  'Sales',   50000.00, '2022-09-05');

SELECT * FROM EMPLOYEES;
```

**Output:**
- Table `EMPLOYEES` successfully created.
- Stage `TUTORIAL_STAGE` successfully created.
- 5 rows inserted.

| EMP_ID | EMP_NAME | DEPARTMENT | SALARY | JOIN_DATE |
|---|---|---|---|---|
| 1 | Arun Kumar | IT | 55000.00 | 2022-01-15 |
| 2 | Priya Sharma | HR | 48000.00 | 2021-11-01 |
| 3 | Rahul Verma | Finance | 62000.00 | 2020-06-20 |
| 4 | Sneha Reddy | IT | 58000.00 | 2023-03-10 |
| 5 | Karthik Iyer | Sales | 50000.00 | 2022-09-05 |

### DML Operations — UPDATE and DELETE

```sql
UPDATE EMPLOYEES SET SALARY = SALARY * 1.10 WHERE DEPARTMENT = 'IT';
DELETE FROM EMPLOYEES WHERE EMP_ID = 5;
SELECT * FROM EMPLOYEES ORDER BY EMP_ID;
```

**Output:**
- 2 rows updated (IT department salary raised by 10%).
- 1 row deleted (EMP_ID = 5, Karthik Iyer).

| EMP_ID | EMP_NAME | DEPARTMENT | SALARY | JOIN_DATE |
|---|---|---|---|---|
| 1 | Arun Kumar | IT | 60500.00 | 2022-01-15 |
| 2 | Priya Sharma | HR | 48000.00 | 2021-11-01 |
| 3 | Rahul Verma | Finance | 62000.00 | 2020-06-20 |
| 4 | Sneha Reddy | IT | 63800.00 | 2023-03-10 |

---

## 3. Data Loading Using SnowSQL

Sample dataset created locally as `employees_data.csv` and loaded via `PUT` + `COPY INTO` into a staging table `EMPLOYEES_LOAD`, then verified with `SELECT COUNT(*)` and `SELECT *`. See the full command set in the main tutorial document (`Snowflake_Tutorial_Assignment.md`).

---

## 4. Snowflake Time Travel

Queried past table state using an `OFFSET`-based Time Travel query, filtering on `DEPARTMENT = 'HR'`:

```sql
SELECT * FROM EMPLOYEES AT(OFFSET => -120) WHERE DEPARTMENT = 'HR';
```

**Output:**

| EMP_ID | EMP_NAME | DEPARTMENT | SALARY | JOIN_DATE |
|---|---|---|---|---|
| 2 | Priya Sharma | HR | 48000.00 | 2021-11-01 |

Also demonstrated Time Travel with a date-based filter, querying employees who joined before 2022 as the table existed 2 minutes prior:

```sql
SELECT * FROM EMPLOYEES AT(OFFSET => -120) WHERE JOIN_DATE < '2022-01-01';
```

**Output:**

| EMP_ID | EMP_NAME | DEPARTMENT | SALARY | JOIN_DATE |
|---|---|---|---|---|
| 2 | Priya Sharma | HR | 48000.00 | 2021-11-01 |
| 3 | Rahul Verma | Finance | 62000.00 | 2020-06-20 |

---

## 5. Data Recovery Using Time Travel

**Scenario:** HR department record accidentally deleted, then recovered.

```sql
-- 1. Accidentally delete the HR records
DELETE FROM EMPLOYEES WHERE DEPARTMENT = 'HR';

-- 2. Confirm they're gone
SELECT * FROM EMPLOYEES WHERE DEPARTMENT = 'HR';   -- returns 0 rows

-- 3. Check table state as it existed 2 minutes ago
SELECT * FROM EMPLOYEES AT(OFFSET => -120) WHERE DEPARTMENT = 'HR';

-- 4. Restore the missing HR records back into the table
INSERT INTO EMPLOYEES
SELECT * FROM EMPLOYEES AT(OFFSET => -120) WHERE DEPARTMENT = 'HR';

-- 5. Final verification: verify all records are restored
SELECT * FROM EMPLOYEES ORDER BY EMP_ID;
```

This confirms the recovered HR record (Priya Sharma, EMP_ID 2) is restored into the live table using Snowflake Time Travel, without needing a backup or external export.

---

## Environment Summary

| Item | Value |
|---|---|
| SnowSQL version | v1.5.1 |
| User | PR58 |
| Role | SYSADMIN |
| Warehouse | TUTORIAL_WH (XSMALL, auto-suspend 60s) |
| Database | TUTORIAL_DB |
| Schema | TUTORIAL_SCHEMA |
| Table | EMPLOYEES |
| Stage | TUTORIAL_STAGE |

## Files in this Submission

- `README.md` — this summary
- `Snowflake_Tutorial_Assignment.md` — full command reference for all 5 sections
- Screenshots — connection verification, object creation, DML, Time Travel, and recovery query outputs

