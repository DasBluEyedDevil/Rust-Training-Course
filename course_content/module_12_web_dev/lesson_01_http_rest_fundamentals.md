# Module 12: Web Development with Rust

# Lesson 1: HTTP & REST Fundamentals

## Welcome to Full-Stack Development!

You've mastered Rust fundamentals and built CLI applications. Now let's build **web applications** that serve users over HTTP!

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is how computers communicate on the web.

### Simple Analogy

Think of HTTP like **mailing letters**:

- **Request**: You send a letter (request) asking for information
- **Response**: The recipient sends back a letter (response) with the information
- **Method**: The type of request (GET = "please send", POST = "please save this")
- **Headers**: Envelope information (sender, content type, etc.)
- **Body**: The actual message content

### HTTP Request Structure

```
GET /api/users/123 HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer token123

[optional body data]
```

**Parts:**
- **Method**: GET (what action to perform)
- **Path**: `/api/users/123` (what resource)
- **Headers**: Metadata about the request
- **Body**: Data being sent (for POST, PUT, PATCH)

### HTTP Response Structure

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 45

{"id": 123, "name": "Alice", "email": "alice@example.com"}
```

**Parts:**
- **Status Code**: 200 (result of the request)
- **Headers**: Metadata about the response
- **Body**: The actual data

## HTTP Methods

### **GET** - Retrieve Data

```
GET /api/products
```

**Purpose**: Fetch data without changing anything

**Analogy**: Looking at items in a store window

**Example uses**:
- Get list of users
- Fetch a single blog post
- Retrieve search results

### **POST** - Create New Data

```
POST /api/products
Content-Type: application/json

{"name": "Laptop", "price": 999}
```

**Purpose**: Create a new resource

**Analogy**: Submitting a form to create a new account

**Example uses**:
- Create a new user
- Submit a comment
- Upload a file

### **PUT** - Update/Replace Data

```
PUT /api/products/123
Content-Type: application/json

{"name": "Gaming Laptop", "price": 1299}
```

**Purpose**: Replace entire resource

**Analogy**: Replacing an old file with a completely new version

### **PATCH** - Partial Update

```
PATCH /api/products/123
Content-Type: application/json

{"price": 1199}
```

**Purpose**: Update only specific fields

**Analogy**: Editing just one paragraph in a document

### **DELETE** - Remove Data

```
DELETE /api/products/123
```

**Purpose**: Delete a resource

**Analogy**: Throwing away an item

## HTTP Status Codes

Status codes tell you what happened:

### **2xx - Success**

- **200 OK**: Request succeeded
- **201 Created**: New resource created
- **204 No Content**: Success, but no data to return

### **3xx - Redirection**

- **301 Moved Permanently**: Resource moved to new URL
- **302 Found**: Temporary redirect

### **4xx - Client Errors**

- **400 Bad Request**: Invalid request data
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Authenticated but not allowed
- **404 Not Found**: Resource doesn't exist
- **422 Unprocessable Entity**: Valid format, but can't process

### **5xx - Server Errors**

- **500 Internal Server Error**: Server crashed
- **503 Service Unavailable**: Server temporarily down

### Quick Reference

```rust
// In Rust with axum, you'll use:
StatusCode::OK                    // 200
StatusCode::CREATED               // 201
StatusCode::BAD_REQUEST           // 400
StatusCode::NOT_FOUND             // 404
StatusCode::INTERNAL_SERVER_ERROR // 500
```

## What is REST?

**REST (Representational State Transfer)** is a set of conventions for building web APIs.

### REST Principles

**1. Resource-Based URLs**

Resources are nouns, not verbs:

✅ **Good:**
```
GET    /api/users           # Get all users
GET    /api/users/123       # Get user 123
POST   /api/users           # Create user
PUT    /api/users/123       # Update user 123
DELETE /api/users/123       # Delete user 123
```

❌ **Bad:**
```
GET  /api/getUsers
POST /api/createUser
POST /api/deleteUser/123
```

**2. Use HTTP Methods Correctly**

- GET for reading
- POST for creating
- PUT/PATCH for updating
- DELETE for deleting

**3. Stateless**

Each request contains everything needed (no relying on server memory of previous requests).

**4. Standard Status Codes**

Use HTTP status codes to indicate results.

**5. JSON Format**

Modern APIs use JSON for data:

```json
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com",
  "created_at": "2025-01-15T10:30:00Z"
}
```

## RESTful API Example

**E-commerce API:**

```
# Products
GET    /api/products           # List all products
GET    /api/products/42        # Get product 42
POST   /api/products           # Create new product
PUT    /api/products/42        # Update product 42
DELETE /api/products/42        # Delete product 42

# Orders
GET    /api/orders             # List all orders
POST   /api/orders             # Create new order
GET    /api/orders/99          # Get order 99

# Nested resources
GET    /api/orders/99/items    # Get items in order 99
POST   /api/orders/99/items    # Add item to order 99
```

## URL Structure

```
https://api.example.com/v1/products?category=electronics&sort=price

