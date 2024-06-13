`SQL` is a querying language that we use to query databases. `MySQL` is a relational database management system.

---

To create a database, we open the command line and: we first need

```Bash
echo 'export PATH=/usr/local/mysql/bin:$PATH'>>~/.bash_profile
. ~/.bash_profile
```

To login into `mySQL`, we need to:

```Plain
mysql -u root -p
```

To create a database, once logged into `mySQL`:

```Plain
create databse allan

# where allan is the name of the databse I want to create
```

Moving forward, I will use popsql instead of using the command line.

Once in popsql, and once I am in the database I created, I can now access it and add/modify/delete relational tables.

---

To create a table, I do:

```Plain
CREATE TABLE student (
 student_id INT PRIMARY KEY,
 name VARCHAR(20),
 major VARCHAR(30)
);
```

We know that in these tables, we have `PRIMARY KEY` but also can have `FOREIGN KEY`, so, we can add those to a table. But first, if the `FOREIGN KEY` references another table, that other table must first exist before the `FOREIGN KEY` can be made.

```Plain
CREATE TABLE branch (
  branch_id INT PRIMARY KEY,
  branch_name VARCHAR(40),
  mgr_id INT,
  mgr_start_date DATE,
  FOREIGN KEY(mgr_id) REFERENCES employee(emp_id) ON DELETE SET NULL
);
```

---

To `query` a database, we use `SELECT` to do so. Say we want to select all computer science majors from a table of students:

```Plain
SELECT student.major
FROM student
WHERE student.major = 'computer science';
```

Notice that we only have `;` on the end of the query, so from `SELECT` to `WHERE`, that is the entire query and so, only a `;` at the very end.

We can also nest these queries. Say we wanted a list of employees who earn over 30000 and we want the list with their names sorted in order by first, then last:

```Plain
SELECT employee.first_name, employee.last_name
FROM eomployee
WHERE employee.emp_id IN (
  SELECT works_with.temp_id
  FROM works_with
  WHERE work_with.total_sales > 30000;
);
```

The keyword here is `IN` which we use like this when we nest queries. We sort the list once, pass the result to a new sort, which then returns the desired list.

---

To update records in a table, say I want to change all instances of the major being biology, to biochemistry:

```Plain
UPDATE student
SET major = 'biochemistry'
WHERE major = "biology';
```

The `WHERE` above can also query whether something is `NULL` so no entry, or `NOT NULL`.

---

We can delete existing records in a table. The general syntax is:

```Plain
DELETE FROM table_name WHERE condition;
```

For example:

```Plain
DELETE FROM Customers WHERE CustomerName='Alfreds Futterkiste';
```

To delete all rows in a table, without actually deleting the table itself, we call:

```Plain
DELETE FROM table_name;
```

Another thing I can do to filter the table is to select the top `x%` of the table:

```Plain
SELECT TOP 50 PERCENT * FROM CUSTOMERS
-- this selects the first 50% of the records from the CUSTOMERS table
```

The `MIN()` and `MAX()`function return the largest and smallest value of the selected column:

```Plain
-- I want to find the cheapest product in a table of a company's products

SELECT MIN(Price) AS SmallestPrice
FROM PRODUCTS
```

In the above code snippet, `Price` is the name of column in the table, and we are saving that value as `SmallestPrice`. Again, we are selecting all of this from the `PRODUCTS` table.

---

What if I want to know the number of elements in a column that match a certain criteria? We can use `COUNT()` to do so.

```Plain
SELECT COUNT(column_name)
FROM table_name
WHERE condition;
```

Some other useful functions for numeric columns are:

```Plain
SELECT AVG(column_name)
FROM table_name
WHERE condition;

-- the above finds the average of a coumn

SELECT SUM(column_name)
FROM table_name
WHERE condition;

-- the above computes the sum of a column
```

---

Another very important `SQL` operator is the `LIKE ... WHERE` operator.

The `LIKE` operator is used in a `WHERE` clause to search for a specified pattern in a column.

- There are 2 wildcards often used in conjunction with the `LIKE`operator:
    - The percent sign `%`represents zero, one, or multiple characters.
    - The underscore sign `_` represents one, single, character.