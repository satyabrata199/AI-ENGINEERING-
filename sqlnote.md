Perfect. Since you've completed **Phase 1 (CRUD)**, we'll move to **Phase 2: Schema Management (DDL)**.

---

# Phase 2: Schema Management (DDL)

**DDL (Data Definition Language)** is used to **create and modify the structure (schema)** of database objects like tables, columns, indexes, and databases.

Unlike CRUD, which works with **rows (records)**, DDL works with the **table definition**.

## Topics We'll Cover

| Step | Topic                   | Command                                          |
| ---- | ----------------------- | ------------------------------------------------ |
| ✅ 1  | Create Table            | `CREATE TABLE` *(already done)*                  |
| 2    | Add Columns             | `ALTER TABLE ... ADD COLUMN`                     |
| 3    | Rename Columns          | `ALTER TABLE ... RENAME COLUMN`                  |
| 4    | Modify Column Data Type | `ALTER TABLE ... ALTER COLUMN` / `MODIFY COLUMN` |
| 5    | Add Constraints         | `ALTER TABLE ... ADD CONSTRAINT`                 |
| 6    | Drop Constraints        | `ALTER TABLE ... DROP CONSTRAINT`                |
| 7    | Drop Columns            | `ALTER TABLE ... DROP COLUMN`                    |
| 8    | Rename Tables           | `ALTER TABLE ... RENAME TO`                      |
| 9    | Truncate Tables         | `TRUNCATE TABLE`                                 |
| 10   | Drop Tables             | `DROP TABLE`                                     |
| 11   | Drop Databases          | `DROP DATABASE`                                  |

> **Note:** The exact syntax for some `ALTER TABLE` operations varies between database systems (MySQL, PostgreSQL, SQL Server, Oracle). I'll point out differences where relevant.

---

# Step 1: Add Columns

Sometimes you need to store new information after a table has already been created.

### Before

```text
Students
+-----------+--------+-----+------------+
| StudentID | Name   | Age | City       |
+-----------+--------+-----+------------+
```

Suppose you now want to store each student's email.

## Syntax

```sql
ALTER TABLE table_name
ADD COLUMN column_name datatype;
```

---

## Example

```sql
ALTER TABLE Students
ADD COLUMN Email VARCHAR(100);
```

---

### After

```text
Students
+-----------+--------+-----+------------+------------------+
| StudentID | Name   | Age | City       | Email            |
+-----------+--------+-----+------------+------------------+
| 1         | Rahul  | 20  | Hyderabad  | NULL             |
| 2         | Priya  | 21  | Delhi      | NULL             |
+-----------+--------+-----+------------+------------------+
```

Notice that existing rows get `NULL` for the new column unless you specify a default value.

---

## Add Multiple Columns

### MySQL

```sql
ALTER TABLE Students
ADD COLUMN Email VARCHAR(100),
ADD COLUMN Phone VARCHAR(15);
```

### PostgreSQL

```sql
ALTER TABLE Students
ADD COLUMN Email VARCHAR(100),
ADD COLUMN Phone VARCHAR(15);
```

---

## Add a Column with a Default Value

```sql
ALTER TABLE Students
ADD COLUMN Country VARCHAR(50) DEFAULT 'India';
```

Now every existing and new row (unless overridden) gets:

```text
Country = India
```

---

## Add a NOT NULL Column

```sql
ALTER TABLE Students
ADD COLUMN Department VARCHAR(50) NOT NULL DEFAULT 'CSE';
```

A default value is usually required when adding a `NOT NULL` column to a table that already contains data.

---

## Verify the Change

To view the updated table structure:

```sql
DESCRIBE Students;
```

or (depending on the database):

```sql
SHOW COLUMNS FROM Students;
```

---

# Practice Exercise

Assume this table already exists:

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(50),
    Age INT,
    City VARCHAR(50)
);
```

Write SQL statements to:

1. Add an `Email` column of type `VARCHAR(100)`.
2. Add a `Phone` column of type `VARCHAR(15)`.
3. Add a `Country` column with the default value `'India'`.

---

Once you're comfortable with **adding columns**, we'll move to **Step 2: Renaming Columns**, where you'll learn how to change column names without losing the data stored in them.


Great! 🎉 You've completed **Step 1: Add Columns**.

Now let's move to the next topic.

---

# Step 2: Rename Columns

Sometimes a column name is unclear or needs to follow a new naming convention.

For example:

Before:

```text
Students
+-----------+--------+-----+------------+
| StudentID | Name   | Age | City       |
+-----------+--------+-----+------------+
```

Suppose you want `Name` to become `FullName`.

---

# Syntax

The syntax depends on the database you're using.

### PostgreSQL

```sql
ALTER TABLE table_name
RENAME COLUMN old_column_name TO new_column_name;
```

### MySQL (8.0+)

```sql
ALTER TABLE table_name
RENAME COLUMN old_column_name TO new_column_name;
```

### SQL Server

```sql
EXEC sp_rename 'Students.Name', 'FullName', 'COLUMN';
```

For this course, we'll use the PostgreSQL/MySQL syntax since it's simple and widely used.

---

# Example

```sql
ALTER TABLE Students
RENAME COLUMN Name TO FullName;
```

---

### Before

```text
StudentID | Name | Age | City
```

### After

```text
StudentID | FullName | Age | City
```

Notice:

* ✅ The **data remains unchanged**
* ✅ Only the **column name changes**
* ❌ No rows are affected

---

## Rename Another Column

Suppose you want:

```text
City
```

to become

```text
Location
```

```sql
ALTER TABLE Students
RENAME COLUMN City TO Location;
```

---

### Result

```text
StudentID | FullName | Age | Location
```

---

# Verify the Changes

To view the updated structure:

```sql
DESCRIBE Students;
```

or

```sql
SHOW COLUMNS FROM Students;
```

---

# Important Notes

✔ Only the column name changes.

✔ The datatype remains the same.

✔ Existing records are preserved.

✔ Constraints (like `PRIMARY KEY` or `NOT NULL`) remain attached to the renamed column.

---

# Practice

Assume your table looks like this:

```text
StudentID | Name | Age | City | Email | Phone
```

Rename:

1. `Name` → `FullName`
2. `City` → `Location`
3. `Phone` → `MobileNumber`

```sql
ALTER TABLE Students
RENAME COLUMN Name TO FullName;

ALTER TABLE Students
RENAME COLUMN City TO Location;

ALTER TABLE Students
RENAME COLUMN Phone TO MobileNumber;
```

---

## Progress

* ✅ Create Table
* ✅ CRUD Operations
* ✅ Add Columns
* ✅ Rename Columns
* ⏳ **Next:** Modify Column Data Types (`ALTER COLUMN` / `MODIFY COLUMN`)
* ⏳ Add Constraints
* ⏳ Drop Constraints
* ⏳ Drop Columns
* ⏳ Rename Tables
* ⏳ TRUNCATE TABLE
* ⏳ DROP TABLE
* ⏳ DROP DATABASE

Once you've practiced renaming columns, we'll move on to **Step 3: Modifying Column Data Types**, where you'll learn how to change a column's datatype (for example, changing `Age` from `INT` to `BIGINT` or increasing the length of a `VARCHAR`).




Excellent! 🎉 Now you've learned how to **add** and **rename** columns.

Let's continue with one of the most common schema changes.

---

# Step 3: Modify Column Data Type

Sometimes the original datatype is no longer sufficient.

For example:

* `VARCHAR(50)` → `VARCHAR(100)`
* `INT` → `BIGINT`
* `DECIMAL(5,2)` → `DECIMAL(10,2)`

Instead of creating a new column, you can modify the existing one.

---

# Example Table

```text
Students
+-----------+----------+-----+------------+------------------+
| StudentID | FullName | Age | Location   | Email            |
+-----------+----------+-----+------------+------------------+
```

Suppose you want to:

* Increase the size of the `Email` column
* Change `Age` to `BIGINT`

---

# Syntax

The syntax differs slightly between database systems.

### MySQL

```sql
ALTER TABLE table_name
MODIFY COLUMN column_name new_datatype;
```

### PostgreSQL

```sql
ALTER TABLE table_name
ALTER COLUMN column_name TYPE new_datatype;
```

We'll show both where appropriate.

---

# Example 1: Increase VARCHAR Size

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN Email VARCHAR(200);
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN Email TYPE VARCHAR(200);
```

---

### Before

```text
Email VARCHAR(100)
```

### After

```text
Email VARCHAR(200)
```

No data is lost because the column is simply allowed to store longer values.

---

# Example 2: Change INT to BIGINT

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN Age BIGINT;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN Age TYPE BIGINT;
```

---

### Before

```text
Age INT
```

### After

```text
Age BIGINT
```

This allows much larger numbers to be stored.

---

# Example 3: Change VARCHAR to TEXT

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN Address TEXT;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN Address TYPE TEXT;
```

Useful when you need to store long descriptions or addresses.

---

# Things to Remember

✅ Increasing a `VARCHAR` size is generally safe.

Example:

```text
VARCHAR(50) → VARCHAR(100)
```

---

⚠ Decreasing a `VARCHAR` size may fail if existing values are longer than the new limit.

Example:

```text
VARCHAR(200) → VARCHAR(20)
```

If a row already contains 50 characters, the database may reject the change.

---

⚠ Changing incompatible types can fail.

Example:

```text
VARCHAR → INT
```

If the column contains text like:

```text
Rahul
Priya
Amit
```

the database cannot convert those values into integers.

---

# Verify the Changes

To check the updated schema:

```sql
DESCRIBE Students;
```

or

```sql
SHOW COLUMNS FROM Students;
```

(PostgreSQL users commonly use `\d Students` in `psql`.)

---

# Practice

Starting with:

```text
StudentID INT
FullName VARCHAR(50)
Age INT
Location VARCHAR(50)
Email VARCHAR(100)
```

Modify:

1. `FullName` → `VARCHAR(100)`
2. `Email` → `VARCHAR(255)`
3. `Age` → `BIGINT`

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN FullName VARCHAR(100);

ALTER TABLE Students
MODIFY COLUMN Email VARCHAR(255);

ALTER TABLE Students
MODIFY COLUMN Age BIGINT;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN FullName TYPE VARCHAR(100);

ALTER TABLE Students
ALTER COLUMN Email TYPE VARCHAR(255);

ALTER TABLE Students
ALTER COLUMN Age TYPE BIGINT;
```

---

## Progress Tracker

* ✅ CRUD Operations
* ✅ Add Columns
* ✅ Rename Columns
* ✅ Modify Column Data Types
* ⏳ **Next:** Add Constraints (`PRIMARY KEY`, `UNIQUE`, `NOT NULL`, `CHECK`, `DEFAULT`, `FOREIGN KEY`)
* ⏳ Drop Constraints
* ⏳ Drop Columns
* ⏳ Rename Tables
* ⏳ TRUNCATE TABLE
* ⏳ DROP TABLE
* ⏳ DROP DATABASE

The next topic, **constraints**, is especially important because it controls what data is allowed into your tables and is a key part of designing reliable databases.


Awesome! 🚀 You're making good progress.

Now we'll move to one of the most important topics in database design.

---

# Step 4: Add Constraints

## What are Constraints?

Constraints are **rules** applied to columns to ensure the data stored in the table is **accurate, valid, and consistent**.

Think of them as the database's way of saying:

> "Only data that follows these rules is allowed."

---

## Common Constraints

| Constraint    | Purpose                                     |
| ------------- | ------------------------------------------- |
| `NOT NULL`    | Value cannot be empty                       |
| `DEFAULT`     | Assigns a default value if none is provided |
| `UNIQUE`      | Prevents duplicate values                   |
| `PRIMARY KEY` | Uniquely identifies each row                |
| `FOREIGN KEY` | Links two tables together                   |
| `CHECK`       | Ensures values meet a condition             |

---

We'll learn them one by one.

---

# 1. NOT NULL

A `NOT NULL` constraint ensures that every row must have a value in that column.

### Before

```sql
CREATE TABLE Students (
    StudentID INT,
    FullName VARCHAR(100)
);
```

Both columns can contain `NULL`.

---

## Add NOT NULL

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN FullName VARCHAR(100) NOT NULL;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN FullName SET NOT NULL;
```

---

### Valid

```sql
INSERT INTO Students
VALUES (1, 'Rahul');
```

---

### Invalid

```sql
INSERT INTO Students
VALUES (2, NULL);
```

**Error:**

```
FullName cannot be NULL.
```

---

# 2. DEFAULT

A default value is automatically inserted when no value is provided.

---

## Example

```sql
ALTER TABLE Students
ADD COLUMN Country VARCHAR(50) DEFAULT 'India';
```

---

### Insert

```sql
INSERT INTO Students (StudentID, FullName)
VALUES (3, 'Priya');
```

---

### Result

| StudentID | FullName | Country |
| --------- | -------- | ------- |
| 3         | Priya    | India   |

You didn't specify `Country`, so the database used the default value.

---

# 3. UNIQUE

Ensures that no two rows have the same value.

Example: Email addresses.

---

```sql
ALTER TABLE Students
ADD CONSTRAINT uq_email
UNIQUE (Email);
```

---

### Valid

```sql
Email = 'rahul@gmail.com'
Email = 'priya@gmail.com'
```

---

### Invalid

```sql
Email = 'rahul@gmail.com'
Email = 'rahul@gmail.com'
```

Error:

```
Duplicate value violates UNIQUE constraint.
```

---

# 4. PRIMARY KEY

A Primary Key uniquely identifies every row.

Rules:

* Cannot be `NULL`
* Cannot be duplicated
* Only one primary key per table (it may consist of multiple columns)

---

```sql
ALTER TABLE Students
ADD CONSTRAINT pk_students
PRIMARY KEY (StudentID);
```

---

### Valid

```text
1
2
3
4
```

---

### Invalid

```text
1
1
2
```

Duplicate primary keys are not allowed.

---

# 5. CHECK

Restricts values based on a condition.

Example:

Age must be at least 18.

```sql
ALTER TABLE Students
ADD CONSTRAINT chk_age
CHECK (Age >= 18);
```

---

### Valid

```text
Age = 20
Age = 25
```

---

### Invalid

```text
Age = 15
```

The database rejects the row.

---

# 6. FOREIGN KEY

A Foreign Key connects two tables.

Suppose:

### Departments

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |

---

### Students

| StudentID | FullName | DepartmentID |
| --------- | -------- | ------------ |
| 1         | Rahul    | 1            |
| 2         | Priya    | 2            |

`DepartmentID` in `Students` refers to `DepartmentID` in `Departments`.

---

```sql
ALTER TABLE Students
ADD CONSTRAINT fk_department
FOREIGN KEY (DepartmentID)
REFERENCES Departments(DepartmentID);
```

---

Now this is allowed:

```text
DepartmentID = 1
DepartmentID = 2
```

But this is rejected:

```text
DepartmentID = 10
```

because there is no department with ID 10.

---

# Constraint Summary

| Constraint    | Prevents                          |
| ------------- | --------------------------------- |
| `NOT NULL`    | Empty values                      |
| `DEFAULT`     | Missing values                    |
| `UNIQUE`      | Duplicate values                  |
| `PRIMARY KEY` | Duplicate or NULL identifiers     |
| `CHECK`       | Invalid values                    |
| `FOREIGN KEY` | Invalid references between tables |

---

# Practice

Assume this table:

```sql
CREATE TABLE Students (
    StudentID INT,
    FullName VARCHAR(100),
    Age INT,
    Email VARCHAR(255),
    DepartmentID INT
);
```

Add these constraints:

1. `FullName` → `NOT NULL`
2. `Email` → `UNIQUE`
3. `StudentID` → `PRIMARY KEY`
4. `Age` → `CHECK (Age >= 18)`
5. `DepartmentID` → `FOREIGN KEY` referencing `Departments(DepartmentID)`

---

## Progress

* ✅ CRUD (DML)
* ✅ Add Columns
* ✅ Rename Columns
* ✅ Modify Data Types
* ✅ Add Constraints
* ⏳ **Next:** Drop Constraints
* ⏳ Drop Columns
* ⏳ Rename Tables
* ⏳ TRUNCATE TABLE
* ⏳ DROP TABLE
* ⏳ DROP DATABASE

**Recommendation:** Spend some time experimenting with each constraint by inserting both valid and invalid data. Seeing the database accept or reject rows will make these concepts much easier to remember before moving on to dropping constraints.


Excellent! 🎉 At this point, you've learned how to **create** constraints. Now let's learn how to **remove** them.

---

# Step 5: Drop Constraints

Sometimes business rules change.

Examples:

* Email no longer needs to be unique.
* Age restriction changes.
* A foreign key relationship is no longer required.

In these cases, we **drop** the constraint.

---

# Before You Drop a Constraint

You need to know its **name**.

For example, if you created:

```sql
ALTER TABLE Students
ADD CONSTRAINT uq_email
UNIQUE (Email);
```

Then the constraint name is:

```text
uq_email
```

Similarly:

```text
pk_students
chk_age
fk_department
```

---

# 1. Drop a PRIMARY KEY

### MySQL

```sql
ALTER TABLE Students
DROP PRIMARY KEY;
```

### PostgreSQL

```sql
ALTER TABLE Students
DROP CONSTRAINT pk_students;
```

---

# 2. Drop a UNIQUE Constraint

Suppose:

```sql
ALTER TABLE Students
ADD CONSTRAINT uq_email
UNIQUE (Email);
```

Remove it:

### MySQL & PostgreSQL

```sql
ALTER TABLE Students
DROP CONSTRAINT uq_email;
```

