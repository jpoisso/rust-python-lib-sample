# Rust library usage in Python Sample

This project serves as an example of a library written in `Rust` that can be packaged and installed within a `Python` project.

## Goals

- [x] Write a trivial library in `Rust`.
- [x] Generate a `Python` wheel from the `Rust` library.
- [x] Install and use the library within a `Python` project.
- [ ] Investigate cross-compilation (Windows and Linux support).
- [ ] Investigate supporting multiple `Python` versions.
- [ ] Measure overhead costs and compare with `Python-native` solutions.

## Overview

We'll be using [pyo3](https://github.com/PyO3/pyo3) in order to provide `Rust` bindings for `Python` and [maturin](https://github.com/PyO3/maturin) to build and publish `Rust` binaries and `Python` packages.

## Setup

### Generate a library

We'll call library `rust-python-lib`.

```shell
cargo new rust-python-lib --lib
cd rust-python-lib
```

At this point, we have a basic `Rust` library skeleton.

### Modify Cargo.toml

We'll need additional configurations in [Cargo.toml](rust-python-lib/Cargo.toml) in order to properly build our solution.

The `lib.name` value is used to provide a `Python` compliant name for the library (certain characters are not allowed, such as dashes).

```toml
[package]
name = "rust-python-lib"
version = "0.1.0"
edition = "2018"

[lib]
name = "rust_python_lib"
crate-type = ["cdylib", "lib"]

[dependencies]
pyo3 = { version = "0.18.3", features = ["extension-module"] }

[build-dependencies]
maturin = "1.0.0"
```

### Implement the library logic

We'll keep the business logic short, for demonstration purposes.

`#[pymodule]` is used to expose functions that can be used from `Python`.

`#[pyfunction]` is used to define a function that can be exposed.

```rust
use pyo3::prelude::*;
use pyo3::wrap_pyfunction;

#[pymodule]
fn rust_python_lib(_: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(sum_as_string, m)?)?;
    Ok(())
}

#[pyfunction]
fn sum_as_string(a: usize, b: usize) -> PyResult<String> {
    Ok((a + b).to_string())
}
```

### Build the solution

#### Install dependencies
```shell
pip install maturin
```

#### Build the package
If you're using the same `venv` as your project, you can simply run `maturin develop` and it will build the wheel and install it within your `venv`.

If you want to generate a `Python` wheel:

```shell
cd rust-python-lib

maturin build -i python
```

The resulting wheel can be found in [rust-python-lib/target/wheels/](rust-python-lib/target/wheels).

#### Installing the package

The name of the wheel will vary depending on your `Operating System` and `Python` version.
```shell
pip install target/wheels/rust_python_lib-0.1.0-cp310-none-win_amd64.whl
```

#### Using the package

- Command line:

```shell
python -c "import rust_python_lib; print(rust_python_lib.sum_as_string(10, 20))"
```

- `Python` file: 
```python
import rust_python_lib
print(rust_python_lib.sum_as_string(10, 20))
```

Both should produce `30`, which is the sum of both numbers computed within our `Rust` library and returned to the `Python` context.