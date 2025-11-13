# 📝 Module 7: File I/O - Assessment Quiz

**Time Limit**: 20-25 minutes
**Passing Score**: 80% (24/30 points)
**Difficulty**: ⭐⭐⭐⭐ Practical Skills

---

## Instructions

This quiz tests your understanding of Rust's file I/O system including command-line arguments, file reading/writing, and path manipulation. Answer all questions honestly without referring to the lessons. Answers and explanations are at the bottom.

---

## Section 1: Command-Line Arguments (8 points)

### Question 1 (2 points)
**What is always at index 0 of command-line arguments?**

a) The first user-provided argument
b) The program name or path
c) The current directory
d) Nothing, it's empty

**Your Answer**: _____

---

### Question 2 (3 points)
**How do you access command-line arguments in Rust?**

a) `std::args::get()`
b) `std::env::args()`
c) `std::cmd::args()`
d) `process::args()`

**Your Answer**: _____

---

### Question 3 (3 points)
**What does this code do?**

```rust
let args: Vec<String> = env::args().collect();
if args.len() < 2 {
    return;
}
let name = &args[1];
```

a) Gets the program name
b) Gets the first user-provided argument
c) Counts all arguments
d) Compile error

**Your Answer**: _____

---

## Section 2: Reading Files (8 points)

### Question 4 (2 points)
**What's the simplest way to read an entire file as a String?**

a) `fs::read(path)`
b) `fs::read_to_string(path)`
c) `File::open(path).read()`
d) `fs::load(path)`

**Your Answer**: _____

---

### Question 5 (3 points)
**What type does `fs::read_to_string()` return?**

a) `String`
b) `Option<String>`
c) `Result<String, std::io::Error>`
d) `Vec<u8>`

**Your Answer**: _____

---

### Question 6 (3 points)
**When should you use BufReader instead of read_to_string()?**

a) For small files
b) For large files or line-by-line processing
c) For binary files only
d) Never, read_to_string is always better

**Your Answer**: _____

---

## Section 3: Writing Files (8 points)

### Question 7 (2 points)
**What does `fs::write()` do if the file already exists?**

a) Appends to it
b) Returns an error
c) Overwrites it completely
d) Creates a backup first

**Your Answer**: _____

---

### Question 8 (3 points)
**How do you append to a file instead of overwriting?**

a) `fs::append(path, content)`
b) `fs::write(path, content, true)`
c) Use `OpenOptions` with `.append(true)`
d) `fs::add(path, content)`

**Your Answer**: _____

---

### Question 9 (3 points)
**What does this code do?**

```rust
use std::fs::OpenOptions;
use std::io::Write;

let mut file = OpenOptions::new()
    .create(true)
    .append(true)
    .open("log.txt")?;

writeln!(file, "Entry")?;
```

a) Overwrites log.txt with "Entry"
b) Creates log.txt if needed and appends "Entry\n"
c) Only works if log.txt exists
d) Compile error

**Your Answer**: _____

---

## Section 4: Paths (6 points)

### Question 10 (2 points)
**What's the difference between Path and PathBuf?**

a) No difference
b) Path is borrowed, PathBuf is owned
c) Path is for files, PathBuf is for directories
d) PathBuf is deprecated

**Your Answer**: _____

---

### Question 11 (2 points)
**How do you safely combine path components?**

a) Use string concatenation with "/"
b) Use `.join()` method
c) Use `+` operator
d) Use `Path::combine()`

**Your Answer**: _____

---

### Question 12 (2 points)
**What does `.extension()` return for "document.txt"?**

a) `String` with value "txt"
b) `Option<&OsStr>`
c) `"txt"`
d) `.txt`

**Your Answer**: _____

---

## Bonus Section: Advanced Concepts (5 points)

### Bonus Question 1 (3 points)
**Will this code work on both Windows and Unix?**

```rust
let path = Path::new("data").join("files").join("doc.txt");
```

a) No, needs different code for each OS
b) Yes, Rust handles path separators automatically
c) Only works on Unix
d) Only works on Windows

**Your Answer**: _____

---

### Bonus Question 2 (2 points)
**What does `fs::create_dir_all()` do differently from `fs::create_dir()`?**

a) They're the same
b) `create_dir_all` creates parent directories as needed
c) `create_dir_all` is faster
d) `create_dir_all` works with files too

**Your Answer**: _____

---

## Scoring