> **Note:** In MySQL, depending on the version, a `UNIQUE` constraint may be backed by an index. In some cases you may use `DROP INDEX`. Always check your database documentation.

---

# 3. Drop a CHECK Constraint

Created as:

```sql
ALTER TABLE Students
ADD CONSTRAINT chk_age
CHECK (Age >= 18);
```

Remove it:

```sql
ALTER TABLE Students
DROP CONSTRAINT chk_age;
```

---

# 4. Drop a FOREIGN KEY

Created as:

```sql
ALTER TABLE Students
ADD CONSTRAINT fk_department
FOREIGN KEY (DepartmentID)
REFERENCES Departments(DepartmentID);
```

### MySQL

```sql
ALTER TABLE Students
DROP FOREIGN KEY fk_department;
```

### PostgreSQL

```sql
ALTER TABLE Students
DROP CONSTRAINT fk_department;
```

---

# 5. Remove NOT NULL

Unlike other constraints, `NOT NULL` is part of the column definition.

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN FullName VARCHAR(100) NULL;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN FullName DROP NOT NULL;
```

Now `NULL` values are allowed again.

---

# How to View Existing Constraints

The exact command depends on the database.

### MySQL

```sql
SHOW CREATE TABLE Students;
```

### PostgreSQL (psql)

```sql
\d Students
```

You can also query the `information_schema` views in both databases.

---

# Practice

Assume the table has these constraints:

* `pk_students`
* `uq_email`
* `chk_age`
* `fk_department`

Write SQL to:

1. Remove the `UNIQUE` constraint on `Email`.
2. Remove the `CHECK` constraint on `Age`.
3. Remove the foreign key on `DepartmentID`.
4. Allow `FullName` to be `NULL`.

---

# Progress Tracker

| Topic               | Status                |
| ------------------- | --------------------- |
| ✅ CRUD (DML)        | Done                  |
| ✅ Add Columns       | Done                  |
| ✅ Rename Columns    | Done                  |
| ✅ Modify Data Types | Done                  |
| ✅ Add Constraints   | Done                  |
| ✅ Drop Constraints  | Done (after practice) |
| ⏳ Drop Columns      | Next                  |
| ⏳ Rename Tables     | Pending               |
| ⏳ TRUNCATE TABLE    | Pending               |
| ⏳ DROP TABLE        | Pending               |
| ⏳ DROP DATABASE     | Pending               |



Awesome! 🚀 You're making good progress.

Now we'll move to one of the most important topics in database design.

---

# Step 4: Add Constraints

## What are Constraints?

Constraints are **rules** applied to columns to ensure the data stored in the table is **accurate, valid, and consistent**.

Think of them as the database's way of saying:

> "Only data that follows these rules is allowed."

---

## Common Constraints

| Constraint    | Purpose                                     |
| ------------- | ------------------------------------------- |
| `NOT NULL`    | Value cannot be empty                       |
| `DEFAULT`     | Assigns a default value if none is provided |
| `UNIQUE`      | Prevents duplicate values                   |
| `PRIMARY KEY` | Uniquely identifies each row                |
| `FOREIGN KEY` | Links two tables together                   |
| `CHECK`       | Ensures values meet a condition             |

---

We'll learn them one by one.

---

# 1. NOT NULL

A `NOT NULL` constraint ensures that every row must have a value in that column.

### Before

```sql
CREATE TABLE Students (
    StudentID INT,
    FullName VARCHAR(100)
);
```

Both columns can contain `NULL`.

---

## Add NOT NULL

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN FullName VARCHAR(100) NOT NULL;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN FullName SET NOT NULL;
```

---

### Valid

```sql
INSERT INTO Students
VALUES (1, 'Rahul');
```

---

### Invalid

```sql
INSERT INTO Students
VALUES (2, NULL);
```

**Error:**

```
FullName cannot be NULL.
```

---

# 2. DEFAULT

A default value is automatically inserted when no value is provided.

---

## Example

```sql
ALTER TABLE Students
ADD COLUMN Country VARCHAR(50) DEFAULT 'India';
```

---

### Insert

```sql
INSERT INTO Students (StudentID, FullName)
VALUES (3, 'Priya');
```

---

### Result

| StudentID | FullName | Country |
| --------- | -------- | ------- |
| 3         | Priya    | India   |

You didn't specify `Country`, so the database used the default value.

---

# 3. UNIQUE

Ensures that no two rows have the same value.

Example: Email addresses.

---

```sql
ALTER TABLE Students
ADD CONSTRAINT uq_email
UNIQUE (Email);
```

---

### Valid

```sql
Email = 'rahul@gmail.com'
Email = 'priya@gmail.com'
```

---

### Invalid

```sql
Email = 'rahul@gmail.com'
Email = 'rahul@gmail.com'
```

Error:

```
Duplicate value violates UNIQUE constraint.
```

---

# 4. PRIMARY KEY

A Primary Key uniquely identifies every row.

Rules:

* Cannot be `NULL`
* Cannot be duplicated
* Only one primary key per table (it may consist of multiple columns)

---

```sql
ALTER TABLE Students
ADD CONSTRAINT pk_students
PRIMARY KEY (StudentID);
```

---

### Valid

```text
1
2
3
4
```

---

### Invalid

```text
1
1
2
```

Duplicate primary keys are not allowed.

---

# 5. CHECK

Restricts values based on a condition.

Example:

Age must be at least 18.

```sql
ALTER TABLE Students
ADD CONSTRAINT chk_age
CHECK (Age >= 18);
```

---

### Valid

```text
Age = 20
Age = 25
```

---

### Invalid

```text
Age = 15
```

The database rejects the row.

---

# 6. FOREIGN KEY

A Foreign Key connects two tables.

Suppose:

### Departments

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |

---

### Students

| StudentID | FullName | DepartmentID |
| --------- | -------- | ------------ |
| 1         | Rahul    | 1            |
| 2         | Priya    | 2            |

`DepartmentID` in `Students` refers to `DepartmentID` in `Departments`.

---

```sql
ALTER TABLE Students
ADD CONSTRAINT fk_department
FOREIGN KEY (DepartmentID)
REFERENCES Departments(DepartmentID);
```

---

Now this is allowed:

```text
DepartmentID = 1
DepartmentID = 2
```

But this is rejected:

```text
DepartmentID = 10
```

because there is no department with ID 10.

---

# Constraint Summary

| Constraint    | Prevents                          |
| ------------- | --------------------------------- |
| `NOT NULL`    | Empty values                      |
| `DEFAULT`     | Missing values                    |
| `UNIQUE`      | Duplicate values                  |
| `PRIMARY KEY` | Duplicate or NULL identifiers     |
| `CHECK`       | Invalid values                    |
| `FOREIGN KEY` | Invalid references between tables |

---

# Practice

Assume this table:

```sql
CREATE TABLE Students (
    StudentID INT,
    FullName VARCHAR(100),
    Age INT,
    Email VARCHAR(255),
    DepartmentID INT
);
```

Add these constraints:

1. `FullName` → `NOT NULL`
2. `Email` → `UNIQUE`
3. `StudentID` → `PRIMARY KEY`
4. `Age` → `CHECK (Age >= 18)`
5. `DepartmentID` → `FOREIGN KEY` referencing `Departments(DepartmentID)`

---

## Progress

* ✅ CRUD (DML)
* ✅ Add Columns
* ✅ Rename Columns
* ✅ Modify Data Types
* ✅ Add Constraints
* ⏳ **Next:** Drop Constraints
* ⏳ Drop Columns
* ⏳ Rename Tables
* ⏳ TRUNCATE TABLE
* ⏳ DROP TABLE
* ⏳ DROP DATABASE

**Recommendation:** Spend some time experimenting with each constraint by inserting both valid and invalid data. Seeing the database accept or reject rows will make these concepts much easier to remember before moving on to dropping constraints.


Excellent! 🎉 At this point, you've learned how to **create** constraints. Now let's learn how to **remove** them.

---

# Step 5: Drop Constraints

Sometimes business rules change.

Examples:

* Email no longer needs to be unique.
* Age restriction changes.
* A foreign key relationship is no longer required.

In these cases, we **drop** the constraint.

---

# Before You Drop a Constraint

You need to know its **name**.

For example, if you created:

```sql
ALTER TABLE Students
ADD CONSTRAINT uq_email
UNIQUE (Email);
```

Then the constraint name is:

```text
uq_email
```

Similarly:

```text
pk_students
chk_age
fk_department
```

---

# 1. Drop a PRIMARY KEY

### MySQL

```sql
ALTER TABLE Students
DROP PRIMARY KEY;
```

### PostgreSQL

```sql
ALTER TABLE Students
DROP CONSTRAINT pk_students;
```

---

# 2. Drop a UNIQUE Constraint

Suppose:

```sql
ALTER TABLE Students
ADD CONSTRAINT uq_email
UNIQUE (Email);
```

Remove it:

### MySQL & PostgreSQL

```sql
ALTER TABLE Students
DROP CONSTRAINT uq_email;
```

> **Note:** In MySQL, depending on the version, a `UNIQUE` constraint may be backed by an index. In some cases you may use `DROP INDEX`. Always check your database documentation.

---

# 3. Drop a CHECK Constraint

Created as:

```sql
ALTER TABLE Students
ADD CONSTRAINT chk_age
CHECK (Age >= 18);
```

Remove it:

```sql
ALTER TABLE Students
DROP CONSTRAINT chk_age;
```

---

# 4. Drop a FOREIGN KEY

Created as:

```sql
ALTER TABLE Students
ADD CONSTRAINT fk_department
FOREIGN KEY (DepartmentID)
REFERENCES Departments(DepartmentID);
```

### MySQL

```sql
ALTER TABLE Students
DROP FOREIGN KEY fk_department;
```

### PostgreSQL

```sql
ALTER TABLE Students
DROP CONSTRAINT fk_department;
```

---

# 5. Remove NOT NULL

Unlike other constraints, `NOT NULL` is part of the column definition.

### MySQL

```sql
ALTER TABLE Students
MODIFY COLUMN FullName VARCHAR(100) NULL;
```

### PostgreSQL

```sql
ALTER TABLE Students
ALTER COLUMN FullName DROP NOT NULL;
```

Now `NULL` values are allowed again.

---

# How to View Existing Constraints

The exact command depends on the database.

### MySQL

```sql
SHOW CREATE TABLE Students;
```

### PostgreSQL (psql)

```sql
\d Students
```

You can also query the `information_schema` views in both databases.

---

# Practice

Assume the table has these constraints:

* `pk_students`
* `uq_email`
* `chk_age`
* `fk_department`

Write SQL to:

1. Remove the `UNIQUE` constraint on `Email`.
2. Remove the `CHECK` constraint on `Age`.
3. Remove the foreign key on `DepartmentID`.
4. Allow `FullName` to be `NULL`.

---

# Progress Tracker

| Topic               | Status                |
| ------------------- | --------------------- |
| ✅ CRUD (DML)        | Done                  |
| ✅ Add Columns       | Done                  |
| ✅ Rename Columns    | Done                  |
| ✅ Modify Data Types | Done                  |
| ✅ Add Constraints   | Done                  |
| ✅ Drop Constraints  | Done (after practice) |
| ⏳ Drop Columns      | Next                  |
| ⏳ Rename Tables     | Pending               |
| ⏳ TRUNCATE TABLE    | Pending               |
| ⏳ DROP TABLE        | Pending               |
| ⏳ DROP DATABASE     | Pending               |

## Next Step: Drop Columns

You'll learn how to permanently remove one or more columns from a table while keeping the rest of the table and its data intact. This is another common schema change performed with `ALTER TABLE`.



Great! You're moving through DDL systematically. 👏

Now let's learn how to **remove columns** from a table.

---

# Step 6: Drop Columns

A **column** stores a specific attribute of your data.

Sometimes a column is no longer needed.

For example:

* `Phone` is no longer required.
* `Address` has been moved to another table.
* `MiddleName` was added by mistake.

Instead of dropping the whole table, you can remove just that column.

---

# Before

```text
Students
+-----------+----------+-----+----------+------------------+------------+
| StudentID | FullName | Age | Location | Email            | Phone      |
+-----------+----------+-----+----------+------------------+------------+
```

Suppose we no longer need the `Phone` column.

---

# Syntax

```sql
ALTER TABLE table_name
DROP COLUMN column_name;
```

---

# Example

```sql
ALTER TABLE Students
DROP COLUMN Phone;
```

---

# Before

```text
StudentID | FullName | Age | Location | Email | Phone
```

---

# After

```text
StudentID | FullName | Age | Location | Email
```

The **Phone** column and all of its data are permanently removed.

---

# Drop Multiple Columns

### MySQL & PostgreSQL

```sql
ALTER TABLE Students
DROP COLUMN Email,
DROP COLUMN Location;
```

---

### Before

```text
StudentID | FullName | Age | Location | Email
```

---

### After

```text
StudentID | FullName | Age
```

---

# Things to Remember

### ✅ Data is permanently deleted

If you drop a column:

```sql
ALTER TABLE Students
DROP COLUMN Email;
```

Everything stored in `Email` is lost.

---

### ⚠ Dependencies Matter

If the column is used by:

* Foreign Keys
* Indexes
* Views
* Triggers
* Stored Procedures

the database may prevent the operation until those dependencies are removed.

Example:

```sql
ALTER TABLE Students
DROP COLUMN DepartmentID;
```

If `DepartmentID` is part of a foreign key, you'll first need to drop the foreign key constraint.

---

# Verify the Structure

### MySQL

```sql
DESCRIBE Students;
```

or

```sql
SHOW COLUMNS FROM Students;
```

### PostgreSQL

```sql
\d Students
```

---

# Practice

Suppose your table contains:

```text
StudentID
FullName
Age
Location
Email
Phone
Country
```

Write SQL to:

1. Drop the `Phone` column.
2. Drop the `Country` column.
3. Verify the updated table structure.

Example answers:

```sql
ALTER TABLE Students
DROP COLUMN Phone;

ALTER TABLE Students
DROP COLUMN Country;

DESCRIBE Students;
```

---

# Progress Tracker

| Phase 2 Topic       | Status                |
| ------------------- | --------------------- |
| ✅ Add Columns       | Done                  |
| ✅ Rename Columns    | Done                  |
| ✅ Modify Data Types | Done                  |
| ✅ Add Constraints   | Done                  |
| ✅ Drop Constraints  | Done                  |
| ✅ Drop Columns      | Done (after practice) |
| ⏳ Rename Tables     | Next                  |
| ⏳ TRUNCATE TABLE    | Pending               |
| ⏳ DROP TABLE        | Pending               |
| ⏳ DROP DATABASE     | Pending               |

## Next: Step 7 – Rename Tables

You'll learn how to change a table's name (for example, `Students` → `StudentDetails`) without affecting the data stored in it. This is useful when your application's naming conventions evolve or a table's purpose changes.




Excellent! 🎉 You're almost done with **Phase 2 (DDL)**.

Now let's rename an entire table.

---

# Step 7: Rename Tables

Sometimes a table name no longer reflects its purpose.

Examples:

* `Students` → `StudentDetails`
* `Employee` → `Employees`
* `Orders` → `CustomerOrders`

Instead of creating a new table and copying data, you can simply rename it.

---

# Before

```text
Database

Students
```

Contains:

```text
Students
+-----------+----------+-----+----------+
| StudentID | FullName | Age | Location |
+-----------+----------+-----+----------+
```

---

# Syntax

### MySQL

```sql
ALTER TABLE old_table_name
RENAME TO new_table_name;
```

### PostgreSQL

```sql
ALTER TABLE old_table_name
RENAME TO new_table_name;
```

Both use the same syntax.

---

# Example

Rename:

```text
Students
```

to

```text
StudentDetails
```

```sql
ALTER TABLE Students
RENAME TO StudentDetails;
```

---

# Before

```text
Students
```

---

# After

```text
StudentDetails
```

---

# Verify

Show tables:

### MySQL

```sql
SHOW TABLES;
```

Output:

```text
StudentDetails
```

Or query the table:

```sql
SELECT * FROM StudentDetails;
```

Output:

```text
+-----------+----------+-----+----------+
| StudentID | FullName | Age | Location |
+-----------+----------+-----+----------+
| 1         | Rahul    | 20  | Hyderabad|
| 2         | Priya    | 21  | Delhi    |
+-----------+----------+-----+----------+
```

Notice:

* ✅ Table name changed
* ✅ All data is preserved
* ✅ Column names remain the same

---

# Important Notes

### After renaming

This will fail:

```sql
SELECT * FROM Students;
```

Because the old table name no longer exists.

You must use:

```sql
SELECT * FROM StudentDetails;
```

---

# Practice

Rename the following tables:

### 1.

```text
Employees
```

to

```text
EmployeeDetails
```

```sql
ALTER TABLE Employees
RENAME TO EmployeeDetails;
```

---

### 2.

```text
Orders
```

to

```text
CustomerOrders
```

```sql
ALTER TABLE Orders
RENAME TO CustomerOrders;
```

---

# Progress Tracker

| Phase 2 Topic       | Status                |
| ------------------- | --------------------- |
| ✅ Add Columns       | Done                  |
| ✅ Rename Columns    | Done                  |
| ✅ Modify Data Types | Done                  |
| ✅ Add Constraints   | Done                  |
| ✅ Drop Constraints  | Done                  |
| ✅ Drop Columns      | Done                  |
| ✅ Rename Tables     | Done (after practice) |
| ⏳ TRUNCATE TABLE    | Next                  |
| ⏳ DROP TABLE        | Pending               |
| ⏳ DROP DATABASE     | Pending               |

