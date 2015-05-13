# The Optimized Rust Programming Language

Added optimization level **4** (`-C opt-level=4`), which disables the bounds checks for arrays elements access. So, some code with a lot of arrays elements access by arbitrary indexes in cycles - runs faster.

# Further work:

I'm considering to remove this new "option" and just fix the broken rust logic later: implement the strict requirement for **access of non-overloaded array elements by index** to be placed **inside "unsafe" blocks**. As this is logically the same operation as dereferencing an arbitrary pointer. So it is inconsistent to force "access by dereferencing the pointer" to be strictly "unsafe", but access to the same memory by arbitrary array index to be "safe" (trying to "fix" it by resource-hungry repeated run-time bounds checks, steeling the resources from something more useful).
After the array access by non-overloaded index will be considered "unsafe" the run-time bounds checks will not be needed at all and will be removed completely. So, this optimization level **4** will be removed too.

# Just to be clear:

I'm doing this **for my own use**, as I like this way more, and I don't like to spend any runtime resources on a lot of unnecessary checks. So, I don't intend to convince anybody in this approach or argue with anybody. I'm just going **to use** this version of language **on my own** for my **resource critical** projects, where I have better things to do for the proccessor than sitting in repeating unnecessary run-time bounds checks during a lot of huge cycles with random data accesses in them.
But if you need these "`unsafe_ref()`" or "`unsafe_mut_ref()`" to eliminate these checks inside "unsafe" blocks - you just totally lose the convenient `[]` indexing syntax, as you will never use it throughout the whole code in time critical software - you will just always go into "unsafe" block and use e.g. "`some_vector.unsafe_mut_ref(idx)`" instead of the same logically unsafe but more readable "`some_vector[idx]`". But if you are trying to migrate from C or C++ to rust - you almost always intend to write the "time critical software", do you?..

# Just a little test:

```
#![feature(std_misc)]
#![feature(collections)]
use std::time::duration::Duration;

fn bound_check_test() -> Vec<u32> {
    let mut cur_vec = vec![];
    cur_vec.resize(100000, 1u32);

    for i in 0..100000 {
        for k in 0..i+1 {
            cur_vec[i] += cur_vec[k];
        }
    }
    return cur_vec;
}

fn main() {
    let mut tot: i64 = 0;
    for i in 0..10 {
        println!("Test iteration: {}", i+1);
        tot += Duration::span(|| {let a = bound_check_test()[0]; if a > 0 {return;}}).num_milliseconds();
    }
    tot /= 10;

    println!("TEST RESULT (one iteration): {} ms", tot);
}
```

On my i7 (32GB memory) machine, compiled with:
```
rustc -C opt-level=4 test.rs
```
gives one iteration time about **2700 ms**.
Compiled with:
```
rustc -C opt-level=3 test.rs
```
gives about **9900 ms**!

So, all these "*the bounds-checked code gives almost invisible drop in speed*" - is a total bullshit.


## Quick Start

Read ["Installing Rust"] from [The Book].

["Installing Rust"]: http://doc.rust-lang.org/book/installing-rust.html
[The Book]: http://doc.rust-lang.org/book/index.html

## Building from Source

1. Make sure you have installed the dependencies:

   * `g++` 4.7 or `clang++` 3.x
   * `python` 2.6 or later (but not 3.x)
   * GNU `make` 3.81 or later
   * `curl`
   * `git`

2. Clone the [source] with `git`:

   ```sh
   $ git clone https://github.com/rust-lang/rust.git
   $ cd rust
   ```

[source]: https://github.com/JimStar/rust

3. Build and install:

    ```sh
    $ ./configure
    $ make && make install
    ```

    > ***Note:*** You may need to use `sudo make install` if you do not
    > normally have permission to modify the destination directory. The
    > install locations can be adjusted by passing a `--prefix` argument
    > to `configure`. Various other options are also supported – pass
    > `--help` for more information on them.

    When complete, `make install` will place several programs into
    `/usr/local/bin`: `rustc`, the Rust compiler, and `rustdoc`, the
    API-documentation tool. This install does not include [Cargo],
    Rust's package manager, which you may also want to build.

[Cargo]: https://github.com/rust-lang/cargo

### Building on Windows

[MSYS2](http://msys2.github.io/) can be used to easily build Rust on Windows:

1. Grab the latest MSYS2 installer and go through the installer.

2. From the MSYS2 terminal, install the `mingw64` toolchain and other required
   tools.

   ```sh
   # Choose one based on platform:
   $ pacman -S mingw-w64-i686-toolchain
   $ pacman -S mingw-w64-x86_64-toolchain

   $ pacman -S base-devel
   ```

3. Run `mingw32_shell.bat` or `mingw64_shell.bat` from wherever you installed
   MYSY2 (i.e. `C:\msys`), depending on whether you want 32-bit or 64-bit Rust.

4. Navigate to Rust's source code, configure and build it:

   ```sh
   $ ./configure
   $ make && make install
   ```

## Notes

Since the Rust compiler is written in Rust, it must be built by a
precompiled "snapshot" version of itself (made in an earlier state of
development). As such, source builds require a connection to the Internet, to
fetch snapshots, and an OS that can execute the available snapshot binaries.

Snapshot binaries are currently built and tested on several platforms:

| Platform \ Architecture        | x86 | x86_64 |
|--------------------------------|-----|--------|
| Windows (7, 8, Server 2008 R2) | ✓   | ✓      |
| Linux (2.6.18 or later)        | ✓   | ✓      |
| OSX (10.7 Lion or later)       | ✓   | ✓      |

You may find that other platforms work, but these are our officially
supported build environments that are most likely to work.

Rust currently needs about 1.5 GiB of RAM to build without swapping; if it hits
swap, it will take a very long time to build.

There is more advice about hacking on Rust in [CONTRIBUTING.md].

[CONTRIBUTING.md]: https://github.com/rust-lang/rust/blob/master/CONTRIBUTING.md

## Getting Help

The Rust community congregates in a few places:

* [Stack Overflow] - Direct questions about using the language.
* [users.rust-lang.org] - General discussion and broader questions.
* [/r/rust] - News and general discussion.

[Stack Overflow]: http://stackoverflow.com/questions/tagged/rust
[/r/rust]: http://reddit.com/r/rust
[users.rust-lang.org]: http://users.rust-lang.org/

## Contributing

To contribute to Rust, please see [CONTRIBUTING](CONTRIBUTING.md).

Rust has an [IRC] culture and most real-time collaboration happens in a
variety of channels on Mozilla's IRC network, irc.mozilla.org. The
most popular channel is [#rust], a venue for general discussion about
Rust, and a good place to ask for help.

[IRC]: https://en.wikipedia.org/wiki/Internet_Relay_Chat
[#rust]: irc://irc.mozilla.org/rust

## License

Rust is primarily distributed under the terms of both the MIT license
and the Apache License (Version 2.0), with portions covered by various
BSD-like licenses.

See [LICENSE-APACHE](LICENSE-APACHE), [LICENSE-MIT](LICENSE-MIT), and [COPYRIGHT](COPYRIGHT) for details.
