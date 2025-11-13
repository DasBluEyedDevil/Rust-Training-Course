# Module 12 Quiz: Web Development Fundamentals

## Questions

### 1. Which HTTP method is used to retrieve data?
a) POST
b) PUT
c) GET ✓
d) DELETE

### 2. What status code indicates successful resource creation?
a) 200 OK
b) 201 Created ✓
c) 204 No Content
d) 400 Bad Request

### 3. What does REST stand for?
a) Reliable State Transfer
b) Representational State Transfer ✓
c) Resource Execution and State Transfer
d) Remote State Transaction

### 4. In Axum 0.8, what is the new path parameter syntax?
a) `/users/:id`
b) `/users/{id}` ✓
c) `/users/$id`
d) `/users/<id>`

### 5. What does `Json<T>` in Axum do?
a) Only serializes responses
b) Only deserializes requests
c) Both serializes and deserializes ✓
d) None of the above

### 6. Which extractor gets URL query parameters?
a) Path<T>
b) Query<T> ✓
c) Form<T>
d) State<T>

### 7. What is `Arc<Mutex<T>>` used for in web applications?
a) Encryption
b) Shared state across threads ✓
c) Async operations
d) JSON parsing

### 8. Which status code indicates unauthorized access?
a) 403 Forbidden
b) 401 Unauthorized ✓
c) 404 Not Found
d) 500 Internal Server Error

### 9. What does `with_state()` do in Axum?
a) Creates new state
b) Shares state with handlers ✓
c) Deletes state
d) Validates state

### 10. Which is the correct way to handle errors in Axum?
a) Return string errors
b) Use panic!()
c) Implement IntoResponse for custom error types ✓
d) Ignore errors

## Answers

1. c) GET
2. b) 201 Created
3. b) Representational State Transfer
4. b) `/users/{id}`
5. c) Both serializes and deserializes
6. b) Query<T>
7. b) Shared state across threads
8. b) 401 Unauthorized
9. b) Shares state with handlers
10. c) Implement IntoResponse for custom error types

**Passing Score**: 8/10 (80%)
