### Hello World
- Functions are introduced with *fn*
- Rust strings are UTF-8 encoded and can contain any Unicode character
- *main* function is the entry point for the program
### Values
![[Pasted image 20251101222839.png]]
- `usize` and `isize` are pointer-width because they are designed to hold memory addresses, array indices, and sizes which are directly tied to how a computer architecture addresses memory
### Arithmetic 
```rust
fn multiply(a: i8, b: i8, c: i8) -> i8 {
    a * b + b * c + c * a
}

fn main() {
    println!("result: {}", multiply(10, 10, 10));
}
```
```bash
thread 'main' (33) panicked at src/main.rs:2:5:
attempt to add with overflow
```
- In C/C++ overflow of signed integers is actually undefined and might do unknown things at runtime. In Rust, its defined.
- In Rust, we get a panic in debug build and wraps in release build.
#### Overflows
- If we are adding two `u8` numbers and the result is 256 (`u8::MAX+1`), Rust could chose to interpret the result as `u16`. But this does not happen as Rust is quiet picky about type conversions. Automatic integer promotion is not Rust's solution to the integer overflow problem.
- When the result of an arithmetic operation is bigger than the maximum value for a given integer type, you can choose to wrap around.
##### overflow-checks
- Is a profile setting
- If `overflow-checks` is set to `true`, Rust will **panic at runtime** when an integer overflow occurs
- If `overflow-checks` is set to `false`, Rust will **wrap around** when an integer operation overflows.
- Default value is `true` for dev profile, `false` for release profile
- Recommendation is to `panic` for both profiles. 
#### Profiles
- `dev`: Selected every time you run `cargo build`, `cargo run` or `cargo test`. Aimed at local development and sacrifices runtime performance in favor of faster compilation times and a better debugging experience.
- `release`: Optimized for runtime performance but incurs longer compilation times. `cargo build --release`. 
- `test`: used when you run `cargo test`. The `test` profile inherits settings from the `dev` profile. 
- `bench`: used by `cargo bench`. The `bench` profile inherits from the `release` profile. 
- Use `dev` for iterative development and debugging, `release` for optimized production builds, `test` for correctness testing and `bench` for performance benchmarking.
#### Conversions
##### as operator
```rust
let a: u32 = 10;

// Cast `a` into the `u64` type
let b = a as u64;

// You can use `_` as the target type if it can be correctly 
// inferred by the compiler.
let c: u64 = a as _;
```
##### Truncation
- Things get more interesting if we go in the opposite direction
```rust
let a: u16 = 255 + 1;
let b = a as u8;
```
- When converting from `u16` to `u8`, Rust compiler will keep the last 8 bits of the `u16` memory representation.
- Rust compiler will actively stop you if it sees you trying to cast a literal value which will result in a truncation.
### Type Inference

