# CPE 365 Study Guide

# Motivation

Speed of access to disk is very slow. Easy to get lots of data with a database.

# Basics

## BasicRDM

**Normalization** is the process of factoring data into tables in such a way as to avoid redundancies or repetitions. Allows us to avoid innacurate edits as well as save space. We only need to edit data in one place to have it effect information throughout the system. 

## Keys And Joins

### Key Citeria

1. Every row has a value for the key.
2. No two rows have the same key.
3. Key is minimal. No superfluous data.

### Primary Key

Primary keys are more efficient than normal keys and have the fastest lookup time. Foreign keys target primary keys for this reason.

### Cartesian Product

Product of two tables combines every combination of rows from each table.

**Query optimizer** figures out the best way to carry out a join without generating excess data.
**Secondary sorting key** is the second value in an order by statement that breaks the tie when the first sorting key match.  
**ER diagrams** or Entity-Relationship Diagram, shows relationships in a database. 

n-m relationships need an additional table.

Use a **table alias** to reference a table multiple times in a statement.

# DB Design

## Creating DB's - DDL

DDL is a subset of commands from SQL.

Why are there so many column type choices?

1. Types allow for flexible data size.
2. Allows for the prevention of human error.
3. Disk storage is made easier with predictable sized data.

### Create Command

Make new databases':
```sql
create database name;
```

**collation order** is the order in which characters are sorted. Can be set when creating a new database.

Make new tables:
```sql
create table <name> 
```
followed by a comma separated list of column declarations wrapped in parentheses. 

To create a table with identical columns as that of another table use the *like* syntax:
```sql
create table newTable like oldTable;
```

Each table needs to have a default value or be initalized to null.

Any comparisons with null are automatically false. Need to check using special comparisons:
```sql
is null or is not null
```

## Data Types

### Integer Types

| Type                  | Bytes | Range              |
| --------------------- | ----- | ------------------ |
| tinyint               | 1     | -128, 127          |
| smallint              | 2     | +/- 32768          |
| mediumint             | 3     | +/- 16 million     |
| int, integer          | 4     | +/- 2 billion      |
| bigint                | 8     | +/- 9 quintillion  |

All of the types above are signed but can also be declared as unsigned. Parenthesized numbers after the integer are formatting cues for displaying text. 

### Decimal Types

Use decimal for exact values where you want no rounding.

| Type                        | Bytes           |
| --------------------------- | --------------- |
| decimal(digits, dec points) | 2 * # of digits |


### Floating Point Types

| Type                  | Bytes | Precision          |
| --------------------- | ----- | ------------------ |
| float                 | 4     | 7 decimal points   |
| double                | 8     | 16 decimal points  |

The parenthasized value after the type adjusts the value to be either float or double. For example:
```sql
float(10)
```
will choose whichever data type supports 10 digits of accuracy.

### String Types


| Type                  | Bytes      | Notes              |
| --------------------- | ---------- | ------------------ |
| char(n)               | n          | stores exactly n chars. empty spots are blank padded |
| varchar(n)            | n+length | only uses as much space as needed for each string. stores the string length  |

varchar can cause disk fragmentation. A varchar(255) will only use one byte to store the length but is bad becuase it doesn't catch human errors.

### Large Data Types

Databases can also store large binary data like images in something called a blob (Binary Large OBject). Very similar to a string in that both are sequences of bytes.

| Type                  | Bytes     | Notes              |
| --------------------- | --------- | ------------------ |
| binary(n)             | n         | similar to char    |
| varbinary(n)          | n+length  | similar to varchar |

For cases where size checking isn't needed text or blob makes a good fit. text is is meant to store string data with character encoding and blob is for raw binary data. Compared to binary forms, text and blob do not allow for scrubbing of the data. 

| Type                  | Bytes | Size               |
| --------------------- | ----- | ------------------ |
| tinytext              | 1     | 255 bytes          |
| text                  | 2     | 65 KiB             |
| mediumtext            | 3     | 16 MiB             |
| longtext              | 4     | 4 GiB              |
| tinyblob              | 1     | 255 bytes          |
| blob                  | 2     | 65 KiB             |
| mediumblob            | 3     | 16 MiB             |
| longblob              | 4     | 4 GiB              |


### Date and Time Types

