# Solving-a-Homicide-Case-with-SQL-A-Data-Analytics-Investigation-2026
This project is a SQL-driven crime investigation based on the open-source SQL Murder Mystery dataset.
# 🔎 SQL City Homicide Case Investigation

![SQL](https://img.shields.io/badge/SQL-Analysis-blue)
![Database](https://img.shields.io/badge/Database-Relational-orange)
![Project Type](https://img.shields.io/badge/Project-Data%20Analytics-green)

---

## 📌 Project Overview

This project uses **Structured Query Language (SQL)** to investigate a fictional homicide that occurred in **SQL City on January 15, 2018**.

The investigation begins with a single record in a `crime_scene_report` table and progressively connects information from witness interviews, gym membership records, gym check-ins, driver's licenses, and person records.

The objective was to demonstrate how SQL can be used to solve a multi-layered analytical problem through **filtering, pattern matching, table creation, joins, and multi-table reasoning**.

The project is based on the open-source **SQL Murder Mystery** dataset developed by Knight Lab, Northwestern University.

---

## 🎯 Objectives

The main objectives of this project were to:

* Identify the witnesses connected to the crime.
* Extract useful information from witness interviews.
* Narrow down suspects using a partial gym membership ID.
* Cross-reference gym attendance records.
* Use a partial vehicle license plate to identify the shooter.
* Investigate the shooter's statement to identify the mastermind.
* Demonstrate practical SQL techniques used in relational data analysis.

---

## 🗃️ Dataset & Tables

The investigation used several interconnected relational tables:

| Table                  | Purpose                                          |
| ---------------------- | ------------------------------------------------ |
| `crime_scene_report`   | Establishes the crime date, city, and crime type |
| `person`               | Contains information about residents             |
| `interview`            | Stores witness and suspect statements            |
| `get_fit_now_member`   | Contains gym membership information              |
| `get_fit_now_check_in` | Records gym attendance                           |
| `drivers_license`      | Contains vehicle and physical-description data   |

Two intermediate tables were also created during the investigation:

* `suspects` — filtered gold gym members whose IDs matched the witness clue.
* `full_data` — combines driver's license and person information using a `LEFT JOIN`.

---

## 🔍 Investigation Workflow

### 1. Establishing the Crime

The investigation began by locating the murder recorded in SQL City on January 15, 2018.

```sql
SELECT *
FROM crime_scene_report
WHERE date = 20180115
  AND city = 'SQL City'
  AND crime_type = 'murder';
```

---

### 2. Identifying the Witnesses

The witness information was obtained using address and name-based filtering.

```sql
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number DESC;
```

The second witness was located using a partial name match:

```sql
SELECT *
FROM person
WHERE name LIKE 'Annabel%'
  AND address_street_name = 'Franklin Ave';
```

The witnesses identified were:

* **Morty Schapiro**
* **Annabel Miller**

Their interviews provided the key clues that drove the rest of the investigation.

---

### 3. Filtering Potential Suspects

Morty reported that the suspect had a **Get Fit Now Gym bag** and that the membership number began with `48Z`. He also indicated that the suspect was a gold member.

This clue was converted into a SQL filter:

```sql
CREATE TABLE suspects AS
SELECT *
FROM get_fit_now_member
WHERE id LIKE '48%'
  AND membership_status = 'gold';
```

This narrowed the investigation to:

* **Joe Germuska**
* **Jeremy Bowers**

---

### 4. Cross-Referencing Gym Check-ins

Annabel stated that she recognized the killer at the gym on **January 9, 2018**.

The two suspects were therefore cross-referenced against gym attendance records:

```sql
SELECT *
FROM get_fit_now_check_in
WHERE check_in_date = 20170109
  AND membership_id IN ('4D5R1', '4KB72');
```

Both suspects had checked in on the relevant date, meaning the gym information alone was insufficient to identify the shooter.

A second independent clue was required.

---

### 5. Searching the Partial License Plate

Morty also remembered that the suspect's vehicle had a license plate containing:

> `H42W`

The driver's license table was searched using SQL pattern matching:

```sql
SELECT *
FROM drivers_license
WHERE plate_number LIKE 'H42W%'
   OR plate_number LIKE '%H42W%'
   OR plate_number LIKE '%H42W';
```

Because the driver's license records did not directly contain the suspect's name, the table was joined with the `person` table.

```sql
CREATE TABLE full_data AS
SELECT
    dl.age,
    dl.height,
    dl.hair_color,
    dl.gender,
    dl.plate_number,
    dl.car_make,
    dl.car_model,
    p.name,
    p.ssn,
    p.address_street_name,
    p.id
FROM drivers_license AS dl
LEFT JOIN person AS p
    ON dl.id = p.license_id;
```

The intersection of the gym-membership and license-plate clues identified:

### 🚨 Jeremy Bowers — The Shooter

---

## 🧩 Identifying the Mastermind

The investigation did not end with identifying the shooter.

Jeremy Bowers' interview revealed that he had been hired by a wealthy woman. His testimony provided several additional characteristics:

* Female
* Height between **65 and 67 inches**
* Red hair
* Drives a **Tesla Model S**
* Attended the **SQL Symphony Concert three times in December 2017**

These characteristics were converted into compound SQL filters:

```sql
SELECT *
FROM full_data
WHERE height BETWEEN 65 AND 67
  AND hair_color = 'red'
  AND gender = 'female'
  AND car_make = 'Tesla'
  AND car_model = 'Model S';
```

The compound filtering isolated the woman who hired Jeremy Bowers.

---

## 💡 Key Insights

### 1. Combining Partial Clues Increases Accuracy

A single clue was not enough to identify the shooter.

The combination of:

```text
Gym Membership ID
        ↓
Gym Check-in Date
        ↓
Partial License Plate
        ↓
Person Record
```

allowed the investigation to progressively narrow the suspect pool.

---

### 2. Intermediate Tables Improve Reproducibility

Creating the `suspects` and `full_data` tables made the investigation easier to audit and reproduce.

Instead of repeatedly writing complex joins and filters, intermediate results could be reused throughout the analysis.

---

### 3. SQL Can Handle Complex Investigative Reasoning

The project demonstrates that relatively simple SQL operations can answer complex questions when the underlying data is relationally structured.

Important techniques included:

* `SELECT`
* `WHERE`
* `LIKE`
* `BETWEEN`
* `ORDER BY`
* `CREATE TABLE AS`
* `LEFT JOIN`
* Multi-condition filtering

---

### 4. Red Herrings Require Corroboration

Both initial gym suspects had checked in on the relevant date.

Rather than stopping at the first apparent match, the investigation required another independent clue — the partial license plate — to break the tie.

---

## 📊 Data Visualization

Although the investigation was primarily SQL-based, visualization was used to communicate the analytical process to non-technical audiences.

The project included:

* **Suspect Pool Narrowing Visualization**
* **Investigation Timeline**

The visualizations were generated from counts and dates established during the SQL investigation.

---

## 🛠️ Tools & Technologies

| Tool / Technology        | Application                                         |
| ------------------------ | --------------------------------------------------- |
| **SQL**                  | Data querying and investigation                     |
| **Relational Database**  | Connecting investigative records                    |
| **SQL Pattern Matching** | Searching partial membership IDs and license plates |
| **SQL JOINs**            | Connecting identity and vehicle records             |
| **CREATE TABLE AS**      | Creating reusable intermediate datasets             |
| **Data Visualization**   | Communicating investigative findings                |

---

## 📈 Skills Demonstrated

* SQL Querying
* Data Filtering
* Pattern Matching with `LIKE`
* Relational Data Analysis
* Multi-table Joins
* Data Investigation
* Analytical Reasoning
* Data Reconciliation
* Intermediate Table Creation
* Query Structuring
* Data Storytelling

---

## 🏆 Results

The investigation successfully resolved the case through a sequence of SQL-based analytical steps.

### Shooter Identified

**Jeremy Bowers**

He was identified by combining:

* Gold gym membership
* Membership ID pattern
* January 9 gym attendance
* Partial license plate `H42W`

### Mastermind Identified

The person who hired Jeremy Bowers was identified using a compound filter based on:

* Gender
* Height
* Hair color
* Vehicle make
* Vehicle model
* Concert attendance information

---

## 📁 Suggested Repository Structure

```text
sql-city-homicide-investigation/
│
├── README.md
│
├── sql/
│   └── investigation.sql
│
├── data/
│   └── README.md
│
├── visualizations/
│   ├── suspect_pool.png
│   └── investigation_timeline.png
│
└── report/
    └── SQL_City_Homicide_Case_Report.pdf
```

---

## 📚 Dataset Reference

This project uses the **SQL Murder Mystery** dataset and case design adapted from the open-source project created by **Knight Lab, Northwestern University**.

---

## 👩‍💻 Project Takeaway

This project demonstrates how SQL can be used beyond basic querying to perform **structured investigative analysis**.

By progressively filtering records, matching partial information, joining related tables, and validating clues across independent datasets, the investigation moved from a single crime record to the identification of both the shooter and the person behind the crime.

The techniques demonstrated here are transferable to real-world applications such as:

* Fraud investigation
* Customer analytics
* Operational data reconciliation
* Identity resolution
* Data quality investigation

---

### ⭐ If you found this project useful, feel free to explore the repository and review the SQL investigation script.


