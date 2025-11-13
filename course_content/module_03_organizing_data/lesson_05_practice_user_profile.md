# Module 3, Lesson 5: Practice Project — Creating a User Profile System

## Project Overview

Build a simple user profile management system that demonstrates structs, enums, Option, and Result!

**What we're building:**
- User profiles with personal information
- Different account types (Regular, Premium, Admin)
- Validation for user data
- Methods to display and modify profiles

**Skills practiced:**
- ✅ Structs to organize data
- ✅ Enums for account types
- ✅ Option for optional fields
- ✅ Result for error handling
- ✅ Methods on structs

## The Complete Project

### **Step 1: Create the Project**

```bash
cargo new user_profiles
cd user_profiles
code .
```

### **Step 2: Build the System (Complete Code)**

Replace `src/main.rs` with:

```rust
// Define account types
#[derive(Debug)]
enum AccountType {
    Regular,
    Premium,
    Admin,
}

// Define user profile struct
#[derive(Debug)]
struct UserProfile {
    username: String,
    email: String,
    age: Option<u32>,           // Age is optional
    account_type: AccountType,
    bio: Option<String>,        // Bio is optional
}

impl UserProfile {
    // Constructor with validation
    fn new(
        username: String,
        email: String,
        age: Option<u32>,
        account_type: AccountType,
    ) -> Result<UserProfile, String> {
        // Validate username
        if username.is_empty() {
            return Err(String::from("Username cannot be empty"));
        }

        // Validate email (basic check)
        if !email.contains('@') {
            return Err(String::from("Invalid email format"));
        }

        // Validate age if provided
        if let Some(a) = age {
            if a > 120 {
                return Err(String::from("Age must be 120 or less"));
            }
        }

        Ok(UserProfile {
            username,
            email,
            age,
            account_type,
            bio: None,  // Start with no bio
        })
    }

    // Display user info
    fn display(&self) {
        println!("\n=== User Profile ===");
        println!("Username: {}", self.username);
        println!("Email: {}", self.email);

        match &self.age {
            Some(a) => println!("Age: {}", a),
            None => println!("Age: Not specified"),
        }

        println!("Account: {:?}", self.account_type);

        match &self.bio {
            Some(b) => println!("Bio: {}", b),
            None => println!("Bio: Not set"),
        }
        println!("===================\n");
    }

    // Set bio
    fn set_bio(&mut self, bio: String) -> Result<(), String> {
        if bio.len() > 200 {
            Err(String::from("Bio too long (max 200 characters)"))
        } else {
            self.bio = Some(bio);
            Ok(())
        }
    }

    // Check if user has premium features
    fn has_premium_access(&self) -> bool {
        matches!(self.account_type, AccountType::Premium | AccountType::Admin)
    }

    // Upgrade to premium
    fn upgrade_to_premium(&mut self) -> Result<(), String> {
        match self.account_type {
            AccountType::Regular => {
                self.account_type = AccountType::Premium;
                Ok(())
            }
            AccountType::Premium => Err(String::from("Already premium")),
            AccountType::Admin => Err(String::from("Admins don't need upgrades")),
        }
    }
}

fn main() {
    println!("=== User Profile System ===\n");

    // Create valid user
    let user1 = UserProfile::new(
        String::from("alice"),
        String::from("alice@example.com"),
        Some(25),
        AccountType::Regular,
    );

    match user1 {
        Ok(mut user) => {
            user.display();

            // Set bio
            match user.set_bio(String::from("I love Rust programming!")) {
                Ok(()) => println!("Bio updated successfully"),
                Err(e) => println!("Error: {}", e),
            }

            user.display();

            // Check premium access
            if user.has_premium_access() {
                println!("User has premium features");
            } else {
                println!("User has basic features");
            }

            // Try to upgrade
            match user.upgrade_to_premium() {
                Ok(()) => println!("Upgraded to premium!"),
                Err(e) => println!("Upgrade error: {}", e),
            }

            user.display();
        }
        Err(e) => println!("Error creating user: {}", e),
    }

    // Try to create invalid users
    println!("\n=== Testing Validation ===\n");

    let invalid_email = UserProfile::new(
        String::from("bob"),
        String::from("not-an-email"),
        Some(30),
        AccountType::Regular,
    );

    match invalid_email {
        Ok(_) => println!("User created"),
        Err(e) => println!("Error: {}", e),
    }

    let invalid_age = UserProfile::new(
        String::from("charlie"),
        String::from("charlie@example.com"),
        Some(150),
        AccountType::Regular,
    );

    match invalid_age {
        Ok(_) => println!("User created"),
        Err(e) => println!("Error: {}", e),
    }

    // Create admin user
    let admin = UserProfile::new(
        String::from("admin"),
        String::from("admin@example.com"),
        None,  // Age not specified
        AccountType::Admin,
    );

    if let Ok(user) = admin {
        user.display();
        println!("Has premium access: {}", user.has_premium_access());
    }
}
```

