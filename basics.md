# Basics
----
- A slice is a two-word object: the first word is a pointer to the data, the second word is the length of the slice. The word size is the same as `usize` , determined by the processor arch, ex., 64 bits on an x64 arch.
- Slices can be used to _borrow a section of the array_ -> `&[T]`.
- Numbers can have arbitrary underscores added for readability: `5_i32` is same as `5i32`.
- Primitive types can be converted to each other through _casting_. For custom types, we use `From` and `Into` traits.
	- The `From` and `Into` traits are inherently linked: If you are able to convert type A from type B, then it should be easy to believe that we should be able to convert type B to type A.
#### From Trait
This trait allows for a type to define how to create _itself_ from another type, ex., `let foo: String = String::from("foo");`
#### Into Trait
This trait is just a the reciprocal of the `From` trait - it defines how to convert a type into another type.

>If you've implemented the `From` trait for your type, `Into` will call it when necessary. However, converse is not true: implementing `Into` for your type will not automatically provide it with an implementation of `From`. For fallible conversions, use `TryFrom` and `TryInto`.

- To convert any type to `String`, just implement `fmt::Display` - it automagically provides the `ToString` implementation.
- To get your own type from a `String`, implement the `FromStr` trait. A string can be converted to any type as long as the `FromStr` trait is implemented for that type.

#### For + Iterators
- By default, the `for` loop will apply the `into_iter` function to the collection. `into_iter`, `iter` and `iter_mut` all handle the conversion of a collection into an iterator, providing different views:
	- `iter` - borrows each element of the collection through each iteration. Thus, leaving the collection available for reuse.
	- `into_iter` - consumes the collection on each iteration, so, the collection is no longer available once it's consumed by the loop.
	- `iter_mut` - mutably borrows each element of the collection, allowing for the collection to be modified in place.
- When you are matching a bunch of array values: you can bind the first and last value, and store th rest of them in a single array/slice:
```rust
let arr = [1, 2, 3, 4];
match arr {
	[first, middle @ .., last] => println!("{}, {}, {}", first, middle, last)
}
```

#### pointers/ref
- Dereferencing uses `*`.
- Destructuring uses `&, ref, ref mut`.
```rust
fn main() {
    // Assign a reference of type `i32`. The `&` signifies there
    // is a reference being assigned.
    let reference = &4;

    match reference {
        // If `reference` is pattern matched against `&val`, it results
        // in a comparison like:
        // `&i32`
        // `&val`
        // ^ We see that if the matching `&`s are dropped, then the `i32`
        // should be assigned to `val`.
        &val => println!("Got a value via destructuring: {:?}", val),
    }

    // To avoid the `&`, you dereference before matching.
    match *reference {
        val => println!("Got a value via dereferencing: {:?}", val),
    }

    // What if you don't start with a reference? `reference` was a `&`
    // because the right side was already a reference. This is not
    // a reference because the right side is not one.
    let _not_a_reference = 3;

    // Rust provides `ref` for exactly this purpose. It modifies the
    // assignment so that a reference is created for the element; this
    // reference is assigned.
    let ref _is_a_reference = 3;

    // Accordingly, by defining 2 values without references, references
    // can be retrieved via `ref` and `ref mut`.
    let value = 5;
    let mut mut_value = 6;

    // Use `ref` keyword to create a reference.
    match value {
        ref r => println!("Got a reference to a value: {:?}", r),
    }

    // Use `ref mut` similarly.
    match mut_value {
        ref mut m => {
            // Got a reference. Gotta dereference it before we can
            // add anything to it.
            *m += 10;
            println!("We added 10. `mut_value`: {:?}", m);
        },
    }
}
```

- Port 0 is special-cased at OS level: trying to bind port 0 will trigger an OS scan for an available port which will then be bound to the app.
- Table driven tests or parametrised tests are really good when dealing with bad inputs -- instead of duplicating the test logic several times we can simple run the same assertion against a collection of known invalid bodies that we expect to fail in the same way. Make sure to have good error messages on failures.
- `cargo tree -d` cmd is helpful for finding all the places that duplicate deps are being pulled in.
- Whenever possible, you should strive to use the same db both for your tests and your prod env.
- If you have some API and you want to enforce a guarantee that only one _user_ can use it — accept a mutable reference, because there cannot be two active mutable references to the same value at the same time in the whole program! This is what `sqlx` does — it has an asynchronous interface, but it doesn’t allow you to run multiple queries concurrently over the same db connection. Requiring an exclusive reference allows them to enfore this guarantee in their API.
- To parse a String, the idiomatic approach to this is to use the parse function and either to arrange for type inference or to specify the type to parse using the 'turbofish' syntax. This will convert the string into the type specified as long as the `FromStr` trait is implemented for that type. This is implemented for numerous types within the standard library. To obtain this functionality on a user defined type simply implement the `FromStr` trait for that type.

