# Stringy
A tiny Rust crate for generating byte-sized enums that represent a fixed,
ordered set of `&str` data.

The original motivation for this crate came up while handwriting lexers and
parsers.

## Features
* No more boilerplate for associating enums with fixed string literals
* Encapsulate a set of string literals as their own type
* Each generated enum has a size of only `1` byte
* Each generated enum defines a total order on its variants (based on
  blanket implementation of derived `Ord` trait) and exposes an interface to
  iterate across all variants in this order.
* Generated data comes with modest documentation. In particular, enum
  variants and relevant associated methods include user-provided data