### **Step 3: Run the Project**

```bash
cargo run
```

**Expected output:**
```
=== User Profile System ===

=== User Profile ===
Username: alice
Email: alice@example.com
Age: 25
Account: Regular
Bio: Not set
===================

Bio updated successfully

=== User Profile ===
Username: alice
Email: alice@example.com
Age: 25
Account: Regular
Bio: I love Rust programming!
===================

User has basic features
Upgraded to premium!

=== User Profile ===
Username: alice
Email: alice@example.com
Age: 25
Account: Premium
Bio: I love Rust programming!
===================

=== Testing Validation ===

Error: Invalid email format
Error: Age must be 120 or less

=== User Profile ===
Username: admin
Email: admin@example.com
Age: Not specified
Account: Admin
Bio: Not set
===================

Has premium access: true
```

## Understanding the Code

**1. Using enums for account types:**
```rust
enum AccountType {
    Regular,
    Premium,
    Admin,
}
```

**2. Using Option for optional fields:**
```rust
age: Option<u32>,      // User might not provide age
bio: Option<String>,   // Bio might not be set
```

**3. Using Result for validation:**
```rust
fn new(...) -> Result<UserProfile, String> {
    if username.is_empty() {
        return Err(String::from("Username cannot be empty"));
    }
    Ok(UserProfile { ... })
}
```

**4. Pattern matching on enums:**
```rust
match self.account_type {
    AccountType::Regular => { /* upgrade */ },
    AccountType::Premium => Err(...),
    AccountType::Admin => Err(...),
}
```

## Challenges and Extensions

### **Challenge 1: Add More Validation**

Add validation for:
- Username must be at least 3 characters
- Username can only contain letters, numbers, underscore
- Email must contain a dot after the @

### **Challenge 2: Add Friend System**

Add a `friends` field (Vec<String>) and methods:
- `add_friend(username: String) -> Result<(), String>`
- `remove_friend(username: String) -> Result<(), String>`
- `list_friends()`

### **Challenge 3: Add User Statistics**

Add fields:
- `post_count: u32`
- `joined_date: String`

Add methods:
- `increment_posts()`
- `get_days_since_joined() -> u32`

### **Challenge 4: Save/Load from File**

Use `std::fs` to save profiles to a text file and load them back.

## Key Takeaways

- ✅ Structs organize related data
- ✅ Enums represent distinct states
- ✅ Option handles optional data
- ✅ Result handles operations that can fail
- ✅ Methods encapsulate behavior
- ✅ Pattern matching handles all cases
- ✅ Validation at construction prevents invalid states

---

## ✅ Module 3 Complete!

You've mastered data organization:
- ✅ Structs (grouping data)
- ✅ Enums (variants)
- ✅ Option (maybe values)
- ✅ Result (error handling)
- ✅ Built a complete user profile system!

**Next: Module 4 — The Ownership System (Rust's superpower)!**