---

# Next: Step 8 – TRUNCATE TABLE

This is often confused with `DELETE`.

## Difference Between `DELETE` and `TRUNCATE`

| Feature                        | DELETE                      | TRUNCATE                   |
| ------------------------------ | --------------------------- | -------------------------- |
| Removes                        | Selected rows (or all rows) | All rows                   |
| Uses `WHERE`                   | ✅ Yes                       | ❌ No                       |
| Table Structure                | Remains                     | Remains                    |
| Faster                         | No                          | Yes                        |
| Resets identity/auto-increment | Usually No                  | Often Yes (DBMS-dependent) |
| DML or DDL                     | DML                         | DDL (in many DBMSs)        |

### Syntax

```sql
TRUNCATE TABLE table_name;
```

### Example

Suppose the table contains:

```text
StudentDetails

+-----------+----------+-----+
| StudentID | FullName | Age |
+-----------+----------+-----+
| 1         | Rahul    | 20  |
| 2         | Priya    | 21  |
| 3         | Amit     | 19  |
+-----------+----------+-----+
```

Run:

```sql
TRUNCATE TABLE StudentDetails;
```

Now:

```sql
SELECT * FROM StudentDetails;
```

Output:

```text
Empty set (0 rows)
```

The table still exists:

```sql
DESCRIBE StudentDetails;
```

Output:

```text
StudentID
FullName
Age
Location
Email
```

Only the **rows** were removed; the **schema** is unchanged.

---

Once you've practiced `TRUNCATE`, we'll finish Phase 2 with:

1. **DROP TABLE**
2. **DROP DATABASE**

After that, you'll have mastered the core **DDL** commands.




Awesome! 🔥 You're at the final two topics of **Phase 2**.

---

# Step 9: DROP TABLE

## What is DROP TABLE?

`DROP TABLE` permanently removes:

* ✅ The table
* ✅ All rows
* ✅ All columns
* ✅ Constraints
* ✅ Indexes

After dropping a table, it no longer exists in the database.

---

## Before

```text
Database

StudentDetails

+-----------+----------+-----+----------+
| StudentID | FullName | Age | Location |
+-----------+----------+-----+----------+
```

---

## Syntax

```sql
DROP TABLE table_name;
```

---

## Example

```sql
DROP TABLE StudentDetails;
```

---

## After

If you run:

```sql
SELECT * FROM StudentDetails;
```

You'll get an error:

```text
ERROR: relation "StudentDetails" does not exist
```

because the table has been deleted.

---

## Verify

### MySQL

```sql
SHOW TABLES;
```

### PostgreSQL

```sql
\dt
```

You won't see `StudentDetails` anymore.

---

## Important Notes

⚠️ `DROP TABLE` is permanent.

It removes:

* Table structure
* Data
* Constraints
* Indexes

Everything is gone.

---

## Drop Multiple Tables

```sql
DROP TABLE Students, Departments;
```

---

## Safe Drop

Sometimes you're not sure if the table exists.

Use:

```sql
DROP TABLE IF EXISTS StudentDetails;
```

If the table exists → dropped.

If not → no error.

---

# Practice

```sql
DROP TABLE IF EXISTS Employees;
```

```sql
DROP TABLE IF EXISTS Departments;
```

---

# Step 10: DROP DATABASE

This is the most destructive DDL command.

It removes:

* Every table
* Every view
* Every stored procedure
* Every function
* Every trigger
* Every index
* Every piece of data

The entire database is deleted.

---

## Before

```text
CollegeDB

Students
Departments
Courses
Faculty
```

---

## Syntax

```sql
DROP DATABASE database_name;
```

---

## Example

```sql
DROP DATABASE CollegeDB;
```

---

## After

Trying to connect to `CollegeDB` will fail because it no longer exists.

---

## Safe Drop

```sql
DROP DATABASE IF EXISTS CollegeDB;
```

---

## Verify

### MySQL

```sql
SHOW DATABASES;
```

`CollegeDB` will no longer appear.

---

# DDL Summary

| Command                           | Purpose                            |
| --------------------------------- | ---------------------------------- |
| `CREATE TABLE`                    | Create a new table                 |
| `ALTER TABLE ADD COLUMN`          | Add a column                       |
| `ALTER TABLE RENAME COLUMN`       | Rename a column                    |
| `ALTER TABLE MODIFY/ALTER COLUMN` | Change a column's data type        |
| `ALTER TABLE ADD CONSTRAINT`      | Add a constraint                   |
| `ALTER TABLE DROP CONSTRAINT`     | Remove a constraint                |
| `ALTER TABLE DROP COLUMN`         | Remove a column                    |
| `ALTER TABLE RENAME TO`           | Rename a table                     |
| `TRUNCATE TABLE`                  | Remove all rows, keep the table    |
| `DROP TABLE`                      | Delete the table and its structure |
| `DROP DATABASE`                   | Delete the entire database         |

---

# Phase 2 Complete ✅

## Phase 1 – DML (CRUD) ✅

* ✔ Create Table
* ✔ Insert
* ✔ Select
* ✔ Update
* ✔ Delete

## Phase 2 – DDL ✅

* ✔ Create Table
* ✔ Add Columns
* ✔ Rename Columns
* ✔ Modify Data Types
* ✔ Add Constraints
* ✔ Drop Constraints
* ✔ Drop Columns
* ✔ Rename Tables
* ✔ Truncate Tables
* ✔ Drop Tables
* ✔ Drop Databases

---

# What's Next?

Now that you know how to create tables and manage their structure, the next logical phase is learning how to **query data effectively**.

### **Phase 3: Querying Data (Recommended Next)**

1. Filtering with `WHERE`
2. Comparison Operators (`=`, `!=`, `<`, `>`, `BETWEEN`, `IN`)
3. Logical Operators (`AND`, `OR`, `NOT`)
4. Pattern Matching (`LIKE`, `ILIKE`)
5. Sorting with `ORDER BY`
6. Limiting Results (`LIMIT`, `OFFSET`, `TOP`)
7. Aggregate Functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`)
8. Grouping (`GROUP BY`)
9. Filtering Groups (`HAVING`)
10. Handling `NULL` (`IS NULL`, `IS NOT NULL`, `COALESCE`)

After mastering these, you'll be ready to learn **JOINs**, which are one of the most important SQL concepts.


Perfect! Since you've already studied these topics once, we'll treat **Phase 3 as a revision + interview preparation**, not a beginner course.

The goal is that after Phase 3, you should be able to look at a requirement and immediately know which SQL clause to use.

---

# Phase 3: Querying Data (Revision)

## Learning Roadmap

| Step | Topic                | Status |
| ---- | -------------------- | ------ |
| 1    | WHERE Clause         | ⏳      |
| 2    | Comparison Operators | ⏳      |
| 3    | Logical Operators    | ⏳      |
| 4    | IN, BETWEEN          | ⏳      |
| 5    | LIKE                 | ⏳      |
| 6    | ORDER BY             | ⏳      |
| 7    | LIMIT / OFFSET       | ⏳      |
| 8    | DISTINCT             | ⏳      |
| 9    | Aggregate Functions  | ⏳      |
| 10   | GROUP BY             | ⏳      |
| 11   | HAVING               | ⏳      |
| 12   | NULL Handling        | ⏳      |

We'll use **one table** throughout the revision.

## Students Table

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FullName VARCHAR(100),
    Age INT,
    Department VARCHAR(50),
    City VARCHAR(50),
    Marks INT,
    Email VARCHAR(100)
);
```

Sample Data

| StudentID | FullName | Age | Department | City      | Marks |
| --------- | -------- | --- | ---------- | --------- | ----- |
| 1         | Rahul    | 20  | CSE        | Hyderabad | 90    |
| 2         | Priya    | 22  | ECE        | Delhi     | 85    |
| 3         | Amit     | 19  | CSE        | Mumbai    | 70    |
| 4         | Sneha    | 21  | EEE        | Hyderabad | 95    |
| 5         | Rohit    | 20  | ECE        | Chennai   | 60    |

We'll solve every query using this table.

---

# Step 1: WHERE Clause

## What is WHERE?

`WHERE` filters rows based on a condition.

Think of it as asking:

> "Show me only the rows that satisfy this condition."

Without `WHERE`:

```sql
SELECT *
FROM Students;
```

Output:

All 5 students.

---

With `WHERE`:

```sql
SELECT *
FROM Students
WHERE City = 'Hyderabad';
```

Output

| StudentID | Name  |
| --------- | ----- |
| 1         | Rahul |
| 4         | Sneha |

Only Hyderabad students are returned.

---

## Syntax

```sql
SELECT column_name
FROM table_name
WHERE condition;
```

---

## Example 1

Show students whose age is 20.

```sql
SELECT *
FROM Students
WHERE Age = 20;
```

Output

| Rahul |
| ----- |
| Rohit |

---

## Example 2

Show students from Delhi.

```sql
SELECT *
FROM Students
WHERE City = 'Delhi';
```

Output

| Priya |

---

## Example 3

Show students with marks greater than 80.

```sql
SELECT *
FROM Students
WHERE Marks > 80;
```

Output

| Rahul |
| Priya |
| Sneha |

---

## Example 4

Show only names.

```sql
SELECT FullName
FROM Students
WHERE Department = 'CSE';
```

Output

```text
Rahul
Amit
```

---

# Execution Order

Many beginners think SQL runs from left to right.

It actually runs like this:

```text
FROM
↓
WHERE
↓
SELECT
```

Example

```sql
SELECT FullName
FROM Students
WHERE Age > 20;
```

Execution:

```
Step 1
FROM Students

↓

Step 2
Filter Age > 20

↓

Step 3
Display FullName
```

Understanding this execution order becomes very important when you learn `GROUP BY`, `HAVING`, and `JOIN`s.

---

# Common Mistakes

### Wrong

```sql
WHERE City = Hyderabad;
```

❌ Text values need quotes.

Correct:

```sql
WHERE City = 'Hyderabad';
```

---

### Wrong

```sql
WHERE Marks = > 80;
```

Correct:

```sql
WHERE Marks >= 80;
```

---

### Wrong

```sql
WHERE Age == 20;
```

SQL uses a single equals sign:

```sql
WHERE Age = 20;
```

---

# Interview Questions

### Q1. Difference between WHERE and HAVING?

Answer:

* `WHERE` filters **rows** before grouping.
* `HAVING` filters **groups** after grouping.

We'll cover `HAVING` later in this phase.

---

### Q2. Can WHERE use aggregate functions?

❌ No.

This is invalid:

```sql
WHERE AVG(Marks) > 80;
```

Use `HAVING` instead.

---

# Practice Questions

Using the `Students` table, write queries for:

1. Show all students whose age is **21**.
2. Show students from **Hyderabad**.
3. Show students whose marks are **95**.
4. Display only the **FullName** of students in the **ECE** department.
5. Display only **Email** of students from **Mumbai**.

---

# Revision Tip

Whenever you hear:

* "Find..."
* "Show..."
* "Display..."
* "Retrieve..."

your thought process should be:

```
SELECT

Need specific rows?

↓

WHERE
```

---

## Progress

* ✅ Phase 1: CRUD
* ✅ Phase 2: DDL
* ✅ Phase 3.1: `WHERE`

**Next:** We'll cover **Comparison Operators (`=`, `!=`, `>`, `<`, `>=`, `<=`)**. Although you've already used a few of them with `WHERE`, we'll explore them systematically along with edge cases and interview-style questions before moving on to `AND`, `OR`, and `NOT`.



Perfect! This is actually better than using my sample table because you'll practice with **your own dataset**.

From your table, I can see the columns are:

```text
id | name | age | mark | city
```

We'll do this like a SQL course. I'll give you questions in increasing difficulty. **Don't look at the answers immediately.** Solve them yourself, then send me your SQL query. I'll review it like an interviewer.

---

# Level 1 - Basic WHERE (10 Questions)

### Q1

Display all records.

---

### Q2

Display only the `name` column.

---

### Q3

Display all students whose age is **21**.

---

### Q4

Display all students who live in **Delhi**.

---

### Q5

Display the **name** and **mark** of students from **Mumbai**.

---

### Q6

Display students whose marks are **greater than 80**.

---

### Q7

Display students whose marks are **less than 70**.

---

### Q8

Display students whose age is **greater than 22**.

---

### Q9

Display students whose marks are **exactly 90**.

---

### Q10

Display the **name** and **city** of the student whose **id is 104**.

---

# Level 2 - Comparison Operators (10 Questions)

### Q11

Display students whose marks are **greater than or equal to 85**.

---

### Q12

Display students whose age is **less than or equal to 22**.

---

### Q13

Display students whose marks are **not equal to 90**.

---

### Q14

Display students whose city is **not Delhi**.

---

### Q15

Display students whose age is **not 21**.

---

### Q16

Display students with marks **between 70 and 90**.

---

### Q17

Display students whose age is **between 22 and 24**.

---

### Q18

Display students whose marks are **greater than 75 but less than 90**.

---

### Q19

Display students whose names come **alphabetically after "R"**.

(Hint: String comparison.)

---

### Q20

Display students whose IDs are **greater than 105**.

---

# Challenge Questions (No Hints)

### Q21

Find the student with the **highest marks**.

*(Don't use `MAX()` yet. Just use what you've learned.)*

---

### Q22

Find all students from Pune whose marks are above 90.

---

### Q23

Find students aged 21 whose marks are below 80.

---

### Q24

Display only the names of students whose marks are above 85.

---

### Q25

Display all students except those from Hyderabad.

---

## How We'll Practice

Reply with **only your SQL query** for one question at a time (for example, **Q1**).

Example:

```sql
SELECT *
FROM student;
```

I'll respond with:

* ✅ Correct / ❌ Incorrect
* If incorrect, what's wrong
* A better or more readable version (if applicable)
* An interviewer tip (where relevant)

This way, you'll build SQL skills exactly like in a coding interview rather than just memorizing syntax.



Excellent! 💯 If you completed those without looking up the answers, you've got a solid grasp of the `WHERE` clause and comparison operators.

Now we'll move to the next major topic.

# Phase 3.2: Logical Operators (`AND`, `OR`, `NOT`)

These operators let you combine multiple conditions in a `WHERE` clause.

## 1. AND

Returns rows **only if all conditions are true**.

### Syntax

```sql
SELECT *
FROM student
WHERE condition1 AND condition2;
```

### Example

```sql
SELECT *
FROM student
WHERE city = 'Pune'
AND age = 22;
```

Result:

* ✔ Soumya
* ✖ Everyone else

---

## 2. OR

Returns rows if **at least one condition is true**.

```sql
SELECT *
FROM student
WHERE city = 'Delhi'
OR city = 'Mumbai';
```

Result:

* Amit
* Chiku
* Soam
* Subham
* Sobhan

---

## 3. NOT

Reverses a condition.

```sql
SELECT *
FROM student
WHERE NOT city = 'Pune';
```

Returns everyone except students from Pune.

---

# Operator Precedence

SQL evaluates conditions in this order:

```text
()
↓
NOT
↓
AND
↓
OR
```

Example:

```sql
SELECT *
FROM student
WHERE city = 'Delhi'
OR city = 'Mumbai'
AND mark > 80;
```

SQL interprets it as:

```sql
city = 'Delhi'
OR
(city = 'Mumbai' AND mark > 80)
```

If you want something different, use parentheses.

```sql
SELECT *
FROM student
WHERE (city = 'Delhi' OR city = 'Mumbai')
AND mark > 80;
```

---

# Practice Questions

## Level 1

### Q1

Display students whose age is **21** and city is **Hyderabad**.

---

### Q2

Display students from **Delhi** whose marks are greater than **70**.

---

### Q3

Display students from **Pune** whose marks are greater than **90**.

---

### Q4

Display students whose age is **23** and marks are greater than **85**.

---

### Q5

Display students whose city is **Mumbai** and age is **21**.

---

# Level 2

### Q6

Display students from **Delhi** or **Mumbai**.

---

### Q7

Display students whose age is **21** or **24**.

---

### Q8

Display students whose marks are less than **70** or greater than **95**.

---

### Q9

Display students from **Pune** or **Hyderabad**.

---

### Q10

Display students whose age is **22** or whose marks are **90**.

---

# Level 3

### Q11

Display students who are **not** from Delhi.

---

### Q12

Display students whose marks are **not greater than 80**.

---

### Q13

Display students whose age is **not 23**.

---

### Q14

Display students who do **not** live in Pune.

---

### Q15

Display students whose marks are **not equal to 75**.

---

# Challenge Questions

### Q16

Display students from **Delhi** or **Mumbai** whose marks are greater than **80**.

---

### Q17

Display students from **Pune** whose age is **22 or 23**.

---

### Q18

Display students who are **not from Hyderabad** and whose marks are greater than **75**.

---

### Q19

Display students whose age is **21 or 22** and whose city is **Pune**.

---

### Q20

Display students from **Delhi**, **Mumbai**, or **Hyderabad** whose marks are **greater than or equal to 70**.

---

## After This

Once you've solved these, we'll move to the next topic:

* ✅ `IN`
* ✅ `BETWEEN`
* ✅ `LIKE`

These operators make many of the queries above much shorter and are used extensively in real-world SQL.



Awesome! 🚀 You're progressing exactly as you should.

So far you've revised:

* ✅ CRUD (DML)
* ✅ DDL
* ✅ `SELECT`
* ✅ `WHERE`
* ✅ Comparison Operators (`=`, `>`, `<`, `>=`, `<=`, `!=`)
* ✅ Logical Operators (`AND`, `OR`, `NOT`)