| Type                  | Format                |
| --------------------- | --------------------- |
| datetime              | YYYY-MM-DD HH:MM:SS   |
| date                  | YYYY-MM-DD            |
| mediumtext            | HH:MM:SS              |
| timestamp             | just like datetime    |

time can represent values up to 840 hours. timestamp and datetime both record time in microsecond accuracy since epoch, Jan 1, 1970. timestamp will show the time whatever the current timezone is translating from UTC. All datetime values are in UTC time.

### Enum Types

Enumeration variables take on values from a fixed list of values. Design goals for enumerations require:

1. Way to ensure that values into an enum column are from the allowed set of values.
2. Order for the values of an enum.
3. Way to translate enum values to readable strings, preferably internationally.


#### Enums as Integers

The simplest approach is to store each enum as an integer. Doesn't meet criteria 1 or 3. This can be ok as many interactions do not involve any humans therefore a human readable format isn't dire. Chaning the ordering requires for all of the instances of the enum to be modified accordingly.

#### Enums as Lookup Tables

Create a separate table of allowed enum values. This method meets the criteria except that it doesn't allow for internationalization. This problem can be fixed by adding an additional language column. Downside to this method is that it requires an additional database join. 

#### Built-In Enum Types

A built-in enum type lists requires enum values listed directly in the declaration as shown:

```sql
builtinColor enum('red', 'green', 'blue')
```

You address these columns as if there were strings (e.g `where color = 'blue'`) but the values are stored as integers with values 1 , 2 and 3. The 0 value is reserved for an error indication. If you try and assign an invalid value into the enum column you get the error value assigned. This method does not support internationalization. Error handling is also weak as it is silent.


## Primary Key Contraints

Create table statements can also include **constraint** declerations. Declaring a column as a primary key will setup an index on that column and also the column will be non-nullable.

### Indexes 

An index is usually a tree that allows for the quick access of data in a table. A B-tree is a fast version of this which only applies to the primary key's index. One can add an index to any column by doing the following:

```sql
INDEX index_name (column_name, column_name, ...)
```

An index does not require that a column be unique. Indexes are spece-expensive so use them wisely. A **clustered index** allows for the rows in memory to be stored sequentially making it easy to access a range of values. In MySQL the primary key is a clustered index.

## Unique Key Contraints

To ensure that a column has all unique values one can put a unique constraint on the column.

```sql
UNIQUE KEY `UKkind_flavor` (`kind`, `flavor`)
```

Unique key constraint does allow for null values.

## Foreign Key Constraints

Foreign key constraints connect parent tables to child tables. The foreign key lives in the child table and references the primary key in the parent table. This creates a 1-n relationship where the n is the child and the 1 is the parent.

```sql
CONSTRAINT `FKReceipt_customerId` FOREIGN KEY (`customerId`) REFERENCES `Customer` (`id`)
```

Adding this constraint ensures that the foreign key to correspond to an actual row in the target table. This constraint can add an index on the key.


## Good Database Design

**DRY**: Don't Repeat Yourself. Very applicable in database design.

Have no **obligatory nulls**, temporary nulls to mean something. 


## Modifying DB's


### Updating Rows

The `update` command changes volumn values in existing rows:

```sql
update Person set unitsCompleted = 18 where id = 3;
```

without a where clause the entire table is modified. Can set multiple columns by separating the assignments with commas.

### Inserting Rows

To add a new row to a table, use the `insert` statement:

```sql
insert into Person values (11, 'Staley', 'Clint', 'M', NULL, NULL, 'Computer Science');
```

Inserts must comply with constraints.

To conduct a column specific insert use the following syntax:
```sql
insert into Person(lastName, firstName, gender, dept) values ('Smith', 'Jane', 'F', 'English');
```

All other columns are left unaffected. 

### Deleting Rows

Use the SQL `delete` command to remove rows from a table:

```sql
delete from Person where id = 13;
```

Moving a row with a foreign key constraint has different effects depending on tthe foreign key constraint.


### Altering Columns

To change column definitions use the `alter` command:


```sql
alter table Person modify column lastName varchar(30);
```

When modifying tables with existing data the effect depends on the change. Shortening a data type will throw an error if it leads to truncation. Changing between general data types will result in type conversions. You typically cannot modify the types in a column referenced by a foreign key as the foreign key type would also need to be changed. 


