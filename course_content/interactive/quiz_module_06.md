# 📝 Module 6: Error Handling - Assessment Quiz

**Time Limit**: 20-25 minutes
**Passing Score**: 80% (24/30 points)
**Difficulty**: ⭐⭐⭐⭐ Critical Concepts

---

## Instructions

This quiz tests your understanding of Rust's error handling system including Option, Result, custom errors, and the ? operator. Answer all questions honestly without referring to the lessons. Answers and explanations are at the bottom.

---

## Section 1: Option<T> Understanding (10 points)

### Question 1 (2 points)
**When should you use Option<T> vs Result<T, E>?**

a) Option when value might not exist, Result when operation can fail with error details
b) Option for numbers, Result for strings
c) Always use Result, Option is deprecated
d) They're interchangeable

**Your Answer**: _____

---

### Question 2 (2 points)
**What does this code print?**

```rust
let x: Option<i32> = None;
let y = x.unwrap_or(10);
println!("{}", y);
```

a) Compile error
b) Panic
c) 10
d) None

**Your Answer**: _____

---

### Question 3 (3 points)
**What does map() do on an Option?**

```rust
let x = Some(5);
let y = x.map(|n| n * 2);
```

a) Creates Some(10)
b) Creates Some(5)
c) Panics
d) Creates None

**Your Answer**: _____

---

### Question 4 (3 points)
**Will this code compile?**

```rust
fn get_first(v: Vec<i32>) -> i32 {
    v.first()?
}
```

a) Yes, compiles fine
b) No, ? operator returns Option but function returns i32
c) Yes, but runtime error
d) No, ? can't be used with vectors

**Your Answer**: _____

---

## Section 2: Result<T, E> Understanding (10 points)

### Question 5 (2 points)
**What are the two variants of Result?**

a) Some and None
b) Ok and Err
c) Success and Failure
d) True and False

**Your Answer**: _____

---

### Question 6 (3 points)
**What does this code do?**

```rust
let result: Result<i32, String> = Ok(42);
let value = result.unwrap_or_else(|e| {
    println!("Error: {}", e);
    0
});
```

a) Panics with error message
b) Returns 42
c) Returns 0 and prints error
d) Compile error

**Your Answer**: _____

---

### Question 7 (2 points)
**What's the main danger of using .unwrap()?**

a) It's slow
b) It panics if Result is Err or Option is None
c) It causes memory leaks
d) It's deprecated

**Your Answer**: _____

---

### Question 8 (3 points)
**What does and_then() do?**

```rust
let result = divide(10, 2)
    .and_then(|n| divide(n, 0));
```

a) Chains operations, stops at first error
b) Adds the results together
c) Runs both operations regardless of errors
d) Compile error

**Your Answer**: _____

---

## Section 3: The ? Operator (10 points)

### Question 9 (3 points)
**What does the ? operator do?**

a) Makes values optional
b) If Ok/Some, unwraps; if Err/None, returns early
c) Always panics
d) Converts Result to Option

**Your Answer**: _____

---

### Question 10 (2 points)
**Can you use ? in a function that returns ()?**

a) Yes, always
b) No, function must return Result or Option
c) Only with Option
d) Only with Result

**Your Answer**: _____

---

### Question 11 (3 points)
**Will this compile?**

```rust
use std::fs;

fn read_file() -> Result<String, std::io::Error> {
    let content = fs::read_to_string("file.txt")?;
    Ok(content.to_uppercase())
}
```

a) Yes, compiles fine
b) No, ? can't be used with fs operations
c) No, return type mismatch
d) Yes, but runtime error

**Your Answer**: _____

---

### Question 12 (2 points)
**What is Box<dyn Error> used for?**

a) Allocating errors on heap
b) Allowing multiple different error types
c) Making errors faster
d) Required for all custom errors

**Your Answer**: _____

---

## Section 4: Custom Error Types (Bonus: 5 points)

### Bonus Question 1 (3 points)
**What trait should custom errors implement to work with Box<dyn Error>?**

a) Display
b) Error
c) Debug
d) Both Error and Display

**Your Answer**: _____

---

### Bonus Question 2 (2 points)
**What does the From trait enable?**

```rust
impl From<io::Error> for MyError {
    fn from(error: io::Error) -> Self {
        MyError::IoError(error)
    }
}
```

a) Automatic error conversion with ?
b) Better error messages
c) Faster code execution
d) Nothing, it's optional