### Blocks and Scopes
- A block in Rust contains a sequence of expressions, enclosed by braces `{}`.
- The final expression of block determines the value and type of the whole block.
- If the last value ends with `;`, then the resulting value and type is `()`.
- A variable's scope is limited to enclosing block.
```rust
fn main() {
    let z = 13;
    let x = {
        let y = 10;
        dbg!(y);
        z - y
    };
    dbg!(x);
    // y is not in scope
    // dbg!(y);
}
```
#### if expressions
- You can use if as an expression
- The last expression of each block becomes the value of the `if` expression
```rust
fn main() {
    let x = 10;
    let size = if x < 20 {"small"} else {"large"};
    println!("number size: {}", size);
}
```
#### match expressions
```rust
fn main() {
    let val = 101;
    match val {
        1 => println!("one"),
        10 => println!("ten"),
        100 => println!("hundred"),
        _ => {
            println!("something else");
        }
    }
}
```
- Evaluated from top to bottom and the first one that matches has its corresponding body executed
- There is no fall-through between cases that switch works in other languages
- Needs to cover all cases or have a `_` for default case
### Loops
#### while
```rust
fn main() {
	let mut x = 200;
	while x >= 10 {
		x = x / 2;
	}
	dbg!(x);
}
```
#### for loop
```rust
fn main() {
	for x in 1..5 {
	    dbg!(x);
	}
	for elem in [2, 4, 8, 16, 32] {
	    dbg!(elem);
	}
	for x in 1..=5 {
	    dbg!(x);
	}
}
```
- Note that the first `for` loop only iterates to `4`. Show the `1..=5` syntax for an inclusive range.
#### loop statement
```rust
fn main() {
    let mut i = 0;
    loop {
        i += 1;
        dbg!(i);
        if i > 100 {
            break;
        }
    }
}
```
- The `loop` statement works like a `while true` loop. Use it for things like servers that will serve connections forever.
#### break and continue
- Similar to other programming languages to be used with loops
#### Labels
- Both `continue` and `break` can optionally take a label argument that is used to break out of nested loops:
```rust
fn main() {
    let s = [[5, 6, 7], [8, 9, 10], [21, 15, 32]];
    let mut elements_searched = 0;
    let target = 0;
    'outer: for i in 0..=2 {
        for j in 0..=2 {
            elements_searched += 1;
            if s[i][j] == target {
                break 'outer;
            }
        }
    }
    dbg!(elements_searched);
}
```
#### Functions
- The last expression in a function body (or any block) becomes the return value. Simply omit the `;` at the end of the expression.
- Some functions have no return value, and return the ‘unit type’, `()`. The compiler will infer this if the return type is omitted.
- Overloading is not supported – each function has a single implementation.
- Default arguments are not supported.
- *Always takes a single set of parameter types.* 
- Like variables, function arguments are immutable by default and you must add `mut` if you want to modify their value. This does not affect how the function is called or how the argument is passed in.
#### Macros
- Macros are expanded into Rust code during compilation, and can take a variable number of arguments.
- They are distinguished by a `!` at the end.
- Examples: `println!()`, `format!()`, `dbg!()`, `todo!()`, `assert!()`
### Tuples and Arrays
#### Arrays
```rust
fn main() {
    let mut arr: [i8; 5] = [1, 2, 3, 4, 5];
    arr[3] = 0;
    println!("{:#?}", arr);
}
```
```rust
fn main() {
    let arr: [u8; 1024] = [0; 1024];
    println!("{:#?}", arr);
}
```
- Arrays can also be initialized using the shorthand syntax, e.g. `[0; 1024]` - useful for large arrays
- A value of the array type `[T; N]` holds `N` (a compile-time constant) elements of the same type `T`. **Note that the length of the array is _part of its type_, which means that `[u8; 3]` and `[u8; 4]` are considered two different types.** 
- Accessing an out-of-bounds array element: The compiler is able to determine that the index is unsafe and will not compile the code.
- Arrays are not heap-allocated. They are regular values with a fixed size known at compile time, meaning they go on the stack.
- There is no way to remove elements from an array, nor add elements to an array. The length of an array is fixed at compile-time, and so its length cannot change at runtime.
- Array accesses are checked at runtime. Rust can usually optimize these checks away; meaning if the compiler can prove the access is safe, it removes the runtime check for better performance.
#### Tuples
```rust
fn main() {
    let t: (i8, bool) = (7, true);
    dbg!(t.0);
    dbg!(t.1);
}
```
- Like arrays, tuples have a fixed length.
- Tuples group together values of different types into a compound type.
- Fields of a tuple can be accessed by the period and the index of the value, e.g. `t.0`, `t.1`.
- The empty tuple `()` is referred to as the “unit type” and signifies absence of a return value, akin to `void` in other languages.
- Unlike arrays, tuples cannot be used in a `for` loop. This is because a `for` loop requires all the elements to have the same type, which may not be the case for a tuple.
- There is no way to add or remove elements from a tuple. The number of elements and their types are fixed at compile time and cannot be changed at runtime.
#### Array Iteration
```rust
fn main() {
    let primes = [2, 3, 4, 5, 7, 11, 13, 17, 19];
    for prime in primes {
        for i in 2..prime {
            assert_ne!(prime % i, 0);
        }
    }
}
```
- This functionality uses the `IntoIterator` trait
#### Patterns and Destructuring
```rust
fn check_order(tuple: (u32, u32, u32)) -> bool {
	let (left, middle, right) = tuple;
	left < middle && middle < right
}

fn main() {
	let tuple = (1, 5, 3);
	println!("{:#?}", if check_order(tuple) { "ordered" } else {"unordered"});
}
```
- The patterns used here are “irrefutable”, meaning that the compiler can statically verify that the value on the right of `=` has the same structure as the pattern.
### References
#### Shared References
```rust
// This works because of Non-Lexical Lifetimes (NLL)
// 
fn main() {
    let a = 'A';
    let b = 'B';

    let mut r: &char = &a;
    dbg!(r);
    println!("{:#?}", a);
    a = 'C';
    println!("{:#?}", a);
    // The compiler is smart enough to see that the borrow of `a` (held by `r`) was last used on the line `dbg!(r);`. After that line, `r` is not read again before it's reassigned, so the compiler ends the shared borrow on `a`.

    r = &b;
    dbg!(r);
    println!("{:#?}", b);
}

// This does not work
fn main() {
    let mut a = 'A';
    let b = 'B';

    let mut r: &char = &a;
    dbg!(r);
    println!("{:#?}", a);
    // `a` is assigned to here but it was already borrowed
    a = 'C';
    println!("{:#?}", a);
    dbg!(r);
    // borrow later used here

    r = &b;
    dbg!(r);
    println!("{:#?}", b);
}

// The below does not work too. 'r' is a & reference so the data it refers to cannot be rewritten.
fn main() {
    let mut a = 'A';
    let b = 'B';

    let mut r: &char = &a;
    dbg!(r);
    println!("{:#?}", a);
    *r = 'C';

    r = &b;
    dbg!(r);
    println!("{:#?}", b);

}
```
- A reference provides a way to access another value without taking ownership of the value, and is also called “borrowing”
- Shared references are read-only, and the referenced data cannot change.
- References are implemented as pointers, and a key advantage is that they can be much smaller than the thing they point to.
- Explicit referencing with `&` is usually required. *However, Rust performs automatic referencing and dereferencing when invoking methods.*
- A shared reference does not allow modifying the value it refers to, even if that value was mutable. Try `*r = 'C'`.
#### Exclusive References
```rust
// cannot borrow `point` as immutable because it is also borrowed as mutable
fn main() {
    let mut point = (1, 2);
    let x_coord = &mut point.0; // mutable borrow occurs here
    let y_coord = &point; // immutable borrow occurs here
    *x_coord = 20; // mutable borrow later used here
    println!("point: {point:?}");
}

// Works
fn main() {
    let mut point = (1, 2);
    let x_coord = &mut point.0;
    *x_coord = 20;
    let y_coord = &point;
    println!("point: {point:?}");
}
```
- “Exclusive” means that only this reference can be used to access the value. No other references (shared or exclusive) can exist at the same time, and the referenced value cannot be accessed while the exclusive reference exists.
#### Slices
```rust
fn main() {
    let a: [i32; 6] = [10, 20, 30, 40, 50, 60];
    println!("a: {a:?}");

    let s: &[i32] = &a[2..4];
    println!("s: {s:?}");
}
```
- We create a slice by borrowing `a` and specifying the starting and ending indexes in brackets.
- To easily create a slice of the full array, we can therefore use `&a[..]`.
- Slices always borrow from another object. In this example, `a` has to remain ‘alive’ (in scope) for at least as long as our slice.
- You can’t append elements of the slice, since it doesn’t own the *backing buffer.*
- To get a larger slice you have to back to the original buffer and create a larger slice from there.
#### Strings
- `&str` is a slice of UTF-8 encoded bytes, similar to `&[u8]`
```rust
// I thought the below would work but it does not.
// expected `&[u8]` found `&str`
fn main() {
    let s1: &[u8] = "World";
    println!("{:#?}", s1);
}
```
- `&str` is a slice of bytes that is guaranteed by the compiler to be valid UTF-8.
- A `&[u8]` on the other hand is just a slice of raw bytes. It could be anything: UTF-8, image data, or just random binary noise. Because of this the two data types are not interchangeable.
- `&str` to `&[u8]` is safe and easy. This is always safe because a valid UTF-8 string is, by definition a valid sequence of bytes. You can use `as_bytes()` method.
- `&[u8]` to `&str` : This conversion can fail because the byte slice might not be valid UTF-8. You must explicitly check it, which is why Rust provides `std::str::from_utf8().`

