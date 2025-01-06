# Memory Management

- The notion of destructor is provided through the `Drop` trait. The destructor is called when the resource goes out of scope. This trait is not required to be implmenented for every type, only implement it for your type if you require its own destructor logic.
- Given vars are in charge of freeing their own resources, **resources can only have one owner**. This prevents resources from being freed more than once.  !! not all the variables own resources (eg. `references`). ==The transfer of ownership is knowns as a __move__.==
- Mutability of data can be changed when ownership is transferred.
- The compiler statically guarantees (via its borrow checker) that references _always_ point to valid objects, so, while the references to an object exists, the object cannot be dropped.
- Mutable data can be mutably borrowed using `&mut T` -> _mutable/exclusive reference_ and gives read/write access to the borrower. `&T` borrows the data via an `immutable/shared reference`, and the borrower can read it but can't modify it.
> Data can be immutably borrowed any number of times, but while immutably borrowed, the original data can't be mutably borrowed. On the other hand, only __one exclusive ref/mutable borrow__ is allowed at a time. The original data can be borrowed again only _after_ the mutable reference has been used for the last time.

- A `ref` borrow on the left hand side of an assignment = an `&` borrow on the right side:
```rust
let ref x = c; // let x = &c;
```
	- When using pattern matching or destructuring via the `let` binding, the `ref` keyword can be used to take reference to the ** fields ** of the struct/tuple rather then taking the ref of the whole thing.
```rust
{
        // `ref` can be paired with `mut` to take mutable references.
        let Point { x: _, y: ref mut mut_ref_to_y } = mutable_point;

        // Mutate the `y` field of `mutable_point` via a mutable reference.
        *mut_ref_to_y = 1;
    }
```

### Lifetimes
- It's a construct that the compiler (it's borrow checker) uses to ensure all borrows are valid. A variable's lifetime begins when it's created and ends when its destroyed.  The point is to track how long refs are valid and ensure that there are no dangling refs.
- When a lifetime is not constrained, it defaults to `'static`.
	- ##### Functions
		- Ignoring elision, function signatures with lifetimes have the following constraints:
			1. any ref _must_ have annotated lifetime.
			2. any ref being returned _must_ have the same lifetime as an input or be `'static`.
	- ##### Bounds
		- `T: 'a`: All references in T must outlive lifetime `'a`.  The type T must be valid for at least as long as `'a`, i.e., the type T must not contain any ref that's shorter then the lifetime `'a`:
	```rust
	/// T implements Debug and all *references* in `T` outlive `'a`. In addition
	/// `'a` must outlive the function.
	fn print_ref<'a, T>(t: &'a T)
	where
	    T: std::fmt::Debug + 'a,
	{
	    println!("`print_ref`: t is {:?}", t);
	}

```
- A longer lifetime can be _coerced_ into a shorter one so that it works inside a scope it normally wouldn't work in. This comes in the form of inferred coercion by the compiler, and also in form of declaring a lifetime difference:
```rust
// <'a: 'b, 'b> reads a lifetime 'a is atleast as long as 'b.
fn choose_first<'a: 'b, 'b>(first: &'a i32, _: &'b i32) -> &'b i32 {
    first
}
```
