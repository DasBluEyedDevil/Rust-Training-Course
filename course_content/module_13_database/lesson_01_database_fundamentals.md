# Module 13: Database Integration

# Lesson 1: Database & SQL Fundamentals

## Why Databases?

In Module 12, we stored data in memory using `Arc<Mutex<Vec<T>>>`. This works for learning, but has problems:

❌ **Problems with in-memory storage:**
- Data disappears when server restarts
- Can't scale beyond one server
- No querying capabilities
- Limited by available RAM

✅ **Benefits of databases:**
- Persistent storage (data survives restarts)
- Powerful querying and indexing
- Concurrent access from multiple servers
- Transactions and data integrity
- Can handle massive datasets

## Types of Databases

### Relational Databases (SQL)

Store data in **tables** with **relationships** between them.

**Popular SQL databases:**
- **PostgreSQL** ⭐ (What we'll use - powerful, open source)
- MySQL / MariaDB
- SQLite (embedded, no server needed)

**When to use:**
- Structured data with clear relationships
- Need transactions and data integrity
- Complex queries and joins
- Traditional business applications

### NoSQL Databases

Store data in flexible formats (documents, key-value, etc.)

**Popular NoSQL databases:**
- MongoDB (documents)
- Redis (key-value, caching)
- Cassandra (wide-column)

**When to use:**
- Flexible or changing schema
- Massive scale horizontal scaling
- High-speed caching
- Real-time analytics

## We'll Use PostgreSQL

**Why PostgreSQL?**
- Most powerful open-source SQL database
- Excellent Rust support (sqlx)
- ACID compliant (reliable transactions)
- JSON support (best of both worlds)
- Free and widely used

## SQL Basics

**SQL (Structured Query Language)** is how you talk to relational databases.

### Creating Tables

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Breaking it down:**
- `CREATE TABLE users` - Make a new table called "users"
- `id SERIAL PRIMARY KEY` - Auto-incrementing unique identifier
- `VARCHAR(255)` - String with max length 255
- `NOT NULL` - Field is required
- `UNIQUE` - No duplicates allowed
- `DEFAULT NOW()` - Automatically set current timestamp

### Data Types

| SQL Type | Rust Equivalent | Description |
|----------|-----------------|-------------|
| `SERIAL` | `i32` | Auto-incrementing integer |
| `INTEGER` | `i32` | Whole number |
| `BIGINT` | `i64` | Large whole number |
| `VARCHAR(n)` | `String` | Variable-length string |
| `TEXT` | `String` | Unlimited-length string |
| `BOOLEAN` | `bool` | True/false |
| `TIMESTAMP` | `DateTime<Utc>` | Date and time |
| `JSON` | `serde_json::Value` | JSON data |

### Inserting Data

```sql
INSERT INTO users (name, email, age)
VALUES ('Alice', 'alice@example.com', 30);

-- Insert multiple rows
INSERT INTO users (name, email, age)
VALUES
    ('Bob', 'bob@example.com', 25),
    ('Charlie', 'charlie@example.com', 35);
```

### Querying Data

```sql
-- Get all users
SELECT * FROM users;

-- Get specific columns
SELECT name, email FROM users;

-- Filter with WHERE
SELECT * FROM users WHERE age > 25;

-- Order results
SELECT * FROM users ORDER BY created_at DESC;

-- Limit results
SELECT * FROM users LIMIT 10;

-- Combine conditions
SELECT * FROM users
WHERE age > 20 AND name LIKE 'A%'
ORDER BY name
LIMIT 5;
```

### Updating Data

```sql
UPDATE users
SET age = 31, email = 'alice.new@example.com'
WHERE id = 1;

-- Update multiple rows
UPDATE users
SET age = age + 1
WHERE age < 30;
```

### Deleting Data

```sql
DELETE FROM users WHERE id = 1;

-- Delete all rows (careful!)
DELETE FROM users WHERE age > 100;
```

## Relationships

Tables can relate to each other.

### One-to-Many

One user can have many posts:

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

**`REFERENCES users(id)`** creates a **foreign key** - links to users table.

### Querying Related Data (JOIN)

```sql
-- Get posts with user information
SELECT posts.*, users.name as author_name
FROM posts
JOIN users ON posts.user_id = users.id
WHERE users.name = 'Alice';
```

### Many-to-Many

Posts can have many tags, tags can be on many posts:

```sql
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id),
    tag_id INTEGER REFERENCES tags(id),
    PRIMARY KEY (post_id, tag_id)
);
```

## Primary Keys & Indexes

### Primary Key

**Uniquely identifies each row**, usually an auto-incrementing `id`.

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,  -- Auto-incrementing primary key
    name VARCHAR(255) NOT NULL
);
```

### Indexes

Make queries faster by creating an index:

```sql
-- Speed up lookups by email
CREATE INDEX idx_users_email ON users(email);

