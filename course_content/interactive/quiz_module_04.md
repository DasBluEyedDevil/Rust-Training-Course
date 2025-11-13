# 📝 Module 4: Ownership System - Assessment Quiz

**Time Limit**: 20-25 minutes
**Passing Score**: 80% (24/30 points)
**Difficulty**: ⭐⭐⭐⭐⭐ Critical Module

---

## Instructions

This quiz tests your understanding of Rust's ownership system - THE most important concept in Rust. Answer all questions honestly without referring to the lessons. Answers and explanations are at the bottom.

---

## Section 1: Conceptual Understanding (10 points)

### Question 1 (2 points)
**What are the three ownership rules in Rust?**

a) Each value has multiple owners, owners can overlap, values persist after owner disappears
b) Each value has an owner, only one owner at a time, value dropped when owner goes out of scope
c) Values are reference counted, multiple refs allowed, garbage collected
d) Ownership is optional, can be disabled, managed by programmer

**Your Answer**: _____

---

### Question 2 (2 points)
**Which is stored on the heap?**

a) `let x = 5;`
b) `let active = true;`
c) `let s = String::from("hello");`
d) `let ch = 'A';`

**Your Answer**: _____

---

### Question 3 (2 points)
**What happens when you assign one variable to another with heap data?**

```rust
let s1 = String::from("hello");
let s2 = s1;
```

a) Both s1 and s2 are valid and point to separate copies
b) s1 is moved to s2; s1 is no longer valid
c) s2 is a reference to s1
d) Compile error

**Your Answer**: _____

---

### Question 4 (2 points)
**What does the borrow checker prevent?**

a) Memory leaks only
b) Data races, use-after-free, double-free
c) Slow code execution
d) Type mismatches

**Your Answer**: _____

---

### Question 5 (2 points)
**When does a value get dropped (freed)?**

a) Manually with `free()`
b) When the garbage collector runs
c) When its owner goes out of scope
d) After 5 seconds of inactivity

**Your Answer**: _____

---

## Section 2: Code Analysis (10 points)

### Question 6 (2 points)
**Will this code compile?**

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);
    println!("{}", s);
}

fn takes_ownership(text: String) {
    println!("{}", text);
}
```

a) Yes, compiles fine
b) No, error: s moved into function
c) Yes, but runtime error
d) No, error: function signature wrong

**Your Answer**: _____

---

### Question 7 (2 points)
**Will this code compile?**

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();
    println!("{}, {}", s1, s2);
}
```

a) No, s1 was moved
b) No, clone not available
c) Yes, clone makes deep copy
d) No, type mismatch

**Your Answer**: _____

---

### Question 8 (3 points)
**What does this print?**

```rust
fn main() {
    let x = 5;
    let y = x;
    println!("{}, {}", x, y);
}
```

a) Compile error: x moved
b) 5, 5
c) 0, 5
d) Undefined behavior

**Your Answer**: _____

---

### Question 9 (3 points)
**Fix this code (what needs to be added?)**

```rust
fn main() {
    let s = String::from("hello");
    let len = calculate_length(s);
    println!("{}", s); // Want to use s here
}

fn calculate_length(text: String) -> usize {
    text.len()
}
```

a) Add `.clone()` when calling function
b) Change function to take `&String`
c) Make s mutable
d) Return the String along with length

**Your Answer**: _____

---

## Section 3: Borrowing & References (10 points)

### Question 10 (2 points)
**What's the difference between `&T` and `&mut T`?**

a) No difference, just style
b) `&T` is immutable reference, `&mut T` is mutable reference
c) `&T` is for stack, `&mut T` is for heap
d) `&mut T` is faster

**Your Answer**: _____

---

### Question 11 (3 points)
**How many mutable references can exist to the same data at once?**

a) Unlimited
b) Exactly one
c) Up to 10
d) Two per thread

**Your Answer**: _____

---

### Question 12 (2 points)
**Will this code compile?**

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s;
    let r2 = &s;
    let r3 = &mut s;
    println!("{}, {}, {}", r1, r2, r3);
}
```

a) Yes, all references are valid
b) No, can't have mutable reference while immutable exist
c) Yes, but runtime panic
d) No, s must be immutable

**Your Answer**: _____

---

### Question 13 (3 points)
**What makes this code safe?**

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{}, {}", r1, r2);

    let r3 = &mut s;
    r3.push_str(" world");
}
```

