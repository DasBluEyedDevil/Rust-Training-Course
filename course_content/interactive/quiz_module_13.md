# Module 13 Quiz: Database Integration

## Questions

### 1. Which crate provides async, type-safe database access?
a) diesel
b) sqlx ✓
c) postgres
d) rusqlite

### 2. What does `query!` macro do?
a) Runs query at runtime
b) Validates SQL at compile time ✓
c) Generates random queries
d) Formats SQL strings

### 3. What is a connection pool?
a) Database table
b) Reusable database connections ✓
c) SQL query
d) Migration tool

### 4. Which method expects exactly one row?
a) fetch_all
b) fetch_optional
c) fetch_one ✓
d) execute

### 5. What SQL command creates a table?
a) INSERT TABLE
b) CREATE TABLE ✓
c) NEW TABLE
d) MAKE TABLE

### 6. What is a foreign key?
a) Primary encryption key
b) Reference to another table ✓
c) Unique identifier
d) Index type

### 7. What is a JOIN used for?
a) Combining tables ✓
b) Deleting data
c) Creating indexes
d) Encrypting data

### 8. What does SERIAL in PostgreSQL do?
a) Stores strings
b) Auto-increments integers ✓
c) Creates timestamps
d) Defines boolean

### 9. Which is correct for parameterized queries?
a) `"SELECT * FROM users WHERE id = {}"
b) `"SELECT * FROM users WHERE id = $1"` ✓
c) `"SELECT * FROM users WHERE id = %s"`
d) `"SELECT * FROM users WHERE id = ?"`

### 10. What tool manages database migrations in SQLx?
a) cargo migrate
b) sqlx migrate ✓
c) diesel migrate
d) psql migrate

## Answers

1. b) sqlx
2. b) Validates SQL at compile time
3. b) Reusable database connections
4. c) fetch_one
5. b) CREATE TABLE
6. b) Reference to another table
7. a) Combining tables
8. b) Auto-increments integers
9. b) `"SELECT * FROM users WHERE id = $1"`
10. b) sqlx migrate

**Passing Score**: 8/10 (80%)