**Your Answer**: _____

---

## Scoring

**Calculate your score:**
- Section 1: ____ / 10 points
- Section 2: ____ / 10 points
- Section 3: ____ / 10 points
- Bonus: ____ / 5 points
- **Total: ____ / 30 points (____ %)**

**Performance Evaluation:**
- 27-30 (90-100%): **Excellent!** Error handling mastered
- 24-26 (80-89%): **Good!** Minor review needed
- 21-23 (70-79%): **Fair**. Review key concepts
- Below 21 (< 70%): **Review module thoroughly**

---

## Answer Key & Explanations

<details>
<summary>Click to reveal answers (try the quiz first!)</summary>

### Section 1: Option<T>

**Q1: a** - Use `Option<T>` when a value might not exist (no error, just absence). Use `Result<T, E>` when an operation can fail and you need to know why.

**Q2: c** - Prints 10. `unwrap_or()` provides a default value when Option is None. Since x is None, it returns 10.

**Q3: a** - Creates `Some(10)`. `map()` applies the function to the value inside Some. 5 * 2 = 10.

**Q4: b** - Won't compile. `v.first()` returns `Option<&i32>`, and `?` would make the function return `Option`, but it's declared to return `i32`. Type mismatch.

### Section 2: Result<T, E>

**Q5: b** - The two variants are `Ok(value)` for success and `Err(error)` for failure.

**Q6: b** - Returns 42. Since result is `Ok(42)`, `unwrap_or_else()` returns the Ok value without calling the closure.

**Q7: b** - `.unwrap()` panics if the Result is Err or Option is None, crashing your program. Use it only when you're certain there's a value.

**Q8: a** - `and_then()` chains operations that return Results. If any operation returns Err, the chain stops and returns that error. First operation succeeds (Ok(5)), second fails (Err("Division by zero")).

### Section 3: The ? Operator

**Q9: b** - If `Ok(value)` or `Some(value)`: unwraps and continues. If `Err(e)` or `None`: returns the error/None immediately from the function.

**Q10: b** - No. The `?` operator can only be used in functions that return `Result<T, E>` or `Option<T>` because it needs to return early with an error.

**Q11: a** - Yes, compiles fine! `fs::read_to_string()` returns `Result<String, io::Error>`, which matches the function's return type. The `?` operator unwraps on success or returns the error.

**Q12: b** - `Box<dyn Error>` is a trait object that can hold any type implementing the Error trait, allowing you to return different error types from the same function.

### Bonus Section

**Bonus 1: d** - Custom errors should implement both `Error` trait (to work with error handling system) and `Display` trait (for user-friendly messages). Debug is also commonly derived.

**Bonus 2: a** - The `From` trait enables automatic error conversion. When you use `?` operator, if the error types don't match, Rust automatically calls `From::from()` to convert between error types.

</details>

---

## Recommendations Based on Score

### 90-100% - Excellent!
You've mastered error handling! You understand the difference between Option and Result, can use the ? operator effectively, and know when to create custom errors.

**Next Steps:**
- Proceed confidently to Module 7
- You're ready for professional Rust development
- Practice building error-resilient applications

### 80-89% - Good!
You understand core concepts with minor gaps.

**Review:**
- Lesson 3: The ? Operator
- Lesson 4: Custom Error Types
- Retake quiz in 1-2 days

### 70-79% - Fair
You grasp basics but need more practice.

**Review:**
- Lessons 1-2: Option and Result deep dives
- Lesson 3: ? operator patterns
- Complete practice project again
- Retake quiz after review

### Below 70% - Needs Review
Error handling is crucial for robust Rust applications!

**Action Plan:**
1. Re-read Module 6 completely
2. Type every code example yourself
3. Complete the File Validator project
4. Try the challenges
5. Study the error handling patterns
6. Retake quiz after thorough review

**Remember**: Good error handling is what separates toy projects from production-ready applications. This skill is essential!

---

## Additional Resources

- [The Rust Book - Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
- [Rust by Example - Error Handling](https://doc.rust-lang.org/rust-by-example/error.html)
- [anyhow crate](https://docs.rs/anyhow/) - Application-level error handling
- [thiserror crate](https://docs.rs/thiserror/) - Library-level error handling
- [Error Handling Survey](https://blog.burntsushi.net/rust-error-handling/)

---

[← Back to Quiz System](QUIZ_SYSTEM.md) | [Module 6 Lessons →](../module_06_error_handling/)