a) r1 and r2 are no longer used after println!, so r3 is OK
b) Compiler is lenient here
c) s is mutable
d) This code won't compile

**Your Answer**: _____

---

## Bonus Section: Advanced (5 bonus points)

### Bonus Question 1 (3 points)
**What are lifetimes for?**

a) Measuring program execution time
b) Ensuring references don't outlive the data they point to
c) Memory optimization
d) Thread management

**Your Answer**: _____

---

### Bonus Question 2 (2 points)
**Which requires explicit lifetime annotations?**

```rust
fn example1(x: &str) -> &str { x }

fn example2(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}
```

a) Both need annotations
b) Neither needs annotations
c) Only example1
d) Only example2

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
- 27-30 (90-100%): **Excellent!** Ownership mastered
- 24-26 (80-89%): **Good!** Minor review needed
- 21-23 (70-79%): **Fair**. Review Lessons 4-5
- Below 21 (< 70%): **Review module thoroughly**

---

## Answer Key & Explanations

<details>
<summary>Click to reveal answers (try the quiz first!)</summary>

### Section 1: Conceptual Understanding

**Q1: b** - The three rules are: each value has an owner, only one owner at a time, value dropped when owner goes out of scope.

**Q2: c** - `String` is heap-allocated and growable. The others (i32, bool, char) are stack-allocated.

**Q3: b** - For heap data, assignment **moves** ownership. s1 becomes invalid after the move.

**Q4: b** - The borrow checker prevents data races, use-after-free, and double-free errors at compile time.

**Q5: c** - Values are automatically dropped when their owner goes out of scope (no manual memory management needed).

### Section 2: Code Analysis

**Q6: b** - No, won't compile. `s` is moved into `takes_ownership()`, so it's invalid in main afterward.

**Q7: c** - Yes! `.clone()` makes a deep copy, so both s1 and s2 are valid.

**Q8: b** - Prints "5, 5". i32 implements Copy, so x is copied (not moved) to y.

**Q9: b or d** - Either change function to take `&String` (borrowing), or return the String with the length as a tuple. Option b is more idiomatic.

### Section 3: Borrowing & References

**Q10: b** - `&T` is an immutable reference (read-only), `&mut T` is a mutable reference (can modify).

**Q11: b** - Exactly **one** mutable reference at a time (prevents data races).

**Q12: b** - Won't compile. Can't have a mutable reference (r3) while immutable references (r1, r2) are still in use.

**Q13: a** - Safe because of **non-lexical lifetimes** (NLL). r1 and r2 aren't used after the println!, so r3 can exist.

### Bonus Section

**Bonus 1: b** - Lifetimes ensure references don't outlive the data they point to.

**Bonus 2: d** - Only example2 needs annotations because the compiler can't determine which input's lifetime the output relates to.

</details>

---

## Recommendations Based on Score

### 90-100% - Excellent!
You've mastered ownership! This puts you ahead of most Rust learners. Key achievement: You understand the borrow checker.

**Next Steps:**
- Proceed confidently to Module 5
- You're ready for real Rust development
- Consider helping others in forums

### 80-89% - Good!
You understand the core concepts with minor gaps.

**Review:**
- Lesson 4: Immutable Borrowing
- Lesson 5: Mutable Borrowing
- Retake quiz in 1-2 days

### 70-79% - Fair
You grasp basics but need more practice.

**Review:**
- Lessons 2-3: Ownership rules and errors
- Lessons 4-5: Borrowing
- Complete Practice Project again
- Retake quiz after review

### Below 70% - Needs Review
Ownership is challenging! Don't feel discouraged.

**Action Plan:**
1. Re-read Module 4 completely
2. Type every code example yourself
3. Complete the Text Processor project
4. Try the exercises again
5. Join [Rust Users Forum](https://users.rust-lang.org/) for help
6. Retake quiz after thorough review

**Remember**: Module 4 is THE hardest module. Many struggle here. Mastery comes with practice!

---

## Additional Resources

- [The Rust Book - Chapter 4](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html)
- [Rust by Example - Ownership](https://doc.rust-lang.org/rust-by-example/scope/move.html)
- [Visualizing Ownership](https://www.youtube.com/watch?v=VFIOSWy93H0) (YouTube)
- [Rustlings Exercises](https://github.com/rust-lang/rustlings)

---

[← Back to Quiz System](QUIZ_SYSTEM.md) | [Module 4 Lessons →](../module_04_ownership/)
