# What Actually Matters

This chapter covers patterns and practices that consistently deliver reliable results in production Rust environments. These practices help teams avoid common pitfalls and build more maintainable systems.

## Let the Compiler Help You

In production environments, working against the compiler is common. Experience shows that the compiler's feedback often highlights genuine design issues rather than simple syntax problems.

Consider this example where the compiler catches a potential issue:

Filename: src/main.rs
```rust
fn main() {
    let data = vec![1, 2, 3];
    let first = data[0]; // This compiles
    let second = data[1]; // This compiles
    // Later in the code...
    println!("First: {}, Second: {}", first, second);
}
```

The compiler will suggest using `get()` method for safer access, which leads to better error handling:

Filename: src/main.rs
```rust
fn main() {
    let data = vec![1, 2, 3];
    let first = data.get(0).unwrap_or(&0); // Safer access
    let second = data.get(1).unwrap_or(&0);
    println!("First: {}, Second: {}", first, second);
}
```

## Abstractions Without the Cost

A key advantage of Rust is that you can write high-level code that compiles down to something efficient. The iterator chains you write become tight loops. The Option and Result types have zero overhead in release mode.

Here's an example showing how high-level Rust code compiles to efficient machine code:

Filename: src/main.rs
```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let sum: i32 = numbers.iter().map(|x| x * 2).sum();
    println!("Sum: {}", sum);
}
```

This iterator chain compiles down to something similar to:

Filename: src/main.rs
```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    let mut sum = 0;
    for x in numbers.iter() {
        sum += x * 2;
    }
    println!("Sum: {}", sum);
}
```

The compiler optimizes the iterator chain into a tight loop, giving you both readability and performance.

## Refactor Without Fear

This approach significantly improves team productivity. You can make significant changes to your codebase and trust that if it compiles, it probably works. The type system catches so many issues that would have been runtime errors in other languages.

Consider this refactoring example:

Filename: src/main.rs
```rust
// Before refactoring
struct User {
    name: String,
    age: u32,
    email: String,
}

fn send_welcome_email(user: &User) {
    println!("Welcome {}!", user.name);
}

// After refactoring - extracting name into its own struct
struct Name(String);
struct User {
    name: Name,
    age: u32,
    email: String,
}

fn send_welcome_email(user: &User) {
    println!("Welcome {}!", user.name.0);
}
```

In many languages, this refactoring would be risky because you might miss updating all the places where `user.name` was used. In Rust, the compiler will catch every place where you need to update the code to use `user.name.0` instead of `user.name`. This safety net enables confident refactoring.

## Performance Isn't an Afterthought

In most languages, you optimize when you have performance problems. In Rust, you start with something that's already pretty fast, and you optimize from there. It's a completely different mindset.

Here's an example showing Rust's performance mindset:

Filename: src/main.rs
```rust
use std::time::Instant;

fn main() {
    let start = Instant::now();
    
    // This is already fast - zero-cost abstractions
    let numbers: Vec<i32> = (0..1_000_000).collect();
    let sum: i64 = numbers.iter().map(|&x| x as i64).sum();
    
    let duration = start.elapsed();
    println!("Sum: {}, Time: {:?}", sum, duration);
}
```

Even without optimization flags, this code performs well because:
- Iterator chains compile to efficient loops
- No runtime garbage collection overhead
- Zero-cost abstractions mean you pay only for what you use

Compare this to languages where you might need to reach for specialized libraries or write manual loops to get similar performance.

Next: [Project Structure](./rust-practices-02-project-structure.md)