-- Speed up filtering by age
CREATE INDEX idx_users_age ON users(age);
```

**When to use indexes:**
- Columns used in WHERE clauses
- Columns used in JOIN conditions
- Columns used for sorting (ORDER BY)

**Trade-off:** Indexes speed up reads but slow down writes.

## Constraints

Enforce data rules at the database level:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,  -- Cannot be NULL
    price DECIMAL(10, 2) CHECK (price > 0),  -- Must be positive
    sku VARCHAR(50) UNIQUE,  -- No duplicates
    category_id INTEGER REFERENCES categories(id)  -- Must exist in categories
);
```

### Types of Constraints

- `NOT NULL` - Value required
- `UNIQUE` - No duplicates
- `CHECK` - Custom validation
- `FOREIGN KEY` - Must reference existing row
- `DEFAULT` - Set default value

## Transactions

**Transactions** ensure multiple operations succeed or fail together:

```sql
BEGIN;

INSERT INTO accounts (name, balance) VALUES ('Alice', 1000);
INSERT INTO accounts (name, balance) VALUES ('Bob', 500);

UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
UPDATE accounts SET balance = balance + 100 WHERE name = 'Bob';

COMMIT;  -- or ROLLBACK to undo
```

**ACID Properties:**
- **Atomic**: All or nothing
- **Consistent**: Database stays valid
- **Isolated**: Transactions don't interfere
- **Durable**: Changes are permanent

## Practice Exercises

### Exercise 1: Design a Blog Schema

Create tables for a blog with:
- Users (id, username, email, password_hash, created_at)
- Posts (id, user_id, title, content, published, created_at, updated_at)
- Comments (id, post_id, user_id, content, created_at)

<details>
<summary>Solution</summary>

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for common queries
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);
CREATE INDEX idx_comments_post_id ON comments(post_id);
```
</details>

### Exercise 2: Write Queries

Given the blog schema above, write SQL for:

1. Get all published posts with author name
2. Get all comments for post ID 5 with commenter names
3. Count posts per user
4. Get the 10 most recent published posts

<details>
<summary>Solutions</summary>

```sql
-- 1. Published posts with author name
SELECT posts.*, users.username as author
FROM posts
JOIN users ON posts.user_id = users.id
WHERE posts.published = TRUE
ORDER BY posts.created_at DESC;

-- 2. Comments for post 5 with commenter names
SELECT comments.*, users.username as commenter
FROM comments
JOIN users ON comments.user_id = users.id
WHERE comments.post_id = 5
ORDER BY comments.created_at ASC;

-- 3. Count posts per user
SELECT users.username, COUNT(posts.id) as post_count
FROM users
LEFT JOIN posts ON users.id = posts.user_id
GROUP BY users.id, users.username
ORDER BY post_count DESC;

-- 4. 10 most recent published posts
SELECT * FROM posts
WHERE published = TRUE
ORDER BY created_at DESC
LIMIT 10;
```
</details>

### Exercise 3: E-commerce Schema

Design tables for an e-commerce site with:
- Products (id, name, description, price, stock)
- Orders (id, user_id, status, total, created_at)
- Order Items (order_id, product_id, quantity, price_at_purchase)

Include appropriate constraints and foreign keys.

<details>
<summary>Solution</summary>

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price_at_purchase DECIMAL(10, 2) NOT NULL,
    UNIQUE(order_id, product_id)
);

-- Indexes
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```
</details>

## Installing PostgreSQL

### macOS

```bash
brew install postgresql@15
brew services start postgresql@15
```

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
```

### Windows

Download installer from [postgresql.org](https://www.postgresql.org/download/windows/)

### Docker (All platforms)

```bash
docker run --name postgres \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  -d postgres:15
```

### Verify Installation

```bash
psql --version
# Should show: psql (PostgreSQL) 15.x
```

## Connecting to PostgreSQL

### Using psql (command-line client)

```bash
# Connect to default database
psql -U postgres

# Create a database
CREATE DATABASE blog_dev;

# Connect to your database
\c blog_dev

# List tables
\dt

# Describe table
\d users

# Quit
\q
```

### GUI Tools

- **pgAdmin** - Official GUI tool
- **DBeaver** - Universal database tool
- **TablePlus** - Modern, clean interface (Mac/Windows)
- **Postico** - Mac only

## Database URL Format

Rust applications connect using a **connection string**:

```
postgresql://username:password@host:port/database
```

**Example:**
```
postgresql://postgres:mypassword@localhost:5432/blog_dev
```

**Parts:**
- `postgresql://` - Database type
- `postgres` - Username
- `mypassword` - Password
- `localhost` - Server address
- `5432` - Port (PostgreSQL default)
- `blog_dev` - Database name

## Key Takeaways

- ✅ Databases provide persistent, queryable storage
- ✅ PostgreSQL is a powerful, open-source SQL database
- ✅ Tables store structured data with defined schemas
- ✅ SQL is used to query and manipulate data
- ✅ Foreign keys create relationships between tables
- ✅ Indexes speed up queries
- ✅ Transactions ensure data consistency
- ✅ Constraints enforce data integrity

**Next**: Setting up SQLx and connecting to PostgreSQL from Rust!

---

**Progress**: Module 13, Lesson 1 complete (63/90+ lessons total)