### Implementation and Methods

- Same name as a struct
```
struct Deck {
	...
}

impl Deck {
	...
}
```

- Used to define methods and associated functions (i.e. class methods)
```
impl Deck {
	fn new() {
		...
	}
	
	// Instance methods need to have &self as the first parameter
	fn shuffle(&self) {
		...
	}
}
```

- Associated function tied to the struct definition
- Methods operate on a specific instance of a struct

### Ownership, Borrowing and Lifetimes

12 rules that you need to remember

![[Pasted image 20251101124628.png]]
#### Updates

- The goal of ownership is to limit the way you can reference and change data
- This limitation will reduce the number of bugs + make your code easier to understand

![[Pasted image 20251101130152.png]]
![[Pasted image 20251101130207.png]]
![[Pasted image 20251101130223.png]]
![[Pasted image 20251101130237.png]]

- Changing engine for mustang changed engine condition for camero too because they were pointing to the same engine object.

**Fixes**
- Multiple things can refer to a value at the same time, but the references ensures the value is read-only
- A value can be updated when there are no read-only references to it
- Above all, Rust wants to avoid unexpected updates
#### Own
- Every value is ***owned*** by a ***single variable*** at a time
- Reassigning the value to another variable ***moves*** the value. The old variable can't be used to access the value anymore.

