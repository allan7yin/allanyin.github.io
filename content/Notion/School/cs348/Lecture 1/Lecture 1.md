### May 7th, 2024

This class was an introduction to database management. Most of the class revolved around high-level concepts. For example _What is a database? What is the data we store in it?_ Well yeah, its tables and relations, but does this data have any meaning if we don't label it and build this relational construct around it? Class started from very very high level questions about the nature of data, and why even build database engines in the first place.

## Module 1: Overview of Data Management

### ==**Case Study: Company Payroll System**==

We'll start this unit with a case study. Namely, we will use a ACME (Some made up company) Payroll System to introduce several concepts. Say this company wanted to carefully **manage information** about its personnel, their salaries, etc. We need to design a way of modeling this data for them.

Infrastructure for this includes:

- Server computer with mass storage
- **PAT**: Payroll application team
- **DBA**: Database Administration

So:

- The company will use an existing RDBMS to manage all the payroll information.
- **PAT** is responsible for the payroll application
- **DBA** is responsible for server computer and supporting all other information systems, for example: **inventory** and **online-order**  
    These kinds of systems are deployed using  
    **three-tier architecture**, which consists of _browser_ -> _client_ -> _server_ -> _RDBMS_, where sever is some application written using **C** and **SQL**

So, what is a DBMS? A DBMS manages 2 kinds of information: **metadata** and **data**. How are these different? Consider the following examples:  
  
_**Logical Metadata**_:

- (This is something designed by the **PAT**, the tables schema if you will)
- Some kind of entity called an `employee`
- This entity has attributes `employee-number`, `name`, and `salary`
- Each employee entity has values for the above attributes
- Employees are identified by their `employee-number`

_**Data**_:

- (Data is enabled by the metadata, without it, we just have some large quantity of numbers and strings, etc. Only with the metadata, is this data useful)
- Mary is an employee
- Mary's employee number is 3142
- Mary's name is "Smith, Mary"
- Mary's salary is $92,000 USD

So, we have some logical construct for data, but how is it stored on a computer? To do so, we will have **physical metadata***:

- (physical metadata refers to the information about the structure and organization of the database stored within the system)
- There is file of records called `emp-file`
- There are record fields `emp-num`, `emp-name`, and `emp-salary`
- Each `emp-file` record has the above fields
- File `emp-file` is organized as a **B-Tree** data structure that supports an `emp-lookup` operation, given a value for field `emp-num`
- Basically, this refers to things like table structure, indexes using for searching, data storage, etc.
- Records in the `emp-file` correspond one-to-one with `employee` entities
- Record fields in `emp-file` encode the corresponding attribute values for `employee` entities, for example, `emp-num` encodes an `employee-number`

Here is a diagram to help visualize this:

  
  
**PAT** is responsible for the `PAYROLL` external schema design. There are others as well. Its important to recognize that in the above, when we refer to `online-order`, `inventory`, `payroll`, these are not individual tables. Rather, they are _"Logical entities”_ in our system.

![[Screenshot_2024-05-08_at_3.49.48_PM.png]]

- There are `online-order` entities, but this is probably represented through a slew of tables. The same is true for `inventory` and `payroll`

**DBA** is responsible for all the physical design. That is, how are those logical schemas going to be stored on disk? How are we going to efficiently query for this data? What if I want to search for a payroll record by a `record_id`?

### Information Access

How do we access data from a database? Before we look into SQL query's, let's try forming one in plain english (which happens, by no coincidence, to resemble toe corresponding SQL query):

- Find the `salary` for any `employee` whose `employee-number` is given by a parameter `p`
- This statement is of course interacting with the **logical metadata**
- We can make a couple of observations about the query:
    - _independent of physical metadata_ -> that is, we don't need to know ANYTHING about this will be retrieved from disk
    - **independent of either** `**inventory**` **or** `**online-order**` **metadata**
    - _declarative_ -> i.e no indication on how to compute the answers

The RDBMS takes a query (like the above) and invokes an operation to fulfill that query:

- Invokes operation `emp-lookup(p)`, just once, on the tile `emp-file`. If an `emp-file` record is found, then extract and return the value of field `emp-salary`.

A couple of observations we can make:

- ==**reliability**== - the data is important to the company
- ==**concurrency**== - there may be multiple concurrent reads or writes to `payroll` users. How can we address this?
- ==**data integrity**== - there are quite a few _integrity constraints_ imposed on `payroll` by the metadata:
    - Employee numbers are unique
    - Employee names need to be valid characters/strings
    - Employee salaries are at least $60,000 (USD)
- ==**security**== - there are lots of restrictions of who should be able to access and update this data (obviously not everyone should be read and write access)
- ==**data is structured**== - The queries and updates request by `payroll`on the data managed by RDBMS is concise and clear

This was a high-level attempt to show the layers of abstractions in a RDBMS, and the idea of what data is and what structured data is.

### Overview of DBMS

In this subsection, we'll provide some concrete textbook definitions or many things we saw in the above case study.

==**Database:**== ==A large and== ==_persistent_== ==collection of metadata and data organized in a way that facilitates efficient== ==_retrieval_== ==and== ==_revision_==

==**Database Management System (DBMS):**== A set of programs that implements a **data model** to manage a database

