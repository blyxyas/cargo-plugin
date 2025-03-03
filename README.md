# Cargo-genplugin

`cargo-genplugin` is a plugin management system and interface designed to easily let the users add functionality.

It works by generating interface code, that the main code can import and use without worrying about loading libraries or creating wrappers.

###### **Note: All functions must have the `#[no_mangle]` attribute**

## Simple Example

```rust
// my_plugin/src/lib.rs
#[no_mangle] // <- Important!
extern fn add(a: u32, b: u32) -> u32 {
	// Custom functionality goes here
	// [...]
	a + b
}
```

```toml
# my_plugin/Cargo.toml
# [...]
[lib]
crate-type = ["rlib", "cdylib"]
# [...]
```

Then run `cargo build -r`. This will create a file called `lib<PLUGIN NAME>.so` in your `target/release` directory.

Now it's the moment to use the program. (You'll have to [install it](#installation) first)

```
cargo plugin my_plugin libmy_plugin.so
```

This will generate a `stubs` cargo project, this project will contain generated code to load and use the plugin functions. In this example:

```rust
// stubs/src/lib.rs
// File autogenerated by cargo-genplugin, you can use the `--fmt` flag to generate formatted files. Do not manually edit, it will be overwritten. Report issues @ github.com/blyxyas/cargo-genplugin. The following functions were generated:

			/*
			* add
*/
use libloading;
			use	lazy_static;
				lazy_static::lazy_static! {
					static ref LIB: libloading::Library =
						unsafe { libloading::Library::new("/home/alejandra/git/wawawa/my_plugin/target/release/libmy_plugin.so").expect("Couldn't load library libmy_plugin.so") };
				}
			
pub unsafe extern fn add(a : u32 , b : u32)-> u32 {let func:libloading::Symbol<unsafe extern fn(a : u32 , b : u32) -> u32>=LIB.get(b"add").expect("Couldn't load function 'add'");return func(a,b,);}
```

###### **Tip: You can use the flag `--fmt` to auto-format it.**

now you can import and use the function add anywhere in your code!

```rust
// my_code/src/main.rs

#[cfg(feature = "custom_add")] // Feature gating might be useful
unsafe fn add(a: u32, b: u32) -> u32 {
	use stubs::add;
	unsafe { add(a, b) }
}

#[cfg(not(feature = "custom_add"))]
fn add(a: u32, b: u32) -> u32 {
	// My boring normal `add` function
	a + b
}

fn main() {
	unsafe {
		let x = add(1, 2); 
		println!("{x}");
	};
}
```

Take into account that the code will always unsafe. Also take into account that plugged-in code can potentially be malicious.

## Installation

Currently `cargo-genplugin` isn't uploaded to [crates.io](https://crates.io), so you'll have to manually compile it yourself.

### Compiling the binary

1. Get the repository

```
git clone https://github.com/blyxyas/cargo-genplugin.git
cd cargo-genplugin
```

2. Install the binary
   
```
cargo install --path .
```

3. **✨ It's ready to use! ✨**

```
cargo genplugin --help
```

#### Stargazers

[![Stargazers repo roster for @blyxyas/cargo-genplugin](https://reporoster.com/stars/blyxyas/cargo-genplugin)](https://github.com/blyxyas/cargo-genplugin/stargazers)

#### LICENSE

`cargo-genplugin` uses **Apache License 2.0**. [More information about licensing](https://github.com/blyxyas/cargo-genplugin/blob/main/LICENSE)