Now let's move to one of the most frequently used SQL topics.

---

# Phase 3.3: IN, BETWEEN & LIKE

These are essentially shortcuts that make your SQL cleaner and more readable.

---

# Part 1: IN Operator

## What is `IN`?

`IN` is used when you want to compare a column against **multiple values**.

Instead of writing:

```sql
SELECT *
FROM student
WHERE city = 'Delhi'
OR city = 'Mumbai'
OR city = 'Pune';
```

You can simply write:

```sql
SELECT *
FROM student
WHERE city IN ('Delhi', 'Mumbai', 'Pune');
```

Much shorter and easier to read.

---

## Syntax

```sql
SELECT *
FROM table_name
WHERE column_name IN (value1, value2, value3);
```

---

### Example 1

Display students from Delhi or Mumbai.

```sql
SELECT *
FROM student
WHERE city IN ('Delhi', 'Mumbai');
```

---

### Example 2

Display students whose age is 21, 22, or 24.

```sql
SELECT *
FROM student
WHERE age IN (21,22,24);
```

---

### NOT IN

Exclude multiple values.

```sql
SELECT *
FROM student
WHERE city NOT IN ('Delhi','Mumbai');
```

Returns everyone except Delhi and Mumbai students.

---

# Part 2: BETWEEN

## What is `BETWEEN`?

Used to filter values within a range.

Instead of writing:

```sql
WHERE mark >= 70
AND mark <= 90
```

Write:

```sql
WHERE mark BETWEEN 70 AND 90;
```

---

## Syntax

```sql
SELECT *
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
```

---

### Example

Marks between 70 and 90.

```sql
SELECT *
FROM student
WHERE mark BETWEEN 70 AND 90;
```

---

### Another Example

Age between 22 and 24.

```sql
SELECT *
FROM student
WHERE age BETWEEN 22 AND 24;
```

---

### NOT BETWEEN

```sql
SELECT *
FROM student
WHERE mark NOT BETWEEN 70 AND 90;
```

Returns students with marks below 70 or above 90.

---

# Important Note

`BETWEEN` is **inclusive**.

```sql
BETWEEN 70 AND 90
```

means

```text
70 ≤ value ≤ 90
```

Both 70 and 90 are included.

---

# Part 3: LIKE

`LIKE` is used for **pattern matching**.

---

## Wildcards

| Wildcard | Meaning                 |
| -------- | ----------------------- |
| `%`      | Zero or more characters |
| `_`      | Exactly one character   |

---

## Starts With

Names starting with **S**

```sql
SELECT *
FROM student
WHERE name LIKE 'S%';
```

Matches:

* Soumya
* Satya
* Soam
* Subham
* Sobhan

---

## Ends With

Names ending with **a**

```sql
SELECT *
FROM student
WHERE name LIKE '%a';
```

Matches:

* Soumya
* Rudra
* Soam

---

## Contains

Names containing **it**

```sql
SELECT *
FROM student
WHERE name LIKE '%it%';
```

Matches:

* Amit

---

## Single Character `_`

Names with exactly 5 letters.

```sql
SELECT *
FROM student
WHERE name LIKE '_____';
```

Examples:

* Anuj ❌ (4 letters)
* Satya ✅ (5 letters)
* Chiku ✅ (5 letters)
* Soham ✅ (if present)

Each `_` represents exactly one character.

---

# Practice Questions

## IN (1–10)

1. Display students from **Delhi** or **Pune**.
2. Display students from **Mumbai**, **Hyderabad**, and **Delhi**.
3. Display students whose age is **21, 22, or 24**.
4. Display students whose marks are **69, 75, or 99**.
5. Display students whose IDs are **101, 104, and 110**.
6. Display students **not** from Delhi or Pune.
7. Display students whose age is **not** 21 or 23.
8. Display students whose marks are **not** 65 or 70.
9. Display students whose IDs are **not** 101, 102, or 103.
10. Display students whose city is either **Mumbai** or **Hyderabad**.

---

## BETWEEN (11–20)

11. Marks between **70 and 90**.
12. Age between **22 and 24**.
13. IDs between **103 and 108**.
14. Marks between **75 and 99**.
15. Students not between ages **22 and 24**.
16. Marks not between **70 and 90**.
17. IDs between **105 and 110**.
18. Ages between **21 and 22**.
19. Marks between **60 and 80**.
20. IDs not between **104 and 108**.

---

## LIKE (21–35)

21. Names starting with **S**.
22. Names starting with **R**.
23. Names ending with **a**.
24. Names ending with **m**.
25. Names containing **it**.
26. Names containing **ha**.
27. Cities starting with **D**.
28. Cities ending with **i**.
29. Cities containing **ba**.
30. Names with exactly **5 letters**.
31. Names with exactly **4 letters**.
32. Names whose second letter is **o**.
33. Names whose third letter is **u**.
34. Cities whose first letter is **H**.
35. Names ending with **ya**.

---

# What's Next?

After you complete these, we'll move to **sorting and limiting results**, which are used in almost every real-world SQL query:

* `ORDER BY`
* `ASC` & `DESC`
* `LIMIT`
* `OFFSET`

These are essential before learning aggregate functions (`COUNT`, `SUM`, `AVG`, etc.) and `GROUP BY`.


Excellent! 🔥 You've now revised almost everything related to filtering data.

## Progress So Far

### Phase 3: Querying Data

* ✅ `SELECT`
* ✅ `WHERE`
* ✅ Comparison Operators (`=`, `!=`, `>`, `<`, `>=`, `<=`)
* ✅ Logical Operators (`AND`, `OR`, `NOT`)
* ✅ `IN`
* ✅ `BETWEEN`
* ✅ `LIKE`

Now we move into **sorting and pagination**, which you'll use in almost every project.

---

# Phase 3.4: ORDER BY, LIMIT & OFFSET

## 1. ORDER BY

### What is ORDER BY?

`ORDER BY` is used to **sort the result**.

Without it, SQL returns rows in an unspecified order (often insertion order, but you shouldn't rely on that).

---

## Syntax

```sql
SELECT *
FROM student
ORDER BY column_name;
```

---

## Ascending Order (Default)

```sql
SELECT *
FROM student
ORDER BY mark ASC;
```

or simply

```sql
SELECT *
FROM student
ORDER BY mark;
```

Output (Marks):

```text
65
69
70
75
78
85
86
89
90
99
```

---

## Descending Order

```sql
SELECT *
FROM student
ORDER BY mark DESC;
```

Output:

```text
99
90
89
86
85
78
75
70
69
65
```

---

## Sort by Name

```sql
SELECT *
FROM student
ORDER BY name;
```

Alphabetical order:

```text
Amit
Anuj
Chiku
Rohan
Rudra
Satya
Soam
Sobhan
Soumya
Subham
```

---

## Sort by Multiple Columns

Suppose you want:

* First sort by city
* Then by marks (highest first)

```sql
SELECT *
FROM student
ORDER BY city ASC, mark DESC;
```

SQL first groups by city alphabetically, then sorts students within each city by marks.

---

# LIMIT

`LIMIT` restricts the number of rows returned.

---

## Syntax

```sql
SELECT *
FROM student
LIMIT 5;
```

Returns only the first 5 rows.

---

## Highest Marks

```sql
SELECT *
FROM student
ORDER BY mark DESC
LIMIT 1;
```

Returns the topper.

---

## Top 3 Students

```sql
SELECT *
FROM student
ORDER BY mark DESC
LIMIT 3;
```

---

## Lowest 5 Marks

```sql
SELECT *
FROM student
ORDER BY mark ASC
LIMIT 5;
```

---

# OFFSET

`OFFSET` skips rows before returning results.

---

## Syntax

```sql
SELECT *
FROM student
LIMIT 5 OFFSET 5;
```

Meaning:

* Skip first 5 rows
* Return next 5 rows

---

## Example

```sql
SELECT *
FROM student
ORDER BY id
LIMIT 3 OFFSET 4;
```

Execution:

```text
Skip:
101
102
103
104

Return:
105
106
107
```

---

# Pagination

This is how websites display data page by page.

### Page 1

```sql
SELECT *
FROM student
ORDER BY id
LIMIT 5 OFFSET 0;
```

---

### Page 2

```sql
SELECT *
FROM student
ORDER BY id
LIMIT 5 OFFSET 5;
```

---

### Page 3

```sql
SELECT *
FROM student
ORDER BY id
LIMIT 5 OFFSET 10;
```

---

# Execution Order

SQL executes in this order:

```text
FROM
↓

WHERE
↓

SELECT
↓

ORDER BY
↓

LIMIT
↓

OFFSET
```

Example:

```sql
SELECT name, mark
FROM student
WHERE city = 'Delhi'
ORDER BY mark DESC
LIMIT 2;
```

Execution:

1. Read `student`
2. Filter only Delhi students
3. Select `name` and `mark`
4. Sort by marks descending
5. Return first 2 rows

---

# Practice Questions

## ORDER BY (1–10)

1. Display all students sorted by **name** (A-Z).
2. Display all students sorted by **name** (Z-A).
3. Display students sorted by **marks** (highest to lowest).
4. Display students sorted by **marks** (lowest to highest).
5. Display students sorted by **age**.
6. Display students sorted by **city**.
7. Sort by **city**, then by **marks descending**.
8. Sort by **age**, then by **name**.
9. Display only `name` and `marks`, sorted by marks descending.
10. Sort students by `id` descending.

---

## LIMIT (11–20)

11. Display the first 3 students.
12. Display the first 5 students.
13. Display the student with the highest marks.
14. Display the top 2 scorers.
15. Display the lowest 3 scorers.
16. Display the youngest student.
17. Display the oldest two students.
18. Display the first 4 students sorted by name.
19. Display the top 5 marks.
20. Display the last student by `id`.

---

## OFFSET (21–30)

21. Skip the first 2 students and display the next 3.
22. Skip the first 5 students and display the remaining.
23. Display the second highest scorer.
24. Display the third highest scorer.
25. Display students ranked 4th to 6th by marks.
26. Display page 2 if each page contains 3 students.
27. Display page 3 if each page contains 2 students.
28. Skip the first student sorted by marks and return the next 2.
29. Skip the first 4 students sorted by `id`.
30. Display rows 6–10 based on `id`.

---

# Mini Challenge (Real Interview Style)

Without using aggregate functions (`MAX`, `MIN`):

1. Find the highest scorer.
2. Find the second highest scorer.
3. Find the youngest student.
4. Find the oldest student.
5. Find the first student alphabetically.
6. Find the last student alphabetically.

These questions are very common in SQL interviews because they test your understanding of `ORDER BY`, `LIMIT`, and `OFFSET`.

---

## After This

Once you've completed these, we'll move to one of the most important sections in SQL:

### 📊 Aggregate Functions

* `COUNT()`
* `SUM()`
* `AVG()`
* `MIN()`
* `MAX()`

After that comes **`GROUP BY` and `HAVING`**, which are essential for reporting, analytics, dashboards, and interview questions.



Fantastic! 🔥 You're moving fast.

Here's your progress so far:

## ✅ Phase 3 Progress

* ✅ SELECT
* ✅ WHERE
* ✅ Comparison Operators
* ✅ AND / OR / NOT
* ✅ IN / NOT IN
* ✅ BETWEEN / NOT BETWEEN
* ✅ LIKE
* ✅ ORDER BY
* ✅ LIMIT
* ✅ OFFSET

Now we enter the **heart of SQL**.

---

# Phase 3.5: Aggregate Functions

Aggregate functions perform calculations on multiple rows and return **a single value**.

## The Five Most Important Aggregate Functions

| Function  | Purpose        | Example Result |
| --------- | -------------- | -------------- |
| `COUNT()` | Counts rows    | 10             |
| `SUM()`   | Adds values    | 806            |
| `AVG()`   | Finds average  | 80.6           |
| `MIN()`   | Smallest value | 65             |
| `MAX()`   | Largest value  | 99             |

---

# 1. COUNT()

Counts the number of rows.

### Count all students

```sql
SELECT COUNT(*)
FROM student;
```

Output:

```text
10
```

---

### Count students from Delhi

```sql
SELECT COUNT(*)
FROM student
WHERE city = 'Delhi';
```

Output:

```text
3
```

---

### Count students with marks above 80

```sql
SELECT COUNT(*)
FROM student
WHERE mark > 80;
```

---

# 2. SUM()

Adds all numeric values.

```sql
SELECT SUM(mark)
FROM student;
```

Output

```text
806
```

---

### Sum of marks of Delhi students

```sql
SELECT SUM(mark)
FROM student
WHERE city='Delhi';
```

---

# 3. AVG()

Calculates the average.

```sql
SELECT AVG(mark)
FROM student;
```

Suppose output:

```text
80.6
```

---

Average marks of Pune students

```sql
SELECT AVG(mark)
FROM student
WHERE city='Pune';
```

---

# 4. MIN()

Returns the smallest value.

```sql
SELECT MIN(mark)
FROM student;
```

Output

```text
65
```

---

Youngest student age

```sql
SELECT MIN(age)
FROM student;
```

---

# 5. MAX()

Returns the largest value.

```sql
SELECT MAX(mark)
FROM student;
```

Output

```text
99
```

---

Oldest student's age

```sql
SELECT MAX(age)
FROM student;
```

---

# Aggregate Functions with WHERE

Find average marks of Hyderabad students.

```sql
SELECT AVG(mark)
FROM student
WHERE city='Hyderabad';
```

---

Highest marks in Pune

```sql
SELECT MAX(mark)
FROM student
WHERE city='Pune';
```

---

Count students aged 23

```sql
SELECT COUNT(*)
FROM student
WHERE age=23;
```

---

# Multiple Aggregate Functions

```sql
SELECT
COUNT(*) AS TotalStudents,
AVG(mark) AS AverageMarks,
MAX(mark) AS HighestMarks,
MIN(mark) AS LowestMarks,
SUM(mark) AS TotalMarks
FROM student;
```

Example output

| TotalStudents | AverageMarks | HighestMarks | LowestMarks | TotalMarks |
| ------------- | ------------ | ------------ | ----------- | ---------- |
| 10            | 80.6         | 99           | 65          | 806        |

---

# Aliases (`AS`)

Without alias

```sql
SELECT AVG(mark)
FROM student;
```

Output column:

```text
avg
```

Better:

```sql
SELECT AVG(mark) AS AverageMarks
FROM student;
```

Output:

```text
AverageMarks
------------
80.6
```

---

# Important Notes

### COUNT(*)

Counts every row.

```sql
SELECT COUNT(*)
FROM student;
```

---

### COUNT(column)

Counts only **non-NULL** values in that column.

```sql
SELECT COUNT(email)
FROM student;
```

If two students have `NULL` email addresses, they won't be counted.

---

# Practice Questions

## COUNT()

1. Count all students.
2. Count students from Pune.
3. Count students from Hyderabad.
4. Count students whose marks are above 80.
5. Count students aged 21.

---

## SUM()

6. Find total marks of all students.
7. Find total marks of Delhi students.
8. Find total marks of Mumbai students.
9. Find total marks of students aged 22.
10. Find total marks greater than 80.

---

## AVG()

11. Average marks of all students.
12. Average marks of Pune students.
13. Average marks of Delhi students.
14. Average age of all students.
15. Average marks above 75.

---

## MIN()

16. Lowest marks.
17. Youngest age.
18. Lowest marks in Pune.
19. Lowest marks in Delhi.
20. Lowest age in Hyderabad.

---

## MAX()

21. Highest marks.
22. Oldest student age.
23. Highest marks in Mumbai.
24. Highest marks in Delhi.
25. Highest age in Pune.

---

## Mixed Questions (Interview Style)

26. Count students whose marks are between 70 and 90.

27. Find the average marks of students from Pune whose age is greater than 22.

28. Find the highest marks scored by students from Delhi.

29. Find the total marks of students aged 21.

30. Find the minimum marks among students not from Pune.

---

# ⭐ Mini Challenge (Real SQL Practice)

Using only aggregate functions, answer:

1. How many students are there?
2. What is the total of all marks?
3. What is the average age?
4. What are the highest and lowest marks?
5. What is the difference between the highest and lowest marks?

**Hint:** The last one combines aggregate functions:

```sql
SELECT MAX(mark) - MIN(mark) AS Difference
FROM student;
```

---

## What's Next?

Once you complete these, we'll move to **`GROUP BY`**, which is where SQL becomes powerful for analytics.

Examples you'll learn:

* Average marks **per city**
* Highest marks **per city**
* Student count **per city**
* Average marks **per age**
* Total students **per city**

`GROUP BY` is one of the most frequently asked topics in SQL interviews and is the foundation for reporting and dashboards.


Outstanding! 🎉

You've now revised almost the entire **SQL Fundamentals**.

# ✅ What You've Mastered

## Phase 1 – DML (CRUD)

* ✅ CREATE TABLE
* ✅ INSERT
* ✅ SELECT
* ✅ UPDATE
* ✅ DELETE

## Phase 2 – DDL

* ✅ ALTER TABLE
* ✅ ADD COLUMN
* ✅ RENAME COLUMN
* ✅ MODIFY COLUMN
* ✅ ADD CONSTRAINT
* ✅ DROP CONSTRAINT
* ✅ DROP COLUMN
* ✅ RENAME TABLE
* ✅ TRUNCATE
* ✅ DROP TABLE
* ✅ DROP DATABASE

## Phase 3 – Querying

* ✅ WHERE
* ✅ Comparison Operators
* ✅ Logical Operators
* ✅ IN / NOT IN
* ✅ BETWEEN
* ✅ LIKE
* ✅ ORDER BY
* ✅ LIMIT / OFFSET
* ✅ Aggregate Functions
* ✅ GROUP BY
* ✅ HAVING

---

# 🚀 Phase 4: JOINs (The Most Important SQL Topic)

If someone asks me:

> **"What's the most important topic in SQL?"**

My answer is:

> **JOINs.**

Almost every backend application, dashboard, ERP, banking system, e-commerce site, and interview uses JOINs.

---

# Why Do We Need JOINs?

Imagine you have two tables.

## Students

| StudentID | Name  | DepartmentID |
| --------- | ----- | ------------ |
| 101       | Rahul | 1            |
| 102       | Priya | 2            |
| 103       | Amit  | 1            |
| 104       | Sneha | 3            |

---

## Departments

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |
| 3            | EEE            |

---

Suppose your manager asks:

> "Show every student's name along with their department name."

Can you do it using only one table?

❌ No.

The student's name is in **Students**.

The department name is in **Departments**.

We need to combine both tables.

That is exactly what **JOIN** does.

---

# Types of JOINs

```
JOINs
│
├── INNER JOIN ⭐⭐⭐⭐⭐
│
├── LEFT JOIN ⭐⭐⭐⭐⭐
│
├── RIGHT JOIN ⭐⭐⭐⭐
│
├── FULL OUTER JOIN ⭐⭐⭐
│
├── CROSS JOIN ⭐⭐
│
└── SELF JOIN ⭐⭐⭐⭐
```

We'll learn them in this exact order.

---

# Step 1: INNER JOIN

This is the most commonly used JOIN.

## Definition

> Returns **only the rows that have matching values in both tables**.

---

### Syntax

```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.column = table2.column;
```

---

### Example

Students

| StudentID | Name  | DepartmentID |
| --------- | ----- | ------------ |
| 101       | Rahul | 1            |
| 102       | Priya | 2            |
| 103       | Amit  | 1            |
| 104       | Sneha | 3            |

Departments

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |
| 3            | EEE            |

Query

```sql
SELECT
    s.StudentID,
    s.Name,
    d.DepartmentName
FROM Students s
INNER JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

Output

| StudentID | Name  | Department |
| --------- | ----- | ---------- |
| 101       | Rahul | CSE        |
| 102       | Priya | ECE        |
| 103       | Amit  | CSE        |
| 104       | Sneha | EEE        |

Notice how SQL matched:

```
Students.DepartmentID
        =
Departments.DepartmentID
```

---

# Table Aliases

Instead of writing

```sql
Students.DepartmentID
```

we write

```sql
s.DepartmentID
```

because

```sql
FROM Students s
```

means

```
Students → s
```

Similarly,

```sql
Departments → d
```

This makes queries much cleaner.

---

# Visual Representation

```
Students                Departments

101 Rahul ----1-------> 1 CSE
102 Priya ----2-------> 2 ECE
103 Amit  ----1-------> 1 CSE
104 Sneha ----3-------> 3 EEE
```

SQL follows the matching IDs and combines the rows.

---

# Practice Setup

Create these tables.

### Departments

```sql
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(50)
);
```

Insert data

```sql
INSERT INTO Departments VALUES
(1,'CSE'),
(2,'ECE'),
(3,'EEE'),
(4,'Mechanical');
```

---

### Students

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(50),
    DepartmentID INT
);
```

