+++
date = '2026-11-13T15:50:06Z'
draft = false
title = 'Learning Rust 0.1'
+++

# Phase 0 and Module 1: Setup and Rust Basics for a Java Developer

Your Java background helps a lot here. Static typing, compilation, and structured code will all feel familiar. The main adjustments come from habits Java taught you that Rust does differently, so I'll point those out as we go.

---

## Phase 0: Setup

Install Rust from **rustup.rs**. On Linux or macOS, this is a single curl command. On Windows, download the installer. Then check that it worked:

```bash
rustc --version
cargo --version
```

Here's how the Rust tooling maps to what you know from Java:

| Java | Rust |
|---|---|
| Maven / Gradle | `cargo` (build tool, dependency manager, test runner, and docs generator in one) |
| `pom.xml` / `build.gradle` | `Cargo.toml` |
| Maven Central | crates.io |
| JAR library | crate |
| `javac` | `rustc` (you'll rarely call it directly) |
| Checkstyle / SpotBugs | `clippy` |
| IntelliJ inspections | `rust-analyzer` (install it in VS Code or use RustRover) |

Create your first project:

```bash
cargo new hello_rust
cd hello_rust
cargo run
```

The commands you'll use every day are:
- **`cargo check`**: type-checks your code quickly without building a binary. Use this constantly.
- **`cargo run`**: builds and runs the program.
- **`cargo test`**: runs your tests.
- **`cargo clippy`**: points out code that isn't idiomatic.
- **`cargo fmt`**: formats your code automatically.

There's one big difference from Java: Rust compiles to a native binary. There's no JVM, no garbage collector, and no runtime warm-up.

---

## Module 1: Syntax and Basics

### 1. Variables are immutable by default

```rust
fn main() {
    let x = 5;          // like `final int x = 5;` in Java
    // x = 6;           // compile error!
    let mut y = 10;     // explicitly mutable
    y += 1;
    println!("x = {x}, y = {y}");
}
```

In Java, a variable is mutable unless you mark it `final`. Rust reverses that: everything is immutable unless you write `mut`. This choice comes up again and again in Rust's design.

**Shadowing** has no Java equivalent. You can redeclare a variable with the same name, even with a different type:

```rust
let input = "42";              // &str (a string slice)
let input: i32 = input.parse().unwrap();  // now an i32, same name
```

This is idiomatic Rust. It's commonly used for conversions like this, so you don't need names like `inputStr` and `inputInt`.

### 2. Types: more explicit than Java

| Java | Rust | Notes |
|---|---|---|
| `byte`, `short`, `int`, `long` | `i8`, `i16`, `i32`, `i64`, `i128` | Signed integers |
| (none) | `u8`, `u16`, `u32`, `u64`, `u128` | Unsigned types exist in Rust |
| (none) | `usize`, `isize` | Pointer-sized; used for indexing and lengths |
| `float`, `double` | `f32`, `f64` | |
| `boolean` | `bool` | |
| `char` (2 bytes, UTF-16) | `char` (4 bytes, any Unicode scalar value) | |

Watch out for these differences:

```rust
let a: i32 = 10;
let b: i64 = a;          // ERROR: Rust never converts implicitly, even when widening
let b: i64 = a as i64;   // explicit cast
let b: i64 = a.into();   // explicit, lossless conversion (preferred)
```

Java silently widens `int` to `long`. Rust refuses. It's annoying for the first week, but it prevents a whole category of bugs.

**Integer overflow** also behaves differently. In debug builds, overflow makes the program **panic** (crash with an error). In release builds, it wraps around. Java always wraps silently. If you actually want wrapping, say so with methods like `wrapping_add`, `checked_add`, or `saturating_add`.

**Compound types:**

```rust
let point: (i32, f64) = (3, 4.5);      // tuple; Java has no built-in equivalent
let (px, py) = point;                  // destructuring
let first = point.0;

let arr: [i32; 5] = [1, 2, 3, 4, 5];   // fixed-size array; the size is part of the type
let zeros = [0; 100];                  // 100 zeros
let v: Vec<i32> = vec![1, 2, 3];       // growable, like ArrayList<Integer>
```

Indexing past the end of an array (`arr[10]`) panics, just as Java throws `ArrayIndexOutOfBoundsException`.

### 3. Expressions vs statements (an important mental shift)

In Rust, almost everything is an **expression**, meaning it produces a value. Because `if` is an expression, Rust doesn't need a ternary operator:

```rust
let status = if score >= 50 { "pass" } else { "fail" };
```

A block evaluates to its last expression, as long as that expression has **no semicolon**:

```rust
let y = {
    let a = 3;
    a * 2        // no semicolon, so the block evaluates to 6
};
```

### 4. Functions

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b        // implicit return: last expression, no semicolon
}