==**Data Model:**== this determines the nature of the metadata and how retrieval and revision is expressed. In a relational database, the data model typically consists of **tables**, **columns**, and **relationships** between tables

  

The diagram above shows some important properties:

- **==Physical Data Independence:==** ==A property of application code devoid of any mention pf the physical schema .== This means the people managing the tables and schema don’t need to make changes to how data is stored, whenever they are making schema changes.
- **==Logical Data Independence:==** _==pairs==_ ==of external schema in which there is no common logical metadata.== This means if `payroll` and `inventory` schemas have no relationships, then they can be modified and worked on without knowing about the other.

  

==**⇒ General Properties of DBMS**==

==First, the DBSMS== ==**provides utilities for db monitoring and maintenance**==. Also, the DBMS adopts some data model for managing structured data via an inference with 2 sub-languages: **DML + DDL**. Let’s see what each of these is and does.

**→ Data Definition Language (DDL)**

Part of the DBMS enabling clients to manage metadata. This is used to define the structure and schema of the database. It includes things like creating, modifying, and deleting database objects such as tables, indexes, views, and constraints → **to manage metadata.** This metadata includes:

- a logical schema: logical metadata about how data is understood
- a physical schema: physical metadata defining how data is encoded
- external schemata: subsets of the logical schema

```SQL
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name VARCHAR(50),
    Age INT
);
```

**→ Data Manipulation Language (DML)**

This is the part of the DBMS enabling clients to manage data. This needs to be **physically data independent,** and can also be **declarative (devoid of algorithmic details on how the data is manipulated)**. DML is used to manage and manipulate the data stored in the database. It includes commands for querying, inserting, updating, and deleting data within database tables.

```SQL
INSERT INTO Students (StudentID, Name, Age) VALUES (1, 'John Doe', 20);
UPDATE Students SET Age = 21 WHERE StudentID = 1;
DELETE FROM Students WHERE StudentID = 1;
```

**==Notice: In the above, SQL was used as an example for both. Thats bceause IT IS!S QL (Structured Query Language) encompasses both DDL (Data Definition Language) and DML (Data Manipulation Language) along with other types of commands.==**

  

A lot of these observations are made to make those working with DBMS more productive. That is why we have physical and logical independence, non-procedural syntax, and supporting **concurrent** data manipulation. While these are great, there are some things we need to consider, namely in the area of concurrency. Reliability and concurrency issues are addressed by supporting **transactions**, in particular **ACID** properties for transactions.

  

**→ Transactions**

Race conditions and overwrites can occur when multiple applications access the same data. Consider the following example:

```Go
// user at ATM 1
withdraw(AC, 1000)
Bal := getbal(AC)

if (Bal > 1000) {
	<give-money>
	setbal(AC, Bal - 1000)
}
```

```Go
// user at ATM 2
withdraw(AC, 500)
// here, the ATM 1 is gettig the balance
Bal := getbal(AC)

if (Bal > 500) {
	<give-money>
	setbal(AC, Bal - 500)
}
```

Say the client’s account balance started with `$1200`, and then the we first check balance at UTM 1. We do the same right after at UTM 2. So, for both operations, the system thinks the balance has sufficient funds, when in reality, after executing one of them, the client will not have enough to withdraw. So, the above will result in the client withdrawing `$1500` when they don’t have that much! What we want is for these operations to be atomic, we need one of the fully finish before the other executes. Let’s formally define this.

  

**→ Transaction:** A sequence of _indivisible_ DML requests. Applications access a database via transactions. Programmers can assume exclusive access to database, as DBMS schedules DML requests from **all** transactions while maintaining data integrity. Namely. transactions are maintain **ACID**:

- **Atomicity:** Transaction occurs entirely or not at all, completely finishes its execution before another transaction starts
- **Consistency:** Each transaction preserves consistency of the db
- **Isolation:** Concurrent do not interfere with each other
- **Durability:** once completed, transaction’s changes are permanent

  

**→ Relational Data Model**

So, we know what a data model is, its this set of rules on the nature of the metadata and how data is retrieved. One type (and the one this course focuses on) is the **Relational Data Model (RM).** The RM accomplishes all of the above we’ve talked about (data independence, declarative data manipulation, and use of transactions). The standard interface for RM is **SQL (Standard Query Language)**. Let’s see an very basic example of this:

![[Screenshot_2024-05-08_at_5.11.10_PM.png]]

3. This query first retrieves a list of distinct names (only the name column is being specified) from the `Author` table. Then, it filters it by those that are not in the `Wrote` table.  
  
DB knows that author is column in  
`Wrote` and that aid is column in `Author`

```SQL
// In the above query, what if the columns had the same name? Then, we should alias our columns in the query:
select distinct A.name 
from AUTHOR A
where not exists (
    select * 
    from WROTE W
    where W.author = A.aid
);
```

![[Screenshot_2024-05-08_at_5.29.40_PM.png]]

![[Screenshot_2024-05-08_at_5.29.52_PM.png]]

The answers to the above questions:

**LeftSide:**

1. The entities are the author and publication (is wrote an entity)
2. The relationships is the `(1,N)` and `(0, N)` → one to many and other is (optional) to many
3. Above
4. Choose an authors A such that there does not exist an entry in wrote by that author

  

**RightSide:**