Insert data

```sql
INSERT INTO Students VALUES
(101,'Rahul',1),
(102,'Priya',2),
(103,'Amit',1),
(104,'Sneha',3),
(105,'Rohit',2);
```

---

# Practice Questions (INNER JOIN)

### Q1

Display all students with their department names.

---

### Q2

Display only `Name` and `DepartmentName`.

---

### Q3

Display students belonging to the **CSE** department.

---

### Q4

Display students from the **ECE** department.

---

### Q5

Display students whose `DepartmentID = 1` along with department names.

---

### Q6

Sort the results by student name.

---

### Q7

Display only `StudentID`, `Name`, and `DepartmentName`.

---

### Q8

Count the number of students in each department.

> *(This combines `JOIN` + `GROUP BY`.)*

---

### Q9

Find the department that has the highest number of students.

---

### Q10 (Interview Question)

Display all students along with their department names in alphabetical order of the department.

---

# What You'll Learn Next

Once you've completed these, we'll intentionally add **unmatched records** to the tables.

That will help you understand:

* **INNER JOIN** (only matching rows)
* **LEFT JOIN** (all left rows + matching right rows)
* **RIGHT JOIN** (all right rows + matching left rows)
* **FULL OUTER JOIN** (everything)

This visual approach makes JOINs much easier to understand than memorizing definitions.


Perfect! Now comes the topic that confuses most beginners but becomes very easy once you **see** it.

# Phase 4.2: LEFT JOIN

---

# Why LEFT JOIN?

Let's modify our data first.

## Students (Left Table)

| StudentID | Name  | DepartmentID |                            |
| --------- | ----- | ------------ | -------------------------- |
| 101       | Rahul | 1            |                            |
| 102       | Priya | 2            |                            |
| 103       | Amit  | 1            |                            |
| 104       | Sneha | 3            |                            |
| 105       | Rohit | 2            |                            |
| 106       | Anuj  | 5            | ← Department doesn't exist |

---

## Departments (Right Table)

| DepartmentID | DepartmentName |               |
| ------------ | -------------- | ------------- |
| 1            | CSE            |               |
| 2            | ECE            |               |
| 3            | EEE            |               |
| 4            | Mechanical     | ← No students |

Notice two special cases:

* Student **Anuj** belongs to DepartmentID **5**, which doesn't exist.
* **Mechanical** department has no students.

---

# What does LEFT JOIN do?

**LEFT JOIN returns:**

* ✅ All rows from the **left table**
* ✅ Matching rows from the **right table**
* ✅ If there's no match, it returns `NULL` for the right table columns

---

## Syntax

```sql
SELECT columns
FROM Students s
LEFT JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

---

## Result

| StudentID | Name  | DepartmentName |
| --------- | ----- | -------------- |
| 101       | Rahul | CSE            |
| 102       | Priya | ECE            |
| 103       | Amit  | CSE            |
| 104       | Sneha | EEE            |
| 105       | Rohit | ECE            |
| 106       | Anuj  | NULL           |

Notice:

* Rahul → Match ✅
* Priya → Match ✅
* Amit → Match ✅
* Sneha → Match ✅
* Rohit → Match ✅
* **Anuj → No matching department → NULL**

But notice something else...

Mechanical department **does not appear** because LEFT JOIN only guarantees all rows from the **left** table.

---

# Visual Representation

```
Students (LEFT)           Departments (RIGHT)

Rahul   ------> CSE
Priya   ------> ECE
Amit    ------> CSE
Sneha   ------> EEE
Rohit   ------> ECE
Anuj    ------> ❌ No Match

Output:
Rahul   CSE
Priya   ECE
Amit    CSE
Sneha   EEE
Rohit   ECE
Anuj    NULL
```

---

# INNER JOIN vs LEFT JOIN

### INNER JOIN

Only matching rows.

```
Rahul
Priya
Amit
Sneha
Rohit
```

Anuj disappears.

---

### LEFT JOIN

Keeps everyone from Students.

```
Rahul
Priya
Amit
Sneha
Rohit
Anuj NULL
```

---

# Finding "Missing" Records

One of the biggest real-world uses of `LEFT JOIN` is finding records with **no match**.

### Find students who don't belong to any department.

```sql
SELECT
    s.StudentID,
    s.Name
FROM Students s
LEFT JOIN Departments d
ON s.DepartmentID = d.DepartmentID
WHERE d.DepartmentID IS NULL;
```

Output

| StudentID | Name |
| --------- | ---- |
| 106       | Anuj |

This pattern is **very common in interviews**.

---

# Another Example

Suppose:

### Customers

| CustomerID | Name  |
| ---------- | ----- |
| 1          | Rahul |
| 2          | Priya |
| 3          | Amit  |

---

### Orders

| OrderID | CustomerID |
| ------- | ---------- |
| 11      | 1          |
| 12      | 1          |
| 13      | 2          |

Question:

> Find customers who never placed an order.

```sql
SELECT c.Name
FROM Customers c
LEFT JOIN Orders o
ON c.CustomerID = o.CustomerID
WHERE o.CustomerID IS NULL;
```

Output

```
Amit
```

This exact pattern appears frequently in SQL interviews.

---

# Practice Questions

### Q1

Display every student with their department name.

---

### Q2

Display students who don't belong to any department.

---

### Q3

Display only student names and department names.

---

### Q4

Display students ordered by department name.

---

### Q5

Count students in each department using `LEFT JOIN`.

---

### Q6

Find departments that have **no students**.

**Hint:** This time, think about **which table should be on the left**.

---

### Q7

Display all students including those whose department is missing.

---

### Q8

Display students whose department name is `NULL`.

---

### Q9 (Interview)

Find all orphan records (students with invalid department IDs).

---

### Q10 (Interview)

Explain the difference between `INNER JOIN` and `LEFT JOIN` using the `Students` and `Departments` tables.

---

# ⭐ Interview Tip

A very common interview question is:

> **When should you use LEFT JOIN instead of INNER JOIN?**

A strong answer is:

> Use `LEFT JOIN` when you want **all rows from the left table**, even if there is no matching row in the right table. It's commonly used to find missing relationships, such as customers with no orders or students without valid departments.

---

## Next Lesson

After you've practiced these, we'll move to:

# **RIGHT JOIN**

You'll immediately notice it's just the mirror image of `LEFT JOIN`, and then we'll compare all four major joins (`INNER`, `LEFT`, `RIGHT`, `FULL OUTER`) side by side so they're easy to remember.


Excellent! You're doing great. Now let's move to **RIGHT JOIN**.

---

# Phase 4.3: RIGHT JOIN

## What is RIGHT JOIN?

A **RIGHT JOIN** returns:

* ✅ All rows from the **right table**
* ✅ Matching rows from the **left table**
* ✅ If there's no match in the left table, it returns `NULL` for the left table columns

Think of it as the opposite of `LEFT JOIN`.

---

## Same Data

### Students (Left Table)

| StudentID | Name  | DepartmentID |
| --------- | ----- | ------------ |
| 101       | Rahul | 1            |
| 102       | Priya | 2            |
| 103       | Amit  | 1            |
| 104       | Sneha | 3            |
| 105       | Rohit | 2            |
| 106       | Anuj  | 5            |

### Departments (Right Table)

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |
| 3            | EEE            |
| 4            | Mechanical     |

Notice:

* Department **Mechanical** has no students.
* Student **Anuj** belongs to a non-existent department.

---

# Syntax

```sql
SELECT columns
FROM Students s
RIGHT JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

---

# Result

| StudentID | Name  | DepartmentName |
| --------- | ----- | -------------- |
| 101       | Rahul | CSE            |
| 103       | Amit  | CSE            |
| 102       | Priya | ECE            |
| 105       | Rohit | ECE            |
| 104       | Sneha | EEE            |
| NULL      | NULL  | Mechanical     |

Notice:

* Mechanical appears even though it has no students.
* Anuj disappears because RIGHT JOIN only guarantees rows from the **right** table.

---

# Visual Representation

```text
Students               Departments

Rahul  ---------> CSE
Amit   ---------> CSE
Priya  ---------> ECE
Rohit  ---------> ECE
Sneha  ---------> EEE

Anuj   ----X

                 Mechanical
                     ↑
                 No Student
```

Output includes Mechanical with NULL student information.

---

# LEFT JOIN vs RIGHT JOIN

### LEFT JOIN

Keeps every student.

```text
Rahul
Priya
Amit
Sneha
Rohit
Anuj NULL
```

---

### RIGHT JOIN

Keeps every department.

```text
CSE
ECE
EEE
Mechanical NULL
```

---

# Finding Departments Without Students

This is a common interview question.

```sql
SELECT
    d.DepartmentID,
    d.DepartmentName
FROM Students s
RIGHT JOIN Departments d
ON s.DepartmentID = d.DepartmentID
WHERE s.StudentID IS NULL;
```

Output

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 4            | Mechanical     |

---

# Easier Alternative

Many developers prefer using a `LEFT JOIN` from the other direction because it's easier to read.

Instead of:

```sql
Students
RIGHT JOIN Departments
```

they write:

```sql
Departments
LEFT JOIN Students
```

Both return the same result.

```sql
SELECT
    d.DepartmentName
FROM Departments d
LEFT JOIN Students s
ON d.DepartmentID = s.DepartmentID
WHERE s.StudentID IS NULL;
```

You'll see this style more often in production code.

---

# Practice Questions

### Q1

Display every department with its students.

---

### Q2

Display departments that have no students.

---

### Q3

Display department names and student names.

---

### Q4

Display all departments ordered alphabetically.

---

### Q5

Count students in every department.

---

### Q6

Display departments with zero students.

---

### Q7

Display department names even if there are no students.

---

### Q8

Find departments whose student information is `NULL`.

---

### Q9 (Interview)

Find all departments that currently have no students.

---

### Q10 (Interview)

Explain the difference between `LEFT JOIN` and `RIGHT JOIN`.

---

# Quick Comparison

| JOIN       | Keeps All Rows From |
| ---------- | ------------------- |
| INNER JOIN | Only matching rows  |
| LEFT JOIN  | Left table          |
| RIGHT JOIN | Right table         |

---

# Interview Tip

Many companies (and many SQL developers) **rarely use `RIGHT JOIN`**.

Instead of writing:

```sql
Students
RIGHT JOIN Departments
```

they usually write:

```sql
Departments
LEFT JOIN Students
```

The result is the same, but many people find it easier to read because the table they want to keep is placed first.

---

# Next Topic: FULL OUTER JOIN

This is where everything comes together.

`FULL OUTER JOIN` returns:

* ✅ All rows from the left table
* ✅ All rows from the right table
* ✅ Matching rows combined
* ✅ Non-matching rows filled with `NULL`

It's the union of `LEFT JOIN` and `RIGHT JOIN` and is another favorite interview topic.


Awesome! You're now at one of the last major JOIN types.

---

# Phase 4.4: FULL OUTER JOIN

## What is FULL OUTER JOIN?

A **FULL OUTER JOIN** returns:

* ✅ All matching rows
* ✅ All rows from the left table
* ✅ All rows from the right table
* ✅ `NULL` wherever a match doesn't exist

Think of it as:

```text
LEFT JOIN
      +
RIGHT JOIN
----------------
FULL OUTER JOIN
```

---

## Example Data

### Students

| StudentID | Name  | DepartmentID |
| --------- | ----- | ------------ |
| 101       | Rahul | 1            |
| 102       | Priya | 2            |
| 103       | Amit  | 1            |
| 104       | Sneha | 3            |
| 105       | Rohit | 2            |
| 106       | Anuj  | 5            |

---

### Departments

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |
| 3            | EEE            |
| 4            | Mechanical     |

Notice:

* Student **Anuj** belongs to department **5**, which doesn't exist.
* Department **Mechanical** has no students.

---

# Syntax

```sql
SELECT columns
FROM Students s
FULL OUTER JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

---

# Result

| StudentID | Name  | DepartmentName |
| --------- | ----- | -------------- |
| 101       | Rahul | CSE            |
| 103       | Amit  | CSE            |
| 102       | Priya | ECE            |
| 105       | Rohit | ECE            |
| 104       | Sneha | EEE            |
| 106       | Anuj  | NULL           |
| NULL      | NULL  | Mechanical     |

Notice:

✔ Matching records appear normally.

✔ Anuj is kept.

✔ Mechanical is also kept.

Nothing is lost.

---

# Visual Representation

```text
Students                Departments

Rahul  ------> CSE
Amit   ------> CSE
Priya  ------> ECE
Rohit  ------> ECE
Sneha  ------> EEE

Anuj   ----X

                 Mechanical
                     ↑
                 No Student
```

FULL JOIN returns **everything**.

---

# INNER vs LEFT vs RIGHT vs FULL

## INNER JOIN

```text
Rahul
Amit
Priya
Rohit
Sneha
```

Only matches.

---

## LEFT JOIN

```text
Rahul
Amit
Priya
Rohit
Sneha
Anuj NULL
```

Keeps every student.

---

## RIGHT JOIN

```text
Rahul
Amit
Priya
Rohit
Sneha

Mechanical NULL
```

Keeps every department.

---

## FULL OUTER JOIN

```text
Rahul
Amit
Priya
Rohit
Sneha
Anuj
Mechanical
```

Keeps everything.

---

# Venn Diagram (Easy to Remember)

```text
INNER JOIN

      (A ∩ B)



LEFT JOIN

      A ∩ B
 + Left Only



RIGHT JOIN

      A ∩ B
         + Right Only



FULL OUTER JOIN

Left Only
     +
 A ∩ B
     +
Right Only
```

---

# Finding Unmatched Records

One useful trick is to find rows that **don't have a match** on either side.

```sql
SELECT
    s.StudentID,
    s.Name,
    d.DepartmentName
FROM Students s
FULL OUTER JOIN Departments d
ON s.DepartmentID = d.DepartmentID
WHERE s.StudentID IS NULL
   OR d.DepartmentID IS NULL;
```

Output

| Student | Department |
| ------- | ---------- |
| Anuj    | NULL       |
| NULL    | Mechanical |

This shows all unmatched rows.

---

# Database Support

Not every database supports `FULL OUTER JOIN`.

| Database   | Supported |
| ---------- | --------- |
| PostgreSQL | ✅         |
| SQL Server | ✅         |
| Oracle     | ✅         |
| SQLite     | ❌         |
| MySQL      | ❌         |

---

## FULL OUTER JOIN in MySQL

MySQL doesn't support it directly.

Instead, combine a `LEFT JOIN` and a `RIGHT JOIN` using `UNION`.

```sql
SELECT *
FROM Students s
LEFT JOIN Departments d
ON s.DepartmentID = d.DepartmentID