### Adding Columns

Add a column by using the `add` keyword:

```sql
alter table Person add favoriteColor varchar(15)
```


### Removing Columns

Remove a column by using the `drop` keyword:

```sql
alter table Person drop favoriteColor;
```

If a column is used in a foreign key then it cannnot be dropped until all references are gone.


### Changing Constraints

To remove a key use this syntax:

```sql
alter table Teaches drop key FKTeaches_sectionId
```

To drop an index instead swap out the key for index. Every foreign key has a **supporting key constraint** that is listed separatley from the constraint itself. The supporting key constraint should be dropped with the FK.

To add a new constraint use:

```sql
alter table Teaches add constraint FKTeaches_sectionId foreign key(sectionId) references Section(id) on delete cascade on update cascade;:w
```

### Live Database Upgrading

**Temporary tables** are tables that can be used for migration but will be removed once a database session has closed.

```sql
create temporary table DeptNames (
    name varchar(30)
);
```

# DB Analysis

## Simple Functions

There are many different types of basic functions that do everything from text formatting to mathematical operations. Type conversion are automatically done for many functions. Best to just reference the SQL manual for this one.

## Aggregation

Aggregate functions act on many rows of a table. For example the following statement gets teh max price from the product table:

```sql
select max(price) from Product;
```

The **distinct** modifier allows for only the distinct values in a column to be considered for aggregation.

```sql
select count(distinct price) from Product;
```

## Group By

The group by clause collapses sets of rows into a single row based on a particular **grouping column**. The `group by` groups all the rows with the same value in the grouping column into a single aggregate row. The aggregate function is applied to the values that make up a single aggregate row. For example, to find the average price of each flavored product we would do:

```sql
select flavor, avg(price) from Product group by flavor
```

We can also group by multiple columns which creates an aggregate row for each combination of values.

## Selecting non-grouping Columns

Selecting by a non-grouping column can lead to unpredictable results as there isn't an obvious choice for what should be returned. Oracle gives an error for selecting non-grouping columns. MySQL allows this. 5.7 may actually fix this. 

## Having Clause

The `having` clause allows for selection on the data post-aggregation.

```sql
select flavor, avg(price) from Product group by flavor having count(*) > 1 order by avg(price);
```

# Adv Queries

## Subqueries

## Outer Joins


# JDBC

The Java DataBase Connectivity (JDBC) library is a collection of classes and methods to interface with a database. Using the JDBC requires a **JDBC driver**, A set off classes that supports the JDBC implementation. There are four different dypes of JDBC driver, numbered 1-4. The lower the number the more general the driver where the highest number has the most functionality.

With JDBC there are many different types of objects used to maintain connections to a database.

| Class                       | Description                          |
| --------------------------- | ------------------------------------ |
| Connection                  | Initial connection to the database                      |
| Statement                   | Can perform queries or updates.                      |
| PreparedStatement           | Is an almost-complete sql command. Typically much faster.  |
| ResultSet                   | Result of a query.                      |

A **connection pool** is maintained by JDBC and creates many Connection objects to easily execute many SQL statements.
An iterator is maintained on a ResultSet, also called a **cursor**, to keep track of the query results. 


# Transactions

Related statements can be grouped into a **transaction** to be applied in one collective action. Transaction also have the added security of being able to be rolled back and erase all changes. 

```sql
start transaction;
...
make changes
...
commit;  or  rollback;
```

Transactions do not protect you from DDL commands, these will take effect immediatly.

## ACID

The acronym ACID are four principal characteristics expected of transactions. ACID stands for Atomic, Consistent, Isolated and Durable.

### Atmoic 

All statements in a transaction will be performed. No transaction is "half performed".

### Consistent

Database actions are performed correctly.

### Isolated

Modifying the same data from two different database sessions.

### Durable

Once committed the effects of a transaction become permanent.


You can also set **auto-commit mode** such that every command is automatically entered into a commit.


## Isolation Levels

Protection against two different sessions accessing and modifying a database is protected using isolation levels. There are four different isolation levels.


### Read Uncommitted 
### Read Committed
### Repeatable Read
### Serializable









# DB Theory

## Basics

## Relational Algebra