fn early_exit(n: i32) -> i32 {
    if n < 0 {
        return 0;   // `return` is used for early exits
    }
    n * 2
}
```

Parameter and return types are always required, while types for local variables are usually inferred. There's also no method overloading in Rust. Each function needs its own name.

A classic beginner bug is adding a semicolon by accident. `a + b;` turns the expression into a statement, so the function returns `()` (Rust's "unit" type, which is roughly like `void`). The compiler will catch it and tell you exactly what went wrong.

### 5. Control flow

```rust
// Infinite loop, and it can return a value!
let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 { break counter * 2; }
};

// while
while counter > 0 { counter -= 1; }

// for loops iterate over anything iterable; there's no C-style for(;;)
for i in 0..5 { }          // 0,1,2,3,4 (exclusive)
for i in 0..=5 { }         // 0..5 inclusive
for x in arr.iter() { }    // like Java's for-each
for (i, x) in arr.iter().enumerate() { }  // index + value
```

### 6. Printing

```rust
let name = "Ferris";
println!("Hello, {name}!");         // inline variable
println!("{} + {} = {}", 2, 3, 5);  // positional
println!("{:?}", arr);              // debug format, for arrays, tuples, etc.
println!("{:.2}", 3.14159);         // 2 decimal places
```

The `!` means `println!` is a **macro**, not a function. For now, just treat it as a print statement. Macros come in Phase 6.

### 7. A preview of something you'll love: no `null`

Rust has no `null`, so `NullPointerException` can't happen. A value that might be absent uses `Option<T>` instead. It's similar to Java's `Optional<T>`, except the compiler forces you to handle the empty case.

```rust
fn find(arr: &[i32], target: i32) -> Option<usize> {
    // returns Some(index) or None
}
```

We'll cover this properly in Module 3. You'll use it lightly in the exercises below. The `&[i32]` syntax means "borrow a slice of integers," which is the subject of Module 2.

---

## Exercises

Do these in a single project (`cargo new module1_exercises`) as separate functions called from `main`:

1. **Temperature conversion.** Write `fn c_to_f(c: f64) -> f64` and `fn f_to_c(f: f64) -> f64`.
2. **Fibonacci.** Write `fn fib(n: u32) -> u64`, iteratively. Then find the largest `n` that fits in a `u64` by using `checked_add` to detect overflow.
3. **FizzBuzz using `match`.** Look up `match` on a tuple: `match (n % 3, n % 5) { (0, 0) => ..., ... }`.
4. **Linear search.** Write `fn linear_search(arr: &[i32], target: i32) -> Option<usize>`.
5. **Binary search.** Write `fn binary_search(arr: &[i32], target: i32) -> Option<usize>` on a sorted array. Be careful with `usize`: it can't go negative, so `mid - 1` when `mid == 0` will panic. Handling that edge case is a good Rust lesson in itself.
6. **Expression practice.** Rewrite one of your functions so it has no `return` keyword anywhere.

## Mini-Project: Number Guessing Game

Build a CLI game with these requirements:
- The program picks a random number from 1 to 100.
- The player keeps guessing. After each guess, print "Too high," "Too low," or "Correct!"
- If the input isn't a number, print a friendly message instead of crashing.
- Count the attempts and show the total at the end.
- **Stretch goal:** offer difficulty levels (1–50, 1–100, 1–1000) and ask whether to play again.

Here's the setup and the input-reading code. Some of it uses concepts from Modules 2 and 3, so treat it as a recipe for now:

```bash
cargo add rand
```

```rust
use std::io;

fn main() {
    let secret: u32 = rand::random_range(1..=100);   // rand 0.9 API

    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)          // &mut = lend it mutably (Module 2)
        .expect("Failed to read line");

    let guess: u32 = match input.trim().parse() {   // parse returns a Result (Module 3)
        Ok(num) => num,
        Err(_) => {
            println!("Please enter a number!");
            return;   // in your loop, use `continue` instead
        }
    };
}
```

If `cargo add` installs a version of `rand` older than 0.9, the API is `rand::thread_rng().gen_range(1..=100)` with `use rand::Rng;`. Check the version in your `Cargo.toml`.

**You're done when:** the game works, it handles bad input without crashing, `cargo clippy` shows no warnings, and `cargo fmt` has been run.G

---