**Calculate your score:**
- Section 1: ____ / 8 points
- Section 2: ____ / 8 points
- Section 3: ____ / 8 points
- Section 4: ____ / 6 points
- Bonus: ____ / 5 points
- **Total: ____ / 30 points (____ %)**

**Performance Evaluation:**
- 27-30 (90-100%): **Excellent!** File I/O mastered
- 24-26 (80-89%): **Good!** Minor review needed
- 21-23 (70-79%): **Fair**. Review key concepts
- Below 21 (< 70%): **Review module thoroughly**

---

## Answer Key & Explanations

<details>
<summary>Click to reveal answers (try the quiz first!)</summary>

### Section 1: Command-Line Arguments

**Q1: b** - Index 0 always contains the program name or path. User arguments start at index 1.

**Q2: b** - `std::env::args()` returns an iterator over command-line arguments. Call `.collect()` to get a Vec.

**Q3: b** - This gets the first user-provided argument (index 1). Index 0 is the program name, so index 1 is the first actual argument from the user.

### Section 2: Reading Files

**Q4: b** - `fs::read_to_string(path)` is the simplest way to read an entire file into a String. Returns `Result<String, io::Error>`.

**Q5: c** - Returns `Result<String, std::io::Error>` because file operations can fail. Must be handled with `?`, `match`, or `unwrap`.

**Q6: b** - Use `BufReader` for large files to avoid loading everything into memory at once, or when you need line-by-line processing. `read_to_string` loads the entire file into memory.

### Section 3: Writing Files

**Q7: c** - `fs::write()` completely overwrites the file if it exists. To append, use `OpenOptions` with `.append(true)`.

**Q8: c** - Must use `OpenOptions::new().append(true).open(path)` to append instead of overwriting. There's no `fs::append()` function.

**Q9: b** - `.create(true)` creates file if it doesn't exist, `.append(true)` appends instead of overwriting, and `writeln!` adds content with a newline.

### Section 4: Paths

**Q10: b** - `Path` is borrowed (like `&str`), `PathBuf` is owned (like `String`). Use `Path` for read-only operations, `PathBuf` when you need to build or modify paths.

**Q11: b** - Use `.join()` method which handles path separators correctly across platforms. Never concatenate paths as strings!

**Q12: b** - Returns `Option<&OsStr>` because files might not have extensions. Need to use `.and_then(|ext| ext.to_str())` to get a string.

### Bonus Section

**Bonus 1: b** - Yes! Rust's `Path` and `PathBuf` automatically use the correct path separator for the OS (`/` on Unix, `\` on Windows). This makes code cross-platform.

**Bonus 2: b** - `create_dir_all` creates all parent directories as needed (like `mkdir -p`), while `create_dir` only creates the final directory and errors if parents don't exist.

</details>

---

## Recommendations Based on Score

### 90-100% - Excellent!
You've mastered file I/O! You can confidently build applications that work with files and the file system.

**Next Steps:**
- Proceed confidently to Module 8
- Build file-based applications
- Explore advanced I/O patterns

### 80-89% - Good!
You understand core concepts with minor gaps.

**Review:**
- Lesson 3: Writing Files (append vs overwrite)
- Lesson 4: Working with Paths
- Retake quiz in 1-2 days

### 70-79% - Fair
You grasp basics but need more practice.

**Review:**
- Lessons 2-3: Reading and writing files
- Lesson 4: Path manipulation
- Complete practice project again
- Retake quiz after review

### Below 70% - Needs Review
File I/O is essential for most real-world applications!

**Action Plan:**
1. Re-read Module 7 completely
2. Type every code example yourself
3. Complete the File Tool project
4. Try all the challenges
5. Practice with real files
6. Retake quiz after thorough review

**Remember**: Working with files is a fundamental skill. Almost every real application needs to read or write data!

---

## Additional Resources

- [The Rust Book - Reading a File](https://doc.rust-lang.org/book/ch12-02-reading-a-file.html)
- [Rust by Example - File I/O](https://doc.rust-lang.org/rust-by-example/std_misc/file.html)
- [std::fs documentation](https://doc.rust-lang.org/std/fs/)
- [std::path documentation](https://doc.rust-lang.org/std/path/)
- [clap documentation](https://docs.rs/clap/)

---

[← Back to Quiz System](QUIZ_SYSTEM.md) | [Module 7 Lessons →](../module_07_file_io/)
