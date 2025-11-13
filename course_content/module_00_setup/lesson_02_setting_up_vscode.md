# Module 0, Lesson 2: Setting up Visual Studio Code with rust-analyzer

## Why Do We Need This?

Imagine you're writing an essay by hand with pen and paper. It works, but it's slow. You have to manually check for spelling mistakes, you can't easily move paragraphs around, and if you want to know what a word means, you have to stop and look it up in a dictionary.

Now imagine writing that same essay on a modern computer with a word processor. It automatically:
- Underlines spelling mistakes as you type
- Suggests better words
- Shows you definitions when you hover over words
- Lets you easily reorganize your text

Writing code is exactly the same. You *could* write Rust programs in Notepad, but you'd be making your life much harder. Instead, we use a special "code editor" that watches what you're typing and helps you in real-time.

## The Tools We're Installing

**Visual Studio Code** (VS Code for short):
- A free, professional code editor
- Works on Windows, macOS, and Linux
- The most popular code editor in the world
- Think of it as "Microsoft Word, but for code"

**rust-analyzer**:
- An extension (plugin) that teaches VS Code to understand Rust
- Shows you errors as you type (before you even run the code)
- Suggests how to complete your code
- Explains what your code does when you hover over it
- Your "Rust expert assistant" that sits next to you while you code

## Installation Steps

### **Step 1: Install Visual Studio Code**

1. Go to: https://code.visualstudio.com/
2. Click the big download button for your operating system
3. Run the installer and follow the default options
4. Launch VS Code when installation completes

### **Step 2: Install the rust-analyzer Extension**

1. In VS Code, click the **Extensions** icon on the left sidebar (looks like four squares)
   - Keyboard shortcut: `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (macOS)
2. In the search box, type: **rust-analyzer**
3. Find the extension named **"rust-analyzer"** by matklad (should be the first result)
4. Click the blue **"Install"** button
5. Wait for it to finish installing

### **Step 3: Test That It's Working**

Let's verify rust-analyzer is active and helping you:

1. In VS Code: **File → Open Folder**
2. Create a new empty folder anywhere on your computer called `rust-test`
3. Open that folder in VS Code
4. Create a new file named `test.rs` (click "New File" in VS Code)
5. Type this code exactly:

```rust
fn main() {
    let x = 5;
    x = 10;
}
```

6. **Look at line 3** (`x = 10;`) — you should see a **red squiggly underline**
7. **Hover your mouse** over that underlined code
8. **You should see an error message** appear in a tooltip!

The error message will say something like: *"cannot assign twice to immutable variable"*

Don't worry about what that means yet—we'll learn about it in Module 1. For now, if you see that error message pop up, **congratulations!** Your rust-analyzer is working perfectly.

## What Just Happened?

rust-analyzer analyzed your code in real-time and detected a problem before you even tried to run it. This is like having a spell-checker for code. Throughout this course, rust-analyzer will be your constant companion, catching mistakes and teaching you Rust's rules.

## Troubleshooting

**No red underline appears:**
- Wait 10-20 seconds—rust-analyzer needs a moment to start up the first time
- Make sure your file is named with `.rs` extension (that's how VS Code knows it's Rust code)
- Check the bottom-right corner of VS Code for any rust-analyzer status messages

**Extension not found:**
- Make sure you spelled it correctly: `rust-analyzer` (with a hyphen)
- Ensure you have internet connection

---

## ✅ Lesson Complete!

Your development environment is now fully set up. You have:
- ✅ VS Code (your code editor)
- ✅ rust-analyzer (your Rust assistant)

You're ready to write actual Rust code!
