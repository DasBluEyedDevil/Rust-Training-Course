# Module 0, Lesson 1: Installing Rust (rustup and cargo)

## Why Do We Need This?

Imagine you want to build furniture. Before you can start, you need a workshop with the right tools. You wouldn't build a chair without a saw, a hammer, and some nails, right?

Programming is the same way. Before you can write programs in any language, you need the special tools that understand that language. These tools will:
- Translate your human-readable instructions into something the computer can understand
- Check your work for mistakes before running it
- Help you organize your projects

We're going to install the complete set of tools you need to write programs in Rust. Think of it as setting up your coding workshop.

## The Official Names

The "workshop setup" we just described has official names:

- **Rustup**: This is the "tool installer and manager." It downloads Rust and keeps it up to date. Think of it as the workshop manager who makes sure you always have the latest, best versions of your tools.

- **Cargo**: This is Rust's "project manager and build tool." It helps you create new projects, manage external code libraries (called "crates"), and compile your code into a running program. Cargo is like your workshop assistant who organizes everything.

- **Rustc**: This is the actual "compiler" (the translator that turns your Rust code into a program the computer can run). You won't use this directly most of the time—Cargo will use it for you.

When you install **rustup**, it automatically installs both **cargo** and **rustc**. It's a three-for-one deal!

## How to Install

### **For Windows Users:**

1. Go to: https://rustup.rs/
2. Download the `rustup-init.exe` file
3. Run the downloaded file
4. When asked, choose option `1` (default installation)
5. Wait for installation to complete (this might take a few minutes)
6. **Close and reopen** your terminal (this is important!)

### **For macOS/Linux Users:**

1. Open your terminal
2. Copy and paste this command:
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
3. Press Enter and follow the prompts
4. When asked, choose option `1` (default installation)
5. Wait for installation to complete
6. **Close and reopen** your terminal

## Verify Your Installation

After installation, open a **new** terminal and run these commands one at a time:

```bash
rustup --version
cargo --version
rustc --version
```

**You should see something like this** (your numbers might be slightly different):
```
rustup 1.27.0 (...)
cargo 1.75.0 (...)
rustc 1.75.0 (...)
```

If you see three version numbers, **congratulations!** Your Rust workshop is ready.

## Troubleshooting

**"Command not found" errors:**
- Make sure you closed and reopened your terminal after installation
- On Windows, try restarting your computer if the commands still don't work

**Windows: "Visual Studio C++ Build tools" error:**
- Follow the link in the error message to install the required Microsoft tools
- This is normal for Windows—Rust needs these to compile programs

---

## ✅ Lesson Complete!

You now have Rust installed and ready to use. In the next lesson, we'll set up a code editor to make writing Rust code much easier.