Protocol: https://
Domain: api.example.com
Version: /v1
Resource: /products
Query params: ?category=electronics&sort=price
```

**Query Parameters** (after `?`):
- Filtering: `?status=active`
- Sorting: `?sort=name&order=asc`
- Pagination: `?page=2&limit=20`
- Search: `?q=laptop`

## Headers

### Common Request Headers

```
Content-Type: application/json        # Format of body
Accept: application/json              # Format you want back
Authorization: Bearer token123        # Authentication
User-Agent: MyApp/1.0                 # Client info
```

### Common Response Headers

```
Content-Type: application/json        # Format of body
Content-Length: 1234                  # Size in bytes
Cache-Control: max-age=3600           # Caching rules
Location: /api/users/123              # New resource URL (for 201)
```

## JSON - The Language of APIs

**JSON (JavaScript Object Notation)** is the standard data format for web APIs.

### JSON Syntax

```json
{
  "string": "text value",
  "number": 42,
  "float": 3.14,
  "boolean": true,
  "null_value": null,
  "array": [1, 2, 3],
  "object": {
    "nested": "value"
  }
}
```

### Rust Structs ↔ JSON

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
    active: bool,
}

// Becomes:
{
  "id": 1,
  "name": "Alice",
  "email": "alice@example.com",
  "active": true
}
```

## Complete Request/Response Flow

### Example: Creating a User

**1. Client sends request:**

```http
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Accept: application/json

{
  "name": "Bob",
  "email": "bob@example.com"
}
```

**2. Server processes:**

- Validates the data
- Creates user in database
- Generates ID and timestamp

**3. Server sends response:**

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/456

{
  "id": 456,
  "name": "Bob",
  "email": "bob@example.com",
  "created_at": "2025-11-13T14:30:00Z"
}
```

## API Design Best Practices

### ✅ DO:

- Use plural nouns for collections: `/users`, `/products`
- Use standard HTTP methods
- Return appropriate status codes
- Include timestamps
- Version your API: `/v1/users`
- Provide clear error messages

### ❌ DON'T:

- Use verbs in URLs: `/getUser`
- Return 200 for errors
- Nest too deeply: `/api/users/1/orders/2/items/3/reviews`
- Change response formats without versioning

## Error Response Format

**Standard error response:**

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": {
      "field": "email",
      "reason": "missing"
    }
  }
}
```

## Practice Exercise

### Exercise 1: Design a Blog API

Design RESTful endpoints for a blog with:
- Posts (title, content, author, published_at)
- Comments (post_id, author, content)
- Tags

**Your turn:** Write the URL endpoints and HTTP methods.

<details>
<summary>Solution</summary>

```
# Posts
GET    /api/posts              # List posts
GET    /api/posts/123          # Get post 123
POST   /api/posts              # Create post
PUT    /api/posts/123          # Update post 123
DELETE /api/posts/123          # Delete post 123

# Comments
GET    /api/posts/123/comments # List comments for post 123
POST   /api/posts/123/comments # Add comment to post 123
DELETE /api/comments/456       # Delete comment 456

# Tags
GET    /api/tags               # List all tags
GET    /api/posts?tag=rust     # Filter posts by tag
```
</details>

### Exercise 2: HTTP Status Codes

What status code should be returned for each scenario?

1. User successfully created ➜ ?
2. Resource not found ➜ ?
3. Invalid email format ➜ ?
4. Authentication token missing ➜ ?
5. Server database crashed ➜ ?

<details>
<summary>Solution</summary>

1. **201 Created** - New resource created successfully
2. **404 Not Found** - Resource doesn't exist
3. **400 Bad Request** - Client sent invalid data
4. **401 Unauthorized** - Authentication required
5. **500 Internal Server Error** - Server problem
</details>

### Exercise 3: Request/Response

Design the request and response for:

**Create a book in a library API**

Book has: title, author, isbn, published_year

<details>
<summary>Solution</summary>

**Request:**
```http
POST /api/books HTTP/1.1
Content-Type: application/json

{
  "title": "The Rust Book",
  "author": "Steve Klabnik",
  "isbn": "978-1718503106",
  "published_year": 2023
}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/books/789

{
  "id": 789,
  "title": "The Rust Book",
  "author": "Steve Klabnik",
  "isbn": "978-1718503106",
  "published_year": 2023,
  "created_at": "2025-11-13T15:00:00Z"
}
```
</details>

## Tools for Testing APIs

### cURL (Command line)

```bash
# GET request
curl https://api.example.com/users

# POST request
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@example.com"}'

# With auth
curl https://api.example.com/users \
  -H "Authorization: Bearer token123"
```

### httpie (Friendlier than curl)

```bash
# GET
http GET https://api.example.com/users

# POST
http POST https://api.example.com/users name=Alice email=alice@example.com
```

### Postman / Insomnia

GUI tools for testing APIs (download from their websites)

## What's Next?

Now that you understand HTTP and REST fundamentals, you're ready to build your first Rust web server with **axum**!

In the next lesson, we'll:
- Set up an axum project
- Create a simple web server
- Handle HTTP requests
- Return JSON responses

## Key Takeaways

- ✅ HTTP is request/response protocol
- ✅ GET retrieves, POST creates, PUT/PATCH updates, DELETE removes
- ✅ Status codes indicate results (2xx success, 4xx client error, 5xx server error)
- ✅ REST uses resource-based URLs and standard HTTP methods
- ✅ JSON is the standard data format for APIs
- ✅ Good API design makes endpoints predictable and easy to use

**Next**: Building your first web server with axum!

---

**Progress**: Module 12, Lesson 1 complete (58/90+ lessons total)
