# Stringy
A tiny Rust crate for generating byte-sized enums that represent a fixed,
ordered set of `&str` data with optional associated data.

The original motivation for this crate came up while handwriting lexers (to
define reserved keywords, operators, fixed textual flags) and
parsers, as well as in representing bytecode operations in corresponding vms.

## Features
* No more boilerplate for associating enums with fixed string literals
* Encapsulate a set of string literals as their own type
* Associate one or more string slices to a single enum variant
* Associate with each enum variant some data of arbitrary type. Note: not all
  variants need to have data associated with them, but it is required that all
  associated data values must be *uniformly typed*. 
* Each generated enum has a size of only `1` byte.
* Each generated enum defines a total order on its variants (based on
  blanket implementation of derived `Ord` trait) and exposes an interface to
  iterate across all variants in this order.
* Generated data comes with modest documentation. In particular, enum
  variants and relevant associated methods include user-provided data as well as
  auto-generated documentation for ease of reference.

# Examples
```rust
use stringy::stringy;

// Let's define some names for colors
stringy! {
    /// An RGB color in multiple languages. This doc comment will show up
    /// in the enum definition for `Color`.
    Color =
        /// Documentation for `Red` variant will also be included when
        /// generating a `stringy`-generated enum.
        Red     "red"
        /// The variant `Green` matches either the slice `"green"`, or the
        /// alternative `"verde"`. 
        Green   "green"
        Blue    "blue"
}

// Now we can test any strings to see if they (fully) match 
// a color name. 
let not_a_color_name = "boop";
let red = "red";
assert_eq!(Color::test_str(not_a_color_name), false);
assert_eq!(Color::from_str(not_a_color_name), None);
assert_eq!(Color::test_str(red), true);
assert_eq!(Color::from_str(red), Some(Color::Red));
// Notice that *alternatives* are considered when matching 
// from `&str` to the enum. In other words, multiple string 
// slices may map to a single variant, BUT the only string 
// slice obtainable from a variant is the primary string 
// slice it was defined with (i.e., the non-alternative).
assert_eq!(Color::from_str("green"), Color::from_str("verde"));

// Each variant is associated with a `usize` value indicating 
// its order relative to the other variants
let idx_red = Color::Red.as_usize();
let idx_blue = Color::Blue.as_usize();
assert_eq!(idx_red, 0);
assert_eq!(idx_blue, 2);

// We can also generate a fixed-size array of all of the 
// possibiities, ordered as defined.
let rgb = Color::VARIANTS;
assert_eq!(rgb, [Color::Red, Color::Green, Color::Blue]);

// *UNSTABLE FEATURE*
// In fact, we can use the fixed-size array constant `VARIANTS` 
// to query a variant by *order of definition*, mimicking the 
// compiler-generated implementation(s) of `PartialOrd`.

// NOTE that the only caveat to this is that the provided index 
// fits within the bounds of the associated array constant 
// `VARIANTS` containing all defined variants.
assert_eq!(Color::VARIANTS[0], Color::Red);

// We can even define an enum with associated data values! 
// Suppose we wanted to describe infix operators along with 
// their associativities and precedence values. We could 
// first describe associativity and precedence as an enum 
// (named `Fixity` below), and then later associate instances 
// of this type with our operator enum!
// NOTE: the `stringy` macro does not have control over the
// associated data provided. Any "testing" or "checking"-related
// functionality must have the necessary traits derived or implemented.
#[derive(PartialEq, Eq, Debug)]
pub struct Fixity {
  Left(usize), 
  Right(usize), 
  None(usize)
}
// Simply add the *name* of the datapoint with the type, in 
// curly brackets after the enum name but before the `=` that 
// precedes variant definitions, and a method named `name` 
// will be generated, returning the provided data -- if it 
// exists -- as an optional type.
stringy! { Operator { fixity: Fixity } 
  = 
    Eq "==" { Fixity::None(4) }
    Add "+" { Fixity::Left(6) }
    Sub "-" { Fixity::Left(6) }
    Mul "*" { Fixity::Left(7) }
    Div "/" { Fixity::Left(7) } 
}

assert_eq!(Operator::Add.fixity(), Some(Fixity::Left(6)))
```