- Want to make sure that something just runs _once_ — use the `std::sync::Once` or the “[once_cell](https://docs.rs/once_cell/1.16.0/once_cell/)” trait.

- `rustc` statically links all Rust code but dynamically links `libc` from the underlying system if you are using Rust std lib. You can get a fully statically linked binary by targeting `linux-musl`.

- When you’ve created a domain type to encode and parse the info. for your domain, and you want to use to use the _internal_ type (for instance, ref of the internal `String` from `SusbcriberName(String)`) — best way to do it, would be to impl `AsRef<T>` trait. Generally, you should impl this trait for a type when the type is _similar enough_ to `T` that we can use a `&self` to get a reference to `T` itself!

- When writing tests for functions that return `Result`, better to use the “[claim](https://docs.rs/claim/%5E0.5.0)” trait — provides better error msgs.

- For email validations, use the “[validator](https://crates.io/crates/validator)” crate.

- To mock and test a HTTP server impl, you can the “[wiremock](https://github.com/lukemathwalker/wiremock-rs)” crate.

    - `wiremock::MockServer` is a full-blown HTTP server. `MockServer::start` asks the OS for a random available port and spins up the server on a background thread, ready to listen for incoming requests.

    - To get a list of all the requests intercepted by our mock server, use `received_requests` method. It will work as long as request recording is enabled (default case).

    - To point your client to your mock server, just retrieve the address of the mock server using the `MockServer::uri` method and use it as the “base_url” for your client.

    - By default, `wiremock::MockServer` returns a “404 Not Found” to all incoming requests. To make it behave differently, mount a `Mock`:

        ```rust
        // `any` matches all incoming requests, regardless of their method, path, headers
        // or body. Use it when you just want to check if a request if fired to a server.
        Mock::given(any())
        	.respond_with(ResponseTemplate::new(200))
        	// required to make this Mock effective
        	.mount(&mock_server)
        	// sets an _expectation_ on our mock: the mock server, during this test,
        	// should receive _exactly_ one request that matches the condition set by this mock.
        	// you can also use range for expectations: expect(1..=3) -> 1 to 3.
        	// Expectations are verified when the `MockServer` goes out of scope.
        	.expect(1)
        	.await;
        ```

    - With `mount` , the behaviour we specify remains active as long as the underlying `MockServer` is up and running. With `mount_as_scoped` , we get back a guard object — a `MockGuard`

        - `MockGuard` has a custom `Drop` impl: when it goes out of scope, _wiremock_ instructs the underlying `MockServer` to stop honouring the specified mock behaviour, because the behaviour is scoped to that block of code. The mock behaviour stays local to the test helper.
    - When `wiremock::MockServer` receives a request, it iterates over all the mounted mocks to check if the request _maches_ their conditions. The matching conditions for a mock are specified using `Mock::given`.

        - wiremock exposes a `Match` trait — everything that impls it can be used as a matcher in `given` and `and`.

            ```rust
            		struct SendBodyMatcher;

                impl wiremock::Match for SendBodyMatcher {
                    fn matches(&self, request: &Request) -> bool {
                        let result: Result<serde_json::Value, _> = serde_json::from_slice(&request.body);
                        if let Ok(body) = result {
                            body.get("From").is_some()
                                && body.get("To").is_some()
                                && body.get("Subject").is_some()
                                && body.get("HtmlBody").is_some()
                                && body.get("TextBody").is_some()
                        } else {
                            false
                        }
                    }
                }
            ```

- Whenever you are performing I/O ops, always set a _timeout_. Choosing the right timeout value is often more an art than a science, especially if retries are involved: set it too low and you might overwhelm the server with retried requests; set it too high and you risk again to see degradation on the client side.

- For tokens, which do not grant access to protected data, you can use _cryptographically secure pseudo-random number generator - a CSPRNG_ : The “rand” crate (and the “std-rng” feature) can help you in generating these.
- The Rust “orphan rule” — it is forbidden to implement a foriegn trait for a foriegn type, where foriegn stands for “from another crate”. This restriction is meant to preserve coherence: image if you added a dependency that defined its own impl of the trait for the same type — which one should compiler choose, when the trait methods are invoked? That’s why, the orphan rule.
- “Trait Objects” like generic type params — are a way to achieve polymorphism in Rust: invoke different impls of the same trait. Generic types are resolved at compile time — static dispatch, trait objects are resolved at runtime — dynamic dispatch.
    - We’ve to wrap a `dyn std::error::Error` in a `Box` because the size of trait objects is not knows at compile-time: trait objects can be used to store different types (as long as all of them impl the same trait) which will most likely have different layouts in memory — they are _unsized_ — they do not impl the `Sized` _marker trait_. A `Box` stores the trait object itself on the heap, while we store the pointer to its heap location on Stack, because the pointer itself has a known size at compile time.
- Errors should be logged at the same place, where they are being handled. If you function is propagating the error upstream (maybe by using the `?` op), it should **not** log the error. If it can, if it makes sense, add more _context_ to it. Whichever fn handles the error, they should take care of logging it. If the error is propagated all the way up to the request handler, delegate logging to a dedicated middleware, such as `tracing_actix_web::TracingLogger`.
- When you spend time working on a problem, you end up deepening your understanding of its “domain”. You often acquire a more precise _language_ that can be used to refine earlier attempts of desribing the desired functionality.
- Generally, if you are implementing a _redirect_ for a webpage, the redirect response requires two elements:
    1. a redirect status code (in 3xx range).
    2. a `Location` header, set to the URL we want to redirect to.
- Use `{:#?}` for pretty printing debug impl of types.