```rust

// Simplified version
fn main() {

	// bank is binded to a value in the heap memory
	let bank = Bank::new(); 
	
	// We reassigned the value to another variable
	let other_bank = bank;

	// will cause compile time error because the value has been moved
	// to other_bank variable so bank is empty.
	println!("{:#?}", bank);
}
```

The above rules were simplified to demonstrate a point. In practical terms, the rules are as follows:
- Every value is ***owned*** by a ***single*** variable, argument, struct, vector, etc. at a time
- Reassigning the value to a variable, passing it to a function, putting it into vector, etc ***moves*** the value. The older value can't be used to access the value anymore
#### Borrowing
- You can create many read-only (immutable) references to a value. These refs can all exist at the same time.
- You can't move a value while a ref to the value exists
#### Mutable References
- Allow us to read a or change a value without moving it
	- I want to make a value
	- Then allow that value to be changed somewhere else
	- Moving the value around manually would be tedious
	- Good solution is to use a mutable reference
- You can make a writable (mutable) reference to a value only if there are no read-only references currently in use. One mutable ref to a value can exist at a time.
- You can't mutate a value through the owner when any ref (mutable or immutable) to the value exists

### Trait, Struct and For Keyword

- **Struct**: Custom data type that allows you to group related data togethe
- **Trait**: Defines a set of methods that a type must provide. It's like a contract or an interface. Trait defines a behavior but it doesn't provide the data or the implementation itself.

```rust
trait Summarize {
	// Any type that implements Summarize MUST provide this method.
	fn summary(&self) -> String;
}

// Create a struct for a News Article
struct NewsArticle {
	headline: String,
	content: String,
}

// Create another struct for a Tweet
struct Tweet { 
	username: String,
	content: String, 
}

// Implement the Trait FOR the Struct
impl Summarize for NewsArticle {
	fn summary(&self) -> String {
		format!("{}, ...", &self.headline)
	}
}

// Implement the SAME Trait for the OTHER Struct
impl Summarize for Tweet {
	fn summary(&self) -> String {
		format!("@{} tweeted: {}", self.username, self.content)
	}
}
```
### Copyable values
- Some types of values are copied instead of moved (numbers, bools, chars, array/tuples with copyable elements.)


