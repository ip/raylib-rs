![logo](logo/raylib-rust_256x256.png)

![rust](https://img.shields.io/badge/rust-1.31+-orange.svg?style=flat-square&logo=rust)
[![crates.io](https://img.shields.io/crates/v/raylib.svg?style=flat-square)](https://crates.io/crates/raylib)
[![docs](https://docs.rs/raylib/badge.svg)](https://docs.rs/raylib)

# raylib-rs

raylib-rs is a Rust binding for [raylib](http://www.raylib.com/) 2.0. It currently targets the _stable_ Rust toolchain, version 1.31 or higher.

Though this binding tries to stay close to the simple C API, it makes some changes to be more idiomatic for Rust.

- Resources are automatically cleaned up when they go out of scope (or when `std::mem::drop` is called), just like all other resources in Rust. This means that "Unload" functions are not exposed (and not necessary).
- Most of the Raylib API is exposed through `RaylibHandle`, which is for enforcing that Raylib is only initialized once, and for making sure the window is closed properly.
- A `RaylibHandle` is obtained through `raylib::init_window(...)` or through the newer `init()` function which will allow you to `build` up some window options before initialization (replaces `set_config_flags`).
- Manually closing the window is unnecessary, because `CloseWindow` is automatically called when `RaylibHandle` goes out of scope.
- `Model::set_material`, `Material::set_shader`, and `MaterialMap::set_texture` methods were added since one cannot set the fields directly. Also enforces correct ownership semantics.
- `Font::from_data`, `Font::set_chars`, and `Font::set_texture` methods were added to create a `Font` from loaded `CharInfo` data.
- `SubText` and `FormatText` are omitted, and are instead covered by Rust's string slicing and Rust's `format!` macro, respectively.

**Disclaimer: I created this binding as a way to learn Rust. There may be some things I can do better, or make more ergonomic for users. Feel free to make suggestions!**

# Installation

So far, I have only tested on Windows. Tips on making things work smoothly on all platforms is appreciated.

1. Add the dependency to your `Cargo.toml`:

```toml
[dependencies]
raylib = "0.9"
```

2. Download raylib 2.0 from https://github.com/raysan5/raylib/releases/tag/2.0.0, and pick the one that matches your Rust toolchain. MSVC with MSVC, MinGW with GNU, 32-bit or 64-bit.

3. Copy `libraylib.a` (for GCC/MinGW) or `raylib.lib` (for MSVC) to the appropriate path in your Rust toolchain.
   - For rustup/MSVC: `.rustup\toolchains\stable-x86_64-pc-windows-msvc\lib\rustlib\x86_64-pc-windows-msvc\lib`
   - For rustup/GNU: `.rustup\toolchains\stable-x86_64-pc-windows-gnu\lib\rustlib\x86_64-pc-windows-gnu\lib`

4. Start coding!

```rust
use raylib::*;

fn main() {
    let rl = raylib::init()
        .size(640, 480)
        .title("Hello, World")
        .build();
    
    while !rl.window_should_close() {
        rl.begin_drawing();
        
        rl.clear_background(Color::WHITE);
        rl.draw_text("Hello, world!", 12, 12, 20, Color::BLACK);

        rl.end_drawing();
    }
}
```

# Tech Notes

- Structs holding resources have RAII/move semantics, including: `Image`, `Texture2D`, `RenderTexture2D`, `Font`, `Mesh`, `Shader`, `Material`, `Model`, `Wave`, `Sound`, `Music`, and `AudioStream`.
- Functions dealing with string data take in `&str` and/or return an owned `String`, for the sake of safety.
- In C, `LoadFontData` returns a pointer to a heap-allocated array of `CharInfo` structs. In this Rust binding, said array is copied into an owned `Vec<CharInfo>`, the original data is freed, and the owned Vec is returned.
- In C, `GetDroppedFiles` returns a pointer to an array of strings owned by raylib. Again, for safety and also ease of use, this binding copies said array into a `Vec<String>` which is returned to the caller.
- I've tried to make linking automatic, though I've only tested on Windows. Other platforms may have other considerations.

# Extras

- In addition to the base library, there is also a convenient `ease` module which contains various interpolation/easing functions ported from raylib's `easings.h`, as well as a `Tween` struct to assist in using these functions.
- Equivalent math and vector operations, ported from `raymath.h`, are `impl`ed on the various Vector and Matrix types. Operator overloading is used for more intuitive design.

# Future Goals

- Port raylib examples over to Rust.
- More tests.
- More platform testing.
- Even more testing.
- Physac port?