UNION

SELECT *
FROM Students s
RIGHT JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

---

# Practice Questions

### Q1

Display every student and every department.

---

### Q2

Display all matching and non-matching records.

---

### Q3

Find students without departments and departments without students.

---

### Q4

Display all department names and student names.

---

### Q5

Count total rows returned by the FULL OUTER JOIN.

---

### Q6

Display only unmatched records.

---

### Q7

Display departments with or without students.

---

### Q8

Display students with or without departments.

---

### Q9 (Interview)

Explain the difference between `LEFT JOIN` and `FULL OUTER JOIN`.

---

### Q10 (Interview)

When would you use a `FULL OUTER JOIN`?

Example answer:

> Use a `FULL OUTER JOIN` when you need a complete comparison of two tables, including matched rows and unmatched rows from both sides, such as data reconciliation or identifying missing relationships.

---

# JOIN Summary So Far

| JOIN            | Returns                             |
| --------------- | ----------------------------------- |
| INNER JOIN      | Only matching rows                  |
| LEFT JOIN       | All left rows + matching right rows |
| RIGHT JOIN      | All right rows + matching left rows |
| FULL OUTER JOIN | All rows from both tables           |

---

# What's Next?

The remaining JOIN topics are:

1. ✅ INNER JOIN
2. ✅ LEFT JOIN
3. ✅ RIGHT JOIN
4. ✅ FULL OUTER JOIN
5. ⏳ **CROSS JOIN**
6. ⏳ **SELF JOIN**
7. ⏳ **Multi-table JOINs**
8. ⏳ **Advanced JOIN interview problems**

These last topics are shorter but very important for interview readiness. After that, you'll have covered all the core SQL JOIN concepts.


Awesome! You're almost done with **all SQL JOINs**. The next one is the easiest to understand.

---

# Phase 4.5: CROSS JOIN

## What is CROSS JOIN?

A **CROSS JOIN** returns **every possible combination** of rows from two tables.

It **doesn't require a matching column**.

### Formula

```text
Total Rows = Rows in Table A × Rows in Table B
```

---

## Example

### Students

| StudentID | Name  |
| --------- | ----- |
| 101       | Rahul |
| 102       | Priya |
| 103       | Amit  |

(3 rows)

---

### Subjects

| SubjectID | Subject |
| --------- | ------- |
| 1         | Math    |
| 2         | Science |

(2 rows)

---

## Query

```sql
SELECT
    s.Name,
    sub.Subject
FROM Students s
CROSS JOIN Subjects sub;
```

---

## Output

| Name  | Subject |
| ----- | ------- |
| Rahul | Math    |
| Rahul | Science |
| Priya | Math    |
| Priya | Science |
| Amit  | Math    |
| Amit  | Science |

Rows returned:

```text
3 × 2 = 6 rows
```

Every student is paired with every subject.

---

# Visual Representation

```text
Students          Subjects

Rahul   ×  Math
        ×  Science

Priya   ×  Math
        ×  Science

Amit    ×  Math
        ×  Science
```

---

# Another Example

### Colors

| Color |
| ----- |
| Red   |
| Blue  |

---

### Sizes

| Size |
| ---- |
| S    |
| M    |
| L    |

Query

```sql
SELECT
    c.Color,
    s.Size
FROM Colors c
CROSS JOIN Sizes s;
```

Output

| Color | Size |
| ----- | ---- |
| Red   | S    |
| Red   | M    |
| Red   | L    |
| Blue  | S    |
| Blue  | M    |
| Blue  | L    |

2 × 3 = **6 rows**

---

# Syntax

```sql
SELECT columns
FROM table1
CROSS JOIN table2;
```

No `ON` clause is needed.

---

# CROSS JOIN vs INNER JOIN

### INNER JOIN

Requires matching values.

```sql
SELECT *
FROM Students s
INNER JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

Only matching records are returned.

---

### CROSS JOIN

No matching condition.

```sql
SELECT *
FROM Students
CROSS JOIN Departments;
```

Every student combines with every department.

If there are:

* 10 students
* 4 departments

Result:

```text
10 × 4 = 40 rows
```

---

# Implicit CROSS JOIN

These two queries are equivalent:

```sql
SELECT *
FROM Students
CROSS JOIN Departments;
```

and

```sql
SELECT *
FROM Students, Departments;
```

The explicit `CROSS JOIN` syntax is preferred because it's clearer.

---

# Real-World Uses

Although not used as often as `INNER JOIN`, `LEFT JOIN`, or `RIGHT JOIN`, `CROSS JOIN` is useful for:

* Generating all combinations (e.g., product sizes and colors)
* Building calendars
* Creating test data
* Scheduling problems
* Matrix-style reports

---

# Practice Questions

### Q1

Display every student with every department.

---

### Q2

Create a `Subjects` table with 3 subjects and display every student with every subject.

---

### Q3

Create a `Days` table (`Monday`, `Tuesday`, `Wednesday`) and pair every student with every day.

---

### Q4

If there are 8 students and 5 departments, how many rows will a `CROSS JOIN` return?

---

### Q5

If Table A has 12 rows and Table B has 7 rows, how many rows will the result contain?

---

### Q6 (Interview)

What is the difference between `INNER JOIN` and `CROSS JOIN`?

**Answer:**

* `INNER JOIN` returns only matching rows based on a condition.
* `CROSS JOIN` returns every possible combination of rows and does not use an `ON` condition.

---

# JOIN Summary So Far

| JOIN              | Returns                             |
| ----------------- | ----------------------------------- |
| ✅ INNER JOIN      | Only matching rows                  |
| ✅ LEFT JOIN       | All left rows + matching right rows |
| ✅ RIGHT JOIN      | All right rows + matching left rows |
| ✅ FULL OUTER JOIN | All rows from both tables           |
| ✅ CROSS JOIN      | Every possible combination          |

---

# Next Topic: SELF JOIN

A **SELF JOIN** is where a table is joined **with itself**. It's commonly used for hierarchical data such as:

* Employees and their managers
* Categories and parent categories
* Friends in a social network
* Family relationships

After `SELF JOIN`, we'll cover **multi-table JOINs** and then move into advanced SQL topics like **subqueries, CTEs, views, indexes, and window functions**, which are heavily used in production systems and technical interviews.


Yes, for **SQL fundamentals**, you've covered the major JOIN types. There are a couple of advanced JOIN-related topics (like `SELF JOIN`, `NATURAL JOIN`, and multi-table joins), but you don't need them immediately. We can revisit them later when doing advanced SQL.

## 🎉 Current Progress

### ✅ Phase 1 — DML

* Create Table
* Insert
* Select
* Update
* Delete

### ✅ Phase 2 — DDL

* ALTER TABLE
* Constraints
* Rename
* Drop
* Truncate

### ✅ Phase 3 — Querying

* WHERE
* Operators
* IN / BETWEEN / LIKE
* ORDER BY
* LIMIT / OFFSET
* Aggregate Functions
* GROUP BY
* HAVING

### ✅ Phase 4 — JOINs

* INNER JOIN
* LEFT JOIN
* RIGHT JOIN
* FULL OUTER JOIN
* CROSS JOIN

---

# 🚀 Phase 5 — Intermediate SQL

This is where SQL becomes powerful. I recommend this order:

| Step | Topic                                      | Importance | Interview |
| ---- | ------------------------------------------ | ---------- | --------- |
| 1    | Subqueries                                 | ⭐⭐⭐⭐⭐      | ✅         |
| 2    | Correlated Subqueries                      | ⭐⭐⭐⭐       | ✅         |
| 3    | EXISTS / NOT EXISTS                        | ⭐⭐⭐⭐⭐      | ✅         |
| 4    | Common Table Expressions (CTE)             | ⭐⭐⭐⭐⭐      | ✅         |
| 5    | Window Functions                           | ⭐⭐⭐⭐⭐      | ✅         |
| 6    | Views                                      | ⭐⭐⭐⭐       | ✅         |
| 7    | Indexes                                    | ⭐⭐⭐⭐⭐      | ✅         |
| 8    | Transactions (COMMIT, ROLLBACK, SAVEPOINT) | ⭐⭐⭐⭐⭐      | ✅         |
| 9    | Stored Procedures & Functions              | ⭐⭐⭐        | Optional  |
| 10   | Triggers                                   | ⭐⭐⭐        | Optional  |

---

# 📌 My Recommended Order

Don't jump directly to window functions.

Follow this sequence:

```
Subqueries
        ↓
Correlated Subqueries
        ↓
EXISTS / NOT EXISTS
        ↓
CTEs
        ↓
Window Functions
        ↓
Views
        ↓
Indexes
        ↓
Transactions
```

This is the order most SQL developers naturally learn because each topic builds on the previous one.

---

# ⭐ Next Topic: Subqueries

A **subquery** is simply a query inside another query.

Example:

```sql
SELECT *
FROM student
WHERE mark = (
    SELECT MAX(mark)
    FROM student
);
```

Execution:

```
SELECT MAX(mark)
↓

99

↓

SELECT *
FROM student
WHERE mark = 99;
```

Output:

```
Rudra    99
```

This is much more powerful than manually writing:

```sql
WHERE mark = 99;
```

because the maximum mark can change over time.

---

# What You'll Learn in the Subquery Module

* Subqueries in `WHERE`
* Subqueries in `FROM`
* Subqueries in `SELECT`
* Single-row vs Multi-row subqueries
* `IN`
* `ANY`
* `ALL`
* Nested subqueries
* Real interview questions

---

## 🎯 Roadmap After Phase 5

```
SQL Fundamentals          ✅
        ↓
Intermediate SQL          ← Next
        ↓
Advanced SQL
(Window Functions)
        ↓
Performance
(Indexes)
        ↓
Transactions
        ↓
Database Design
        ↓
Normalization
        ↓
SQL Interview Problems
(100+)
```

### I recommend we start with **Subqueries** next.

It's one of the most frequently asked SQL interview topics and forms the foundation for CTEs and many advanced query patterns.



Perfect! Now you're entering **Intermediate SQL**. This is where SQL starts feeling like a programming language rather than just a query language.

---

# Phase 5.1: Subqueries

## What is a Subquery?

A **subquery** is a query written **inside another SQL query**.

Think of it like a function call in programming.

```text
Outer Query
    |
    +--> Inner Query (Subquery)
```

The **inner query executes first**, and its result is used by the outer query.

---

# Syntax

```sql
SELECT column_name
FROM table_name
WHERE column_name = (
    SELECT ...
);
```

---

## Our Table

We'll use your existing `student` table.

```text
id | name | age | mark | city
```

---

# Example 1: Student with Highest Marks

Without a subquery:

```sql
SELECT *
FROM student
WHERE mark = 99;
```

Problem:

What if tomorrow someone scores 100?

The query becomes incorrect.

---

Using a subquery:

```sql
SELECT *
FROM student
WHERE mark = (
    SELECT MAX(mark)
    FROM student
);
```

Execution:

```text
Step 1

SELECT MAX(mark)
FROM student;

↓

99

↓

Step 2

SELECT *
FROM student
WHERE mark = 99;
```

Now the query always returns the highest scorer.

---

# Example 2: Student with Lowest Marks

```sql
SELECT *
FROM student
WHERE mark = (
    SELECT MIN(mark)
    FROM student
);
```

---

# Example 3: Above Average Students

```sql
SELECT *
FROM student
WHERE mark > (
    SELECT AVG(mark)
    FROM student
);
```

Suppose

Average = 81.4

SQL executes:

```sql
WHERE mark > 81.4
```

---

# Example 4: Youngest Students

```sql
SELECT *
FROM student
WHERE age = (
    SELECT MIN(age)
    FROM student
);
```

---

# Example 5: Oldest Students

```sql
SELECT *
FROM student
WHERE age = (
    SELECT MAX(age)
    FROM student
);
```

---

# Why Use Subqueries?

Imagine your manager asks:

> Show all students whose marks are above the class average.

Without subqueries:

1. Find average.
2. Copy result.
3. Write another query.

Two steps.

With a subquery:

One query.

```sql
SELECT *
FROM student
WHERE mark >
(
    SELECT AVG(mark)
    FROM student
);
```

---

# Types of Subqueries

There are four major types.

```text
Subqueries
│
├── Single Row ⭐⭐⭐⭐⭐
├── Multiple Row ⭐⭐⭐⭐⭐
├── Correlated ⭐⭐⭐⭐⭐
└── Nested ⭐⭐⭐⭐
```

Today we'll focus on **Single Row Subqueries**.

---

# Single Row Subquery

Returns exactly one value.

Examples:

```sql
SELECT MAX(mark)
FROM student;
```

returns

```text
99
```

Only one value.

---

Another:

```sql
SELECT AVG(mark)
FROM student;
```

returns

```text
81.4
```

Again, one value.

---

# Operators Used

Single-row subqueries commonly use:

```text
=
>
<
>=
<=
<>
```

Example:

```sql
SELECT *
FROM student
WHERE mark >
(
SELECT AVG(mark)
FROM student
);
```

---

# Execution Order

```text
Outer Query

↓

Need Subquery

↓

Run Subquery

↓

Return Result

↓

Run Outer Query
```

Always remember:

> **The subquery executes first.**

---

# Common Mistakes

### ❌ Wrong

```sql
SELECT *
FROM student
WHERE mark =
SELECT MAX(mark)
FROM student;
```

Missing parentheses.

Correct:

```sql
SELECT *
FROM student
WHERE mark =
(
SELECT MAX(mark)
FROM student
);
```

---

### ❌ Wrong

Using a subquery that returns multiple rows with `=`.

Example:

```sql
WHERE city =
(
SELECT city
FROM student
);
```

This will fail because the subquery returns many cities.

We'll learn how to handle that using `IN`, `ANY`, and `ALL` next.

---

# Practice Questions

### Level 1

1. Find the student with the highest marks.
2. Find the student with the lowest marks.
3. Find students whose marks are above the average.
4. Find students whose marks are below the average.
5. Find the oldest student.

---

### Level 2

6. Find the youngest student.
7. Find students whose age is greater than the average age.
8. Find students whose age is less than the average age.
9. Find students having the maximum age.
10. Find students having the minimum age.

---

### Level 3

11. Find students whose marks equal the highest marks.
12. Find students whose marks equal the lowest marks.
13. Find students whose age equals the average age (if any).
14. Find students whose marks are greater than the average marks but less than the maximum marks.
15. Find students whose marks are between the average and maximum marks.

---

### Challenge Questions

16. Find the second-highest marks **without using `LIMIT`** (we'll refine this later).

17. Find students who scored above the average in Hyderabad only.

18. Find students older than the youngest student.

19. Find students whose marks are less than the highest marks.

20. Find all students except the topper(s).

---

# Interview Questions

### Q1. What is a subquery?

A query inside another SQL query whose result is used by the outer query.

---

### Q2. Which query executes first?

**The inner query (subquery)** executes first.

---

### Q3. Can a subquery return multiple rows?

Yes. There are **single-row** and **multiple-row** subqueries. The operator you use (`=`, `IN`, `ANY`, `ALL`, etc.) depends on how many rows the subquery returns.

---

## 📍 Next Lesson

Once you've completed these, we'll move to **Multiple-Row Subqueries**, where you'll learn:

* `IN`
* `ANY`
* `ALL`
* `EXISTS`

These are heavily used in interview questions and production SQL.


Excellent! You're moving at a great pace. Now we'll cover **Multiple-Row Subqueries**, one of the most common SQL interview topics.

---

# Phase 5.2: Multiple-Row Subqueries

## What is a Multiple-Row Subquery?

Unlike a single-row subquery, which returns **one value**, a multiple-row subquery returns **multiple values**.

Example:

```sql
SELECT city
FROM student;
```

Output:

```text
Delhi
Mumbai
Pune
Delhi
Hyderabad
...
```

Since multiple rows are returned, using `=` will cause an error.

❌ Wrong

```sql
SELECT *
FROM student
WHERE city = (
    SELECT city
    FROM student
);
```

Error:

```text
Subquery returns more than one row
```

---

# Operators Used

| Operator       | Used For                                       |
| -------------- | ---------------------------------------------- |
| `IN`           | Match any value in the subquery result         |
| `NOT IN`       | Exclude values in the subquery result          |
| `ANY` / `SOME` | Compare with at least one returned value       |
| `ALL`          | Compare with every returned value              |
| `EXISTS`       | Check if the subquery returns at least one row |
| `NOT EXISTS`   | Check if the subquery returns no rows          |

We'll learn them one by one.

---

# 1. IN

Suppose we have another table:

## TopStudents

| StudentID |
| --------- |
| 101       |
| 104       |
| 107       |

Find the details of these students.

```sql
SELECT *
FROM student
WHERE id IN (
    SELECT StudentID
    FROM TopStudents
);
```

Execution:

```text
Subquery

↓

101
104
107

↓

Outer Query

