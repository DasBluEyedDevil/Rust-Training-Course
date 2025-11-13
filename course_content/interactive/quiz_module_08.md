# 📝 Module 8: Traits & Generics - Assessment Quiz

**Time Limit**: 25-30 minutes
**Passing Score**: 80% (24/30 points)
**Difficulty**: ⭐⭐⭐⭐⭐ Advanced Concepts

---

## Instructions

This quiz tests your understanding of Rust's trait system, generics, and iterators. Answer all questions honestly without referring to the lessons. Answers and explanations are at the bottom.

---

## Section 1: Traits (10 points)

### Question 1 (2 points)
**What is a trait in Rust?**

a) A type of variable
b) A definition of shared behavior that types can implement
c) A way to create classes
d) A memory management tool

**Your Answer**: _____

---

### Question 2 (3 points)
**How do you implement a trait for a type?**

a) `trait TypeName for TraitName { }`
b) `impl TraitName for TypeName { }`
c) `implement TraitName on TypeName { }`
d) `TypeName implements TraitName { }`

**Your Answer**: _____

---

### Question 3 (2 points)
**Can traits have default method implementations?**

a) No, all methods must be implemented by types
b) Yes, traits can provide default implementations
c) Only if marked with #[default]
d) Only for Debug and Display traits

**Your Answer**: _____

---

### Question 4 (3 points)
**What does `&impl Trait` mean in a function parameter?**

a) A reference to the Trait itself
b) A reference to any type implementing Trait
c) Implementation details
d) Compile error

**Your Answer**: _____

---

## Section 2: Generics (10 points)

### Question 5 (2 points)
**What does `<T>` represent in `fn process<T>(value: T)`?**

a) A specific type T
b) A type parameter that can be any type
c) A template
d) A trait

**Your Answer**: _____

---

### Question 6 (3 points)
**What is a trait bound?**

```rust
fn process<T: Display>(value: T) {
    println!("{}", value);
}
```

a) T must equal Display
b) T must implement the Display trait
c) T must be a string
d) T must be printable

**Your Answer**: _____

---

### Question 7 (2 points)
**Are generics a runtime or compile-time feature?**

a) Runtime - types determined during execution
b) Compile-time - monomorphization creates specialized code
c) Both runtime and compile-time
d) Neither, they're interpreted

**Your Answer**: _____

---

### Question 8 (3 points)
**Will this compile?**

```rust
struct Pair<T> {
    first: T,
    second: T,
}

let mixed = Pair {
    first: 5,
    second: "hello",
};
```

a) Yes, T can be different types
b) No, both fields must be the same type T
c) Yes, but with a warning
d) Only if T implements Copy

**Your Answer**: _____

---

## Section 3: Common Traits (5 points)

### Question 9 (2 points)
**Which trait must be implemented to use `println!("{:?}", value)`?**

a) Display
b) Debug
c) Print
d) Format

**Your Answer**: _____

---

### Question 10 (3 points)
**What's the difference between Clone and Copy?**

a) No difference
b) Clone is explicit (.clone()), Copy is implicit (automatic)
c) Clone is faster
d) Copy works with heap data

**Your Answer**: _____

---

## Section 4: Iterators (5 points)

### Question 11 (2 points)
**What does `.iter()` return?**

a) The collection itself
b) An iterator over immutable references
c) An iterator over mutable references
d) An iterator that takes ownership

**Your Answer**: _____

---

### Question 12 (3 points)
**What does this code produce?**

```rust
let numbers = vec![1, 2, 3, 4, 5];
let result: Vec<i32> = numbers
    .iter()
    .map(|x| x * 2)
    .filter(|x| *x > 5)
    .collect();
```

a) [2, 4, 6, 8, 10]
b) [6, 8, 10]
c) [3, 4, 5]
d) Compile error

**Your Answer**: _____

---

## Bonus Section: Advanced Understanding (5 points)

### Bonus Question 1 (3 points)
**What is monomorphization?**

a) Making code polymorphic
b) Compiler generating specialized code for each concrete type used with generics
c) Converting traits to types
d) A runtime optimization

**Your Answer**: _____

---

### Bonus Question 2 (2 points)
**Which is true about iterator adapters (map, filter, etc.)?**

a) They execute immediately
b) They're lazy - only execute when consumed
c) They're slower than loops
d) They can't be chained

**Your Answer**: _____

---

## Scoring

**Calculate your score:**
- Section 1: ____ / 10 points
- Section 2: ____ / 10 points
- Section 3: ____ / 5 points
- Section 4: ____ / 5 points
- Bonus: ____ / 5 points
- **Total: ____ / 30 points (____ %)**

**Performance Evaluation:**
- 27-30 (90-100%): **Excellent!** Advanced concepts mastered
- 24-26 (80-89%): **Good!** Minor review needed
- 21-23 (70-79%): **Fair**. Review key concepts
- Below 21 (< 70%): **Review module thoroughly**