WHERE id IN (101,104,107)
```

---

# 2. NOT IN

Find students who are **not** in the TopStudents table.

```sql
SELECT *
FROM student
WHERE id NOT IN (
    SELECT StudentID
    FROM TopStudents
);
```

---

# 3. ANY (or SOME)

`ANY` means:

> Compare with **at least one** value returned by the subquery.

Example:

```sql
SELECT *
FROM student
WHERE mark > ANY (
    SELECT mark
    FROM student
    WHERE city = 'Delhi'
);
```

Suppose Delhi marks are:

```text
70
85
90
```

SQL checks:

```text
mark > 70
OR
mark > 85
OR
mark > 90
```

If **any one** is true, the row is returned.

---

# 4. ALL

`ALL` means:

> Compare with **every** value returned by the subquery.

Example:

```sql
SELECT *
FROM student
WHERE mark > ALL (
    SELECT mark
    FROM student
    WHERE city='Delhi'
);
```

Delhi marks:

```text
70
85
90
```

Equivalent to:

```text
mark > 70
AND
mark > 85
AND
mark > 90
```

So only students scoring **above 90** are returned.

---

# ANY vs ALL

Suppose the subquery returns:

```text
50
70
90
```

| Condition | Meaning                                                 |
| --------- | ------------------------------------------------------- |
| `> ANY`   | Greater than at least one value (effectively > 50 here) |
| `> ALL`   | Greater than every value (greater than 90)              |
| `< ANY`   | Less than at least one value                            |
| `< ALL`   | Less than every value (less than 50)                    |

---

# EXISTS

`EXISTS` checks whether the subquery returns **at least one row**.

Example tables:

### Departments

| DepartmentID | DepartmentName |
| ------------ | -------------- |
| 1            | CSE            |
| 2            | ECE            |
| 3            | EEE            |

### Students

| Name  | DepartmentID |
| ----- | ------------ |
| Rahul | 1            |
| Priya | 2            |

Find departments that have students.

```sql
SELECT *
FROM Departments d
WHERE EXISTS (
    SELECT 1
    FROM Students s
    WHERE s.DepartmentID = d.DepartmentID
);
```

Result:

```text
CSE
ECE
```

EEE is excluded because no student belongs to it.

---

# NOT EXISTS

Find departments without students.

```sql
SELECT *
FROM Departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM Students s
    WHERE s.DepartmentID = d.DepartmentID
);
```

Output:

```text
EEE
```

This is an extremely common interview question.

---

# Execution Order

```text
Outer Query
      ↓
Subquery
      ↓
Result
      ↓
Outer Query Continues
```

---

# Practice Questions

## Using IN

1. Create a `TopStudents` table and display their details.
2. Find students whose IDs are in `TopStudents`.
3. Find students whose IDs are not in `TopStudents`.
4. Create a `PassedStudents` table and find all passed students.
5. Find students not in `PassedStudents`.

---

## Using ANY

6. Find students scoring more than **any** student from Delhi.
7. Find students younger than **any** student from Pune.
8. Find students scoring less than **any** student from Mumbai.

---

## Using ALL

9. Find students scoring more than **all** students from Delhi.
10. Find students younger than **all** students from Pune.
11. Find students scoring less than **all** students from Mumbai.

---

## Using EXISTS

12. Find departments having students.
13. Find departments without students.
14. Find cities having at least one student older than 22.
15. Find cities with no students above 90 marks.

---

# Interview Questions

### Q1. Difference between `IN` and `EXISTS`?

* `IN` compares a value against a list returned by a subquery.
* `EXISTS` only checks whether the subquery returns at least one row.

---

### Q2. Difference between `ANY` and `ALL`?

* `ANY` → condition must be true for **at least one** returned value.
* `ALL` → condition must be true for **every** returned value.

---

### Q3. When is `EXISTS` preferred?

For correlated subqueries and checking the existence of related records, especially with large datasets.

---

# Next Topic: Correlated Subqueries

This is where subqueries become much more powerful.

You'll learn queries like:

* Employees earning more than the average salary **in their own department**
* Students scoring above the average **in their own city**
* Customers with orders above the average **for their region**

Correlated subqueries are one of the most frequently asked SQL interview topics because they combine filtering, aggregation, and row-by-row evaluation.




Awesome! 🔥 You're now entering **Advanced SQL** territory.

Up to now, every subquery you've written has been **independent**. Now we'll learn subqueries that **depend on the current row** of the outer query.

---

# Phase 5.3: Correlated Subqueries

## What is a Correlated Subquery?

A **correlated subquery** is a subquery that references a column from the **outer query**.

Unlike a normal subquery:

* ✅ Normal subquery executes **once**
* ✅ Correlated subquery executes **once for every row** of the outer query

This is the biggest difference.

---

# Normal Subquery

Example:

```sql
SELECT *
FROM student
WHERE mark > (
    SELECT AVG(mark)
    FROM student
);
```

Execution:

```text
Average Marks

↓

81.5

↓

Outer Query Executes
```

The subquery runs **only one time**.

---

# Correlated Subquery

Suppose you want:

> Find students whose marks are **above the average marks of their own city**.

This cannot be solved with a simple subquery.

---

## Student Table

| id  | name  | city   | mark |
| --- | ----- | ------ | ---- |
| 101 | Rahul | Delhi  | 90   |
| 102 | Amit  | Delhi  | 70   |
| 103 | Priya | Mumbai | 95   |
| 104 | Rohit | Mumbai | 75   |
| 105 | Sneha | Pune   | 88   |

---

## Query

```sql
SELECT s1.*
FROM student s1
WHERE mark >
(
    SELECT AVG(s2.mark)
    FROM student s2
    WHERE s2.city = s1.city
);
```

Notice this line:

```sql
WHERE s2.city = s1.city
```

The subquery is using `s1.city` from the outer query.

That makes it a **correlated subquery**.

---

# Execution

For Rahul

```text
Outer Row

Rahul
Delhi

↓

Subquery

Average marks of Delhi

↓

80

↓

90 > 80

↓

Return Rahul
```

---

For Priya

```text
Outer Row

Priya
Mumbai

↓

Average of Mumbai

↓

85

↓

95 > 85

↓

Return Priya
```

---

For every student SQL repeats the process.

```text
Student1

↓

Run Subquery

↓

Student2

↓

Run Subquery

↓

Student3

↓

Run Subquery
```

This is why correlated subqueries are slower than normal subqueries on large datasets.

---

# Example 2

Find students older than the average age of their city.

```sql
SELECT s1.*
FROM student s1
WHERE age >
(
SELECT AVG(s2.age)
FROM student s2
WHERE s1.city=s2.city
);
```

---

# Example 3

Find students with the highest marks in each city.

```sql
SELECT *
FROM student s1
WHERE mark =
(
SELECT MAX(mark)
FROM student s2
WHERE s1.city=s2.city
);
```

Output

```text
Delhi → Rahul

Mumbai → Priya

Pune → Sneha
```

---

# Example 4

Find the youngest student in every city.

```sql
SELECT *
FROM student s1
WHERE age =
(
SELECT MIN(age)
FROM student s2
WHERE s1.city=s2.city
);
```

---

# EXISTS + Correlated Subquery

Suppose

Departments

```text
1 CSE
2 ECE
3 EEE
```

Students

```text
Rahul 1

Priya 2
```

Find departments having students.

```sql
SELECT *
FROM Departments d
WHERE EXISTS
(
SELECT 1
FROM Students s
WHERE s.DepartmentID=d.DepartmentID
);
```

This is also a correlated subquery because:

```sql
d.DepartmentID
```

comes from the outer query.

---

# NOT EXISTS

Find departments with no students.

```sql
SELECT *
FROM Departments d
WHERE NOT EXISTS
(
SELECT 1
FROM Students s
WHERE s.DepartmentID=d.DepartmentID
);
```

---

# Correlated vs Normal Subquery

| Normal        | Correlated                   |
| ------------- | ---------------------------- |
| Executes once | Executes for every outer row |
| Independent   | Depends on outer query       |
| Faster        | Usually slower               |
| Easier        | More powerful                |

---

# Practice Questions

## Level 1

1. Find students scoring above the average marks of their city.
2. Find students older than the average age of their city.
3. Find the highest scorer in each city.
4. Find the lowest scorer in each city.
5. Find the youngest student in every city.

---

## Level 2

6. Find the oldest student in every city.
7. Find students scoring above their city's average and older than the city's average age.
8. Find students with the minimum age in each city.
9. Find students having maximum marks in their age group.
10. Find students whose marks are below their city's average.

---

## Level 3

Using **Students** and **Departments** tables:

11. Find departments that have students.
12. Find departments that have no students.
13. Find students whose department exists.
14. Find students whose department does not exist.
15. Find departments having more than one student (combine with `GROUP BY`).

---

# Interview Questions

### Q1. What is a correlated subquery?

A subquery that references columns from the outer query and executes once for each row processed by the outer query.

---

### Q2. Which is faster?

A normal subquery is generally faster because it executes once. A correlated subquery may execute many times, so it can be slower unless the optimizer rewrites it.

---

### Q3. Can a correlated subquery be replaced?

Often, **yes**. Many correlated subqueries can be rewritten using:

* `JOIN`
* `GROUP BY`
* `CTE`
* Window functions (`ROW_NUMBER()`, `RANK()`, etc.)

These alternatives are often more efficient and easier to maintain.

---

# 📚 What Comes Next?

You have now learned:

* ✅ Single-row subqueries
* ✅ Multiple-row subqueries
* ✅ Correlated subqueries

The next topic is one of the most important features in modern SQL:

## **Phase 5.4: Common Table Expressions (CTEs)**

You'll learn:

* `WITH` clause
* Writing readable SQL
* Reusing intermediate results
* Multiple CTEs
* Recursive CTEs
* Replacing deeply nested subqueries

CTEs are heavily used in production code and are a favorite topic in SQL interviews because they make complex queries much easier to read and maintain.



Perfect! You're now at one of the most important SQL features used in production code.

# Phase 5.4: Common Table Expressions (CTE)

> **CTE = Temporary named result set that exists only during the execution of a single query.**

Think of it as creating a **temporary table** for one query.

---

# Why Do We Need CTEs?

Suppose you want to find students whose marks are above the average.

Without a CTE:

```sql
SELECT *
FROM student
WHERE mark >
(
    SELECT AVG(mark)
    FROM student
);
```

Works fine.

But imagine a much larger query with 5–6 nested subqueries.

It quickly becomes hard to read.

---

## Using a CTE

```sql
WITH AverageMarks AS
(
    SELECT AVG(mark) AS avg_mark
    FROM student
)

SELECT *
FROM student
WHERE mark >
(
    SELECT avg_mark
    FROM AverageMarks
);
```

Much more readable.

---

# Syntax

```sql
WITH cte_name AS
(
    SELECT ...
)

SELECT *
FROM cte_name;
```

---

# Example 1

Calculate average marks.

```sql
WITH AvgMarks AS
(
    SELECT AVG(mark) AS Average
    FROM student
)

SELECT *
FROM AvgMarks;
```

Output

| Average |
| ------- |
| 81.5    |

---

# Example 2

Students above average

```sql
WITH AvgMarks AS
(
SELECT AVG(mark) AS Average
FROM student
)

SELECT *
FROM student
WHERE mark >
(
SELECT Average
FROM AvgMarks
);
```

---

# Example 3

Students below average

```sql
WITH AvgMarks AS
(
SELECT AVG(mark) Average
FROM student
)

SELECT *
FROM student
WHERE mark <
(
SELECT Average
FROM AvgMarks
);
```

---

# Example 4

City-wise Average

```sql
WITH CityAverage AS
(
SELECT city,
AVG(mark) AvgMarks
FROM student
GROUP BY city
)

SELECT *
FROM CityAverage;
```

Output

| City   | AvgMarks |
| ------ | -------- |
| Delhi  | 80       |
| Mumbai | 85       |
| Pune   | 88       |

Notice that the CTE behaves like a temporary table.

---

# Joining a CTE

```sql
WITH CityAverage AS
(
SELECT city,
AVG(mark) AvgMarks
FROM student
GROUP BY city
)

SELECT
s.name,
s.city,
s.mark,
c.AvgMarks
FROM student s
JOIN CityAverage c
ON s.city=c.city;
```

Output

| Name  | City  | Mark | AvgMarks |
| ----- | ----- | ---- | -------- |
| Rahul | Delhi | 90   | 80       |
| Amit  | Delhi | 70   | 80       |

---

# Multiple CTEs

You can create more than one.

```sql
WITH

AvgMarks AS
(
SELECT AVG(mark) AvgMark
FROM student
),

MaxMarks AS
(
SELECT MAX(mark) MaxMark
FROM student
)

SELECT
AvgMark,
MaxMark
FROM AvgMarks,
MaxMarks;
```

---

# CTE vs Subquery

## Subquery

```sql
SELECT *
FROM student
WHERE mark >
(
SELECT AVG(mark)
FROM student
);
```

---

## CTE

```sql
WITH AvgMarks AS
(
SELECT AVG(mark) Average
FROM student
)

SELECT *
FROM student
WHERE mark >
(
SELECT Average
FROM AvgMarks
);
```

Both produce the same result.

The CTE is often easier to read and maintain, especially as queries become more complex.

---

# Recursive CTE (Introduction)

Recursive CTEs are used for:

* Employee hierarchy
* Folder structure
* Family tree
* Category hierarchy

Example structure:

```sql
WITH RECURSIVE Numbers AS
(
SELECT 1 AS n

UNION ALL

SELECT n+1
FROM Numbers
WHERE n<10
)

SELECT *
FROM Numbers;
```

Output

```text
1
2
3
4
5
6
7
8
9
10
```

We'll cover recursion in depth later.

---

# Execution Order

```text
WITH

↓

CTE Executes

↓

Temporary Result Created

↓

Main Query Executes

↓

CTE Disappears
```

Remember:

A CTE exists **only for the current SQL statement**.

---

# Practice Questions

## Level 1

1. Create a CTE to calculate average marks.
2. Display students above average.
3. Display students below average.
4. Create a CTE for maximum marks.
5. Display the topper using the CTE.

---

## Level 2

6. Create a city-wise average marks CTE.
7. Join the CTE with the `student` table.
8. Display students whose marks are above their city's average.
9. Create an age-wise average CTE.
10. Display students older than their age group's average.

---

## Level 3

11. Create two CTEs:

* Average marks
* Maximum marks

12. Display both values.

13. Create a city-wise student count CTE.

14. Join it with the `student` table.

15. Display each student along with the number of students in their city.

---

# Interview Questions

### Q1. What is a CTE?

A Common Table Expression is a temporary named result set that exists only during the execution of a single SQL statement.

---

### Q2. Why use a CTE?

* Improves readability
* Breaks complex queries into smaller steps
* Can be referenced multiple times in the same query
* Supports recursive queries

---

### Q3. Difference between a CTE and a View?

| CTE                        | View                                       |
| -------------------------- | ------------------------------------------ |
| Temporary                  | Permanent database object                  |
| Exists for one query       | Exists until dropped                       |
| Cannot be indexed directly | Can be indexed (depending on the database) |
| Great for readability      | Great for reusable logic                   |

---

# 🎯 What's Next?

You're now entering the **most important advanced SQL topic**:

## **Window Functions**

This includes:

* `ROW_NUMBER()`
* `RANK()`
* `DENSE_RANK()`
* `LEAD()`
* `LAG()`
* `FIRST_VALUE()`
* `LAST_VALUE()`
* Running totals
* Moving averages
* Partitioning (`PARTITION BY`)

Window functions are among the **top SQL interview topics** at companies like Amazon, Microsoft, Google, Meta, and many product-based companies because they solve complex analytical problems efficiently without collapsing rows like `GROUP BY` does.



Excellent. You're now learning one of the **most important SQL topics** for data analysis and interviews.

> **Window Functions** are asked in almost every SQL interview for mid-level roles.

---

# Phase 5.5: Window Functions

## What is a Window Function?

A **window function** performs calculations across a set of rows **without reducing the number of rows**.

This is the biggest difference from `GROUP BY`.

---

## GROUP BY

```sql
SELECT city, AVG(mark)
FROM student
GROUP BY city;
```

Output

| City   | AvgMark |
| ------ | ------- |
| Delhi  | 80      |
| Mumbai | 85      |
| Pune   | 88      |

Notice:

The rows are **collapsed** into one row per city.

---

## Window Function

```sql
SELECT
    name,
    city,
    mark,
    AVG(mark) OVER(PARTITION BY city) AS CityAverage
FROM student;
```

Output

| Name  | City   | Mark | CityAverage |
| ----- | ------ | ---- | ----------- |
| Rahul | Delhi  | 90   | 80          |
| Amit  | Delhi  | 70   | 80          |
| Priya | Mumbai | 95   | 85          |
| Rohit | Mumbai | 75   | 85          |
| Sneha | Pune   | 88   | 88          |

Notice:

No rows disappeared.

This is why they're called **window functions**.

---

# Syntax

```sql
Function()
OVER(
    PARTITION BY ...
    ORDER BY ...
)
```

`OVER()` tells SQL this is a window function.

---

# PARTITION BY

Think of it as:

> **GROUP BY without collapsing rows.**

Example

```sql
SELECT
name,
city,
mark,
AVG(mark) OVER(PARTITION BY city)
FROM student;
```

Each city becomes its own "window."

---

# Window Functions You Must Know

```
Window Functions