---

## Answer Key & Explanations

<details>
<summary>Click to reveal answers (try the quiz first!)</summary>

### Section 1: Traits

**Q1: b** - A trait defines shared behavior that multiple types can implement. Like interfaces in other languages, traits specify methods that implementing types must provide.

**Q2: b** - Use `impl TraitName for TypeName { }` to implement a trait for a type. This syntax reads: "implement TraitName for TypeName".

**Q3: b** - Yes! Traits can provide default implementations for methods. Types can use the default or override it with their own implementation.

**Q4: b** - `&impl Trait` means the parameter is a reference to any type that implements the Trait. It's syntactic sugar for trait bounds.

### Section 2: Generics

**Q5: b** - `<T>` declares a type parameter - a placeholder for any type. When you use the function, Rust fills in T with the actual type.

**Q6: b** - `: Display` is a trait bound that constrains T to types implementing Display. This ensures you can use Display methods on T.

**Q7: b** - Generics are a compile-time feature. Rust uses monomorphization to generate specialized code for each concrete type, resulting in zero runtime cost!

**Q8: b** - No, won't compile. Both `first` and `second` must be the same type T. To allow different types, you'd need `Pair<T, U>` with two type parameters.

### Section 3: Common Traits

**Q9: b** - The `Debug` trait enables `{:?}` formatting. Use `#[derive(Debug)]` for automatic implementation or implement manually for custom formatting.

**Q10: b** - `Clone` requires explicit `.clone()` call and works with heap data. `Copy` is implicit (automatic) and only works with types that are fully stack-allocated.

### Section 4: Iterators

**Q11: b** - `.iter()` returns an iterator over immutable references (`&T`). Use `.iter_mut()` for mutable references or `.into_iter()` to take ownership.

**Q12: b** - Result is `[6, 8, 10]`.
- Map: [1, 2, 3, 4, 5] → [2, 4, 6, 8, 10]
- Filter (> 5): [2, 4, 6, 8, 10] → [6, 8, 10]
- Collect: Creates Vec<i32>

### Bonus Section

**Bonus 1: b** - Monomorphization is when the compiler generates specialized code for each concrete type used with a generic function. If you call `process(5)` and `process(3.14)`, the compiler creates `process_i32()` and `process_f64()`. This is why generics have zero runtime cost!

**Bonus 2: b** - Iterator adapters are lazy - they don't execute until a consumer (like `collect()`, `sum()`, `count()`) is called. This allows chaining multiple transformations efficiently without creating intermediate collections.

</details>

---

## Recommendations Based on Score

### 90-100% - Excellent!
You've mastered advanced Rust! You understand traits, generics, and iterators at a deep level.

**Next Steps:**
- Proceed confidently to the Capstone Project
- You're ready to build complex Rust applications
- Explore advanced crate ecosystems (tokio, serde, etc.)

### 80-89% - Good!
You understand core concepts with minor gaps.

**Review:**
- Lesson 2: Generics and trait bounds
- Lesson 4: Iterator chains and laziness
- Retake quiz in 1-2 days

### 70-79% - Fair
You grasp basics but need more practice.

**Review:**
- Lessons 1-2: Traits and Generics fundamentals
- Lesson 4: Iterator methods
- Complete practice project again
- Study the trait bound syntax
- Retake quiz after review

### Below 70% - Needs Review
These concepts are essential for idiomatic Rust!

**Action Plan:**
1. Re-read Module 8 completely
2. Type every code example yourself
3. Complete the Generic Filter project
4. Try all the challenges
5. Experiment with trait implementations
6. Practice iterator chains with real data
7. Retake quiz after thorough review

**Remember**: Traits and generics are what make Rust powerful and ergonomic. They enable zero-cost abstractions and type-safe polymorphism!

---

## Additional Practice

Try implementing these:

1. **Custom Iterator**: Create an iterator that generates Fibonacci numbers
2. **Generic Stack**: Build a generic stack with push/pop/peek
3. **Trait Inheritance**: Create traits that build on other traits
4. **Multiple Trait Bounds**: Write functions requiring multiple trait implementations
5. **Iterator Chains**: Process real data (CSV file) using only iterator methods

---

## Additional Resources

- [The Rust Book - Traits](https://doc.rust-lang.org/book/ch10-02-traits.html)
- [The Rust Book - Generic Types](https://doc.rust-lang.org/book/ch10-01-syntax.html)
- [Rust by Example - Traits](https://doc.rust-lang.org/rust-by-example/trait.html)
- [Rust by Example - Generics](https://doc.rust-lang.org/rust-by-example/generics.html)
- [Iterator Documentation](https://doc.rust-lang.org/std/iter/trait.Iterator.html)
- [Common Traits](https://doc.rust-lang.org/std/index.html#primitives)

---

[← Back to Quiz System](QUIZ_SYSTEM.md) | [Module 8 Lessons →](../module_08_traits_generics/)