├── ROW_NUMBER() ⭐⭐⭐⭐⭐
├── RANK() ⭐⭐⭐⭐⭐
├── DENSE_RANK() ⭐⭐⭐⭐⭐
├── LEAD() ⭐⭐⭐⭐
├── LAG() ⭐⭐⭐⭐
├── FIRST_VALUE() ⭐⭐⭐
├── LAST_VALUE() ⭐⭐⭐
├── SUM() OVER() ⭐⭐⭐⭐
├── AVG() OVER() ⭐⭐⭐⭐
└── COUNT() OVER() ⭐⭐⭐
```

We'll learn them one by one.

---

# 1. ROW_NUMBER()

Assigns a unique number to each row.

Example

```sql
SELECT
name,
mark,
ROW_NUMBER() OVER(ORDER BY mark DESC) AS RankNo
FROM student;
```

Output

| Name  | Mark | RankNo |
| ----- | ---- | ------ |
| Priya | 95   | 1      |
| Rahul | 90   | 2      |
| Sneha | 88   | 3      |
| Rohit | 75   | 4      |
| Amit  | 70   | 5      |

Every row gets a unique number.

---

# PARTITION + ROW_NUMBER

```sql
SELECT
name,
city,
mark,
ROW_NUMBER() OVER(
PARTITION BY city
ORDER BY mark DESC
) AS CityRank
FROM student;
```

Output

| Name  | City   | Rank |
| ----- | ------ | ---- |
| Rahul | Delhi  | 1    |
| Amit  | Delhi  | 2    |
| Priya | Mumbai | 1    |
| Rohit | Mumbai | 2    |
| Sneha | Pune   | 1    |

Ranking restarts for each city.

---

# 2. RANK()

Suppose marks are

```
95
95
90
80
```

Query

```sql
SELECT
name,
mark,
RANK() OVER(ORDER BY mark DESC) AS Ranking
FROM student;
```

Output

| Mark | Rank |
| ---- | ---- |
| 95   | 1    |
| 95   | 1    |
| 90   | 3    |
| 80   | 4    |

Notice:

Rank **2** is skipped.

---

# 3. DENSE_RANK()

```sql
SELECT
name,
mark,
DENSE_RANK() OVER(ORDER BY mark DESC)
FROM student;
```

Output

| Mark | Rank |
| ---- | ---- |
| 95   | 1    |
| 95   | 1    |
| 90   | 2    |
| 80   | 3    |

No skipped numbers.

---

# ROW_NUMBER vs RANK vs DENSE_RANK

Suppose

```
95
95
90
80
```

| Marks | ROW_NUMBER | RANK | DENSE_RANK |
| ----- | ---------- | ---- | ---------- |
| 95    | 1          | 1    | 1          |
| 95    | 2          | 1    | 1          |
| 90    | 3          | 3    | 2          |
| 80    | 4          | 4    | 3          |

**Memorize this table.**

It's one of the most common SQL interview questions.

---

# 4. LEAD()

Returns the **next row's value**.

```sql
SELECT
name,
mark,
LEAD(mark) OVER(ORDER BY mark DESC) AS NextMark
FROM student;
```

Example

| Mark | NextMark |
| ---- | -------- |
| 95   | 90       |
| 90   | 88       |
| 88   | 75       |
| 75   | 70       |
| 70   | NULL     |

---

# 5. LAG()

Returns the **previous row's value**.

```sql
SELECT
name,
mark,
LAG(mark) OVER(ORDER BY mark DESC) AS PreviousMark
FROM student;
```

Output

| Mark | Previous |
| ---- | -------- |
| 95   | NULL     |
| 90   | 95       |
| 88   | 90       |
| 75   | 88       |

---

# Running Total

```sql
SELECT
name,
mark,
SUM(mark)
OVER(
ORDER BY id
) AS RunningTotal
FROM student;
```

Output

| Name  | Running Total |
| ----- | ------------- |
| Rahul | 90            |
| Amit  | 160           |
| Priya | 255           |
| Rohit | 330           |

Very common in reporting and dashboards.

---

# Running Average

```sql
SELECT
name,
AVG(mark)
OVER(
ORDER BY id
) AS RunningAverage
FROM student;
```

---

# Count Students per City

```sql
SELECT
name,
city,
COUNT(*)
OVER(PARTITION BY city) AS StudentsInCity
FROM student;
```

Unlike `GROUP BY`, every student row remains visible.

---

# Top N Students Per City (Interview Favorite)

Find the topper from each city.

```sql
SELECT *
FROM
(
SELECT
name,
city,
mark,
ROW_NUMBER()
OVER(
PARTITION BY city
ORDER BY mark DESC
) AS rn
FROM student
) t
WHERE rn=1;
```

This pattern is asked in many interviews.

---

# Practice Questions

## Level 1

1. Assign row numbers by marks.
2. Rank students by marks.
3. Dense rank students by marks.
4. Rank students within each city.
5. Count students in each city using a window function.

---

## Level 2

6. Find the topper of each city.
7. Find the top 2 students from each city.
8. Display the previous student's marks using `LAG()`.
9. Display the next student's marks using `LEAD()`.
10. Calculate a running total of marks.

---

## Level 3

11. Calculate a running average.
12. Show each student's mark and their city's average.
13. Show each student's mark and the overall average.
14. Rank students by age within each city.
15. Display the youngest student from every city.

---

# Interview Questions

### Q1. Difference between `GROUP BY` and Window Functions?

| GROUP BY             | Window Function                |
| -------------------- | ------------------------------ |
| Collapses rows       | Keeps all rows                 |
| One result per group | One result per row             |
| Used for summaries   | Used for analytics and ranking |

---

### Q2. Difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`?

* **ROW_NUMBER()**: Every row gets a unique number.
* **RANK()**: Ties share the same rank, and gaps appear.
* **DENSE_RANK()**: Ties share the same rank, with no gaps.

---

### Q3. When would you use `LAG()` or `LEAD()`?

To compare a row with the previous or next row, such as finding day-over-day sales changes or comparing consecutive records.

---

# 🚀 Next Phase

Once you're comfortable with window functions, we'll move to **Database Performance & Design**, covering:

1. ✅ Views
2. ✅ Indexes
3. ✅ Transactions (`COMMIT`, `ROLLBACK`, `SAVEPOINT`)
4. ✅ Normalization (1NF, 2NF, 3NF, BCNF)
5. ✅ SQL optimization and execution plans

These topics are essential for writing efficient, production-quality SQL.



Excellent! 🎉 At this point, you've covered around **85–90% of SQL** that most backend developers, data analysts, and interview candidates use.

The remaining topics are about **performance, reliability, and database design**.

---

# Phase 6: Views

## What is a View?

A **View** is a **virtual table** created from the result of a SQL query.

Unlike a normal table:

* ❌ It doesn't store data itself (in most databases).
* ✅ It stores only the SQL query.
* ✅ Every time you query the view, SQL executes the stored query.

Think of it like this:

```text
Table (Stores Data)
        ↓

View (Stores Query)

        ↓

Whenever you SELECT from View,
the query runs automatically.
```

---

# Why Use Views?

Imagine your manager asks for this report every day:

```sql
SELECT
    name,
    city,
    mark
FROM student
WHERE mark > 80
ORDER BY mark DESC;
```

Instead of writing it every day...

Create a view once.

---

# Creating a View

```sql
CREATE VIEW TopStudents AS
SELECT
    id,
    name,
    city,
    mark
FROM student
WHERE mark > 80;
```

Now simply write:

```sql
SELECT *
FROM TopStudents;
```

SQL runs the stored query behind the scenes.

---

# View Example

Suppose `student` contains:

| id  | name  | city   | mark |
| --- | ----- | ------ | ---- |
| 101 | Rahul | Delhi  | 90   |
| 102 | Amit  | Delhi  | 70   |
| 103 | Priya | Mumbai | 95   |
| 104 | Rohit | Mumbai | 75   |
| 105 | Sneha | Pune   | 88   |

Create the view:

```sql
CREATE VIEW TopStudents AS
SELECT
    id,
    name,
    city,
    mark
FROM student
WHERE mark >= 85;
```

Now:

```sql
SELECT *
FROM TopStudents;
```

Output:

| id  | name  | city   | mark |
| --- | ----- | ------ | ---- |
| 101 | Rahul | Delhi  | 90   |
| 103 | Priya | Mumbai | 95   |
| 105 | Sneha | Pune   | 88   |

---

# Updating the Base Table

Suppose you insert:

```sql
INSERT INTO student
VALUES
(106,'Anuj',22,92,'Pune');
```

Now run:

```sql
SELECT *
FROM TopStudents;
```

Output:

| Name  |
| ----- |
| Rahul |
| Priya |
| Sneha |
| Anuj  |

The view automatically reflects the latest data because it executes the stored query each time.

---

# View with JOIN

```sql
CREATE VIEW StudentDepartment AS
SELECT
    s.name,
    d.DepartmentName
FROM Students s
JOIN Departments d
ON s.DepartmentID = d.DepartmentID;
```

Now:

```sql
SELECT *
FROM StudentDepartment;
```

Instead of writing the JOIN every time.

---

# Updating a View

Some simple views can be updated directly:

```sql
UPDATE TopStudents
SET city = 'Hyderabad'
WHERE id = 101;
```

Whether this works depends on your database and the complexity of the view. Views involving joins, aggregates, or `GROUP BY` are often read-only.

---

# Renaming a View

Many databases don't support `RENAME VIEW` directly.

The common approach is:

```sql
DROP VIEW TopStudents;
```

Then recreate it:

```sql
CREATE VIEW BestStudents AS
SELECT ...
```

---

# Deleting a View

```sql
DROP VIEW TopStudents;
```

Only the view is removed.

The original table is untouched.

---

# View vs Table

| Table               | View                                |
| ------------------- | ----------------------------------- |
| Stores data         | Stores a query                      |
| Occupies storage    | Minimal storage (stores definition) |
| Can insert directly | Depends on the view                 |
| Permanent data      | Virtual representation              |

---

# Advantages

* Reuse complex queries
* Improve readability
* Restrict access to sensitive columns
* Simplify reporting
* Centralize business logic

---

# Practice Questions

## Level 1

1. Create a view showing students with marks above 80.
2. Query the view.
3. Create a view showing only `name` and `city`.
4. Create a view showing students from Delhi.
5. Drop one of the views.

---

## Level 2

6. Create a view using an `INNER JOIN` between `Students` and `Departments`.
7. Create a view showing city-wise average marks.
8. Create a view showing student count per city.
9. Create a view using `GROUP BY`.
10. Query all created views.

---

## Level 3

11. Create a view showing the topper from each city (using a window function or subquery).
12. Create a view of students above the overall average marks.
13. Create a view combining student and department details.
14. Try updating a simple view and observe the result.
15. Drop all the views you created.

---

# Interview Questions

### Q1. What is a View?

A view is a virtual table based on the result of a SQL query. It stores the query, not the data (in most databases).

---

### Q2. Why use Views?

* Simplify complex queries
* Improve security
* Reuse query logic
* Present filtered or aggregated data

---

### Q3. Difference between View and Table?

| Table                    | View                                     |
| ------------------------ | ---------------------------------------- |
| Stores actual data       | Stores only a query                      |
| Can exist independently  | Depends on underlying tables             |
| Uses disk space for data | Usually stores only the query definition |

---

# 📍 Next Topic: Indexes (Very Important)

Indexes are one of the most frequently discussed performance topics in SQL interviews.

You'll learn:

* What an index is
* Clustered vs Non-clustered indexes
* Primary key indexes
* Composite indexes
* When indexes speed up queries
* When indexes can actually slow things down
* How to choose the right columns to index

Understanding indexes will help you write queries that scale from thousands to millions of rows.

Excellent. 🔥 You're now entering the **Database Performance** section, which is heavily tested in backend interviews.

---

# Phase 7: Indexes (One of the Most Important SQL Topics)

> **An Index is a data structure that makes searching faster.**

Think of it like the index at the back of a book.

Without an index:

📖 You read every page until you find the topic.

With an index:

📖 You jump directly to the correct page.

Databases work the same way.

---

# Without an Index

Suppose your `student` table has:

| id        | name  | city   | mark |
| --------- | ----- | ------ | ---- |
| 1         | Rahul | Delhi  | 90   |
| 2         | Priya | Mumbai | 85   |
| 3         | Amit  | Delhi  | 70   |
| ...       | ...   | ...    | ...  |
| 1,000,000 | Rohit | Pune   | 95   |

Now run:

```sql
SELECT *
FROM student
WHERE name = 'Rahul';
```

Without an index, SQL checks:

```text
Row 1

↓

Row 2

↓

Row 3

↓

...

↓

Until Rahul is found
```

This is called a **Full Table Scan**.

Time Complexity:

```text
O(n)
```

---

# With an Index

Create an index:

```sql
CREATE INDEX idx_name
ON student(name);
```

Now the database can jump directly to "Rahul."

```text
Index

↓

Rahul

↓

Pointer

↓

Actual Row
```

Time Complexity is much closer to:

```text
O(log n)
```

---

# Syntax

## Create Index

```sql
CREATE INDEX index_name
ON table_name(column_name);
```

Example:

```sql
CREATE INDEX idx_city
ON student(city);
```

---

## View Indexes

Different databases have different commands.

For MySQL:

```sql
SHOW INDEXES
FROM student;
```

---

## Drop Index

MySQL:

```sql
DROP INDEX idx_city
ON student;
```

SQL Server:

```sql
DROP INDEX idx_city ON student;
```

PostgreSQL:

```sql
DROP INDEX idx_city;
```

---

# How an Index Works

Imagine this table:

| id | name  |
| -- | ----- |
| 5  | Rahul |
| 3  | Amit  |
| 9  | Priya |
| 7  | Sneha |

Index stores something like:

```text
Amit → Row 3

Priya → Row 9

Rahul → Row 5

Sneha → Row 7
```

Notice:

The index is **sorted**, even if the table isn't.

---

# Primary Key Index

```sql
CREATE TABLE student(
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
```

When you create a `PRIMARY KEY`, the database **automatically creates an index**.

So:

```sql
SELECT *
FROM student
WHERE id = 101;
```

is already fast.

You don't need another index on `id`.

---

# Unique Index

```sql
CREATE UNIQUE INDEX idx_email
ON users(email);
```

Benefits:

* Fast searching
* Prevents duplicate values

---

# Composite Index

Suppose you frequently search like this:

```sql
SELECT *
FROM student
WHERE city='Delhi'
AND age=22;
```

Create:

```sql
CREATE INDEX idx_city_age
ON student(city, age);
```

This is called a **Composite Index**.

---

# When Should You Create an Index?

Good candidates:

* Primary Keys
* Foreign Keys
* Frequently searched columns (`WHERE`)
* Columns used in `JOIN`
* Columns used in `ORDER BY`
* Columns used in `GROUP BY`

---

# When Should You Avoid an Index?

Avoid indexing:

* Very small tables
* Columns with very few unique values (e.g., `gender` with only `Male`/`Female`)
* Columns that are updated constantly
* Every single column (too many indexes hurt performance)

---

# Why Too Many Indexes Are Bad

Indexes speed up reads but slow down writes.

When you insert:

```sql
INSERT INTO student
VALUES (...);
```

The database must:

1. Insert the row.
2. Update every affected index.

More indexes = slower inserts, updates, and deletes.

---

# Clustered vs Non-Clustered Index

## Clustered Index

The table data itself is stored in the order of the index.

```text
Index
↓

Actual Data
```

Only **one clustered index** per table is possible.

Typically created automatically for the primary key in SQL Server.

---

## Non-Clustered Index

The index is separate and points to the data.

```text
Index

↓

Pointer

↓

Actual Data
```

You can have many non-clustered indexes.

---

# Example

```sql
SELECT *
FROM student
WHERE city='Delhi';
```

If `city` has an index:

```text
Index

↓

Delhi

↓

Matching Rows
```

Without an index:

```text
Scan every row
```

---

# EXPLAIN (Query Plan)

Most databases let you see how a query will execute.

MySQL:

```sql
EXPLAIN
SELECT *
FROM student
WHERE city='Delhi';
```

Look for:

* **Using index** ✅
* **Full table scan** ❌

Learning to read execution plans is a valuable performance skill.

---

# Practice Questions

## Level 1

1. Create an index on `name`.
2. Create an index on `city`.
3. Drop the city index.
4. Display all indexes.
5. Create a unique index on `email` (if your table has one).

---

## Level 2

6. Create a composite index on `(city, age)`.
7. Run a query using both columns.
8. Create an index on `mark`.
9. Compare query execution with and without the index (using `EXPLAIN`).
10. Observe how inserts behave after adding multiple indexes.

---

## Level 3

11. Decide which columns in your `student` table should be indexed and explain why.
12. Identify columns that should **not** be indexed.
13. Write queries that benefit from indexes.
14. Write queries where an index is unlikely to help.
15. Use `EXPLAIN` to analyze at least three different queries.

---

# Interview Questions

### Q1. What is an index?

A data structure that speeds up data retrieval by allowing the database to locate rows efficiently without scanning the entire table.

---

### Q2. Why not index every column?

Because indexes consume storage and slow down `INSERT`, `UPDATE`, and `DELETE` operations.

---

### Q3. Which operations benefit from indexes?

* `WHERE`
* `JOIN`
* `ORDER BY`
* `GROUP BY`

---

### Q4. Which operations become slower because of indexes?

* `INSERT`
* `UPDATE`
* `DELETE`

---

### Q5. Difference between Clustered and Non-Clustered Index?

| Clustered                  | Non-Clustered              |
| -------------------------- | -------------------------- |
| Data stored in index order | Index stored separately    |
| One per table              | Multiple allowed           |
| Faster range scans         | Good for selective lookups |

---

# 🎯 Next Topic: Transactions (Very Important)

You'll learn:

* `BEGIN`
* `COMMIT`
* `ROLLBACK`
* `SAVEPOINT`
* ACID properties
* Real-world banking examples

Transactions are one of the **most important concepts** for backend development because they ensure data remains consistent even if something goes wrong during a series of database operations.


