# Elixir Basics

Elixir's built-in types are:
  - Value types - they represent numbers, ranges, names and regular expressions:
    - Arbitrary-sized integers
    - Floating-point numbers
    - Atoms
    - Ranges
    - Regular expressions

  - System types:
    - PIDs & ports
    - References

  - Collection types:
    - Tuples
    - Lists
    - Maps
    - Binaries

> Functions are a type too.

## Value Types

### Integers

Integer literals can be written as `decimal(1234)`, `hexadecimal(0xcafe)`, `octal(0o765)` and `binary(0b1010)`.

Decimal numbers may contain underscores - oftenly used to separate groups of three digits when writing large numbers. For example `1_000_000`.

### Floating-point numbers

Floating-point numbers are written using a decimal point and there must be at least one digit before and after the decimal point. An optional trailing exponent may be given. 

>Examples: `1.0`, `0.2456`, `0.314e1`, `34.0e-5`

Floats are **IEEE 754** double precision giving them about **16** digits of accuracy
  and **a maximum exponent of around 10^308**

### Atoms
Atoms are constants that represent something's name, they are written using a **leading colon(:)** which can be followed by an atom word or an Elixir operator.

An atom word is a sequence of letters, digits, underscores, and `@` signs and may end with an exclamation point or a question mark.

You can also create atoms containing arbitrary characters by enclosing in double quotes the characters following the colon.

Examples of Atoms: `:hello`, `:is_binary?`, `:var@2`, `:<>`, `:===`, `:"func/3"`, `:"long john"`

> An atom's name is it's value.

### Ranges
Ranges are represented as `start..end` where `start` and `end` can be values of any type. If you want to iterate over a range of values, the two extremes must be integers.

### Regular expressions
Elixir has regular expression literals written as `~r{regexp}` or `~r{regexp}opts`.

Here, the delimiters for regular expression literals are shown as `{` and `}` but they are considerably more flexible. You can choose non-alphanumeric xters as delimiters eg `~r/regexp/`.

## System Types
These types reflect resources in the underlying Erlang VM.

### PIDs and Ports
A PID is a reference to a local or a remote process and a Port is a reference
  to a resource (typically external to the application) that you'll be reading
  or writing.
The PID of the current process is available by calling `self`.

### References
The function `make_ref` creates a globally unique reference.

## Collection Types
Elixir collections can hold values of any type including other collections.

### Tuples
A tuple is an ordered collection of values. It's immutable. Eg: `{ 1, 2 }`, `{ :ok, "Hallo" }`

Tuples can be used in pattern matching. E.g: `{status, count, action} = {:ok, 42, "next"}`

### Lists
In Elixir, a list is a linked data structure. They are easy to traverse lineally but expensive to traverse randomly.
  - `++` is used in concatenation of lists
  - `--` is used to return the difference between lists
  - `in` is used to return membership in a list - boolean

#### Keyword lists:
For example: `[name: "Robley", city: "Nairobi", likes: "Elixir"]`

Elixir converts the above into: `[{:name, "Robley"}, {:city, "Nairobi"}, {:likes, "Elixir"}]`

Elixir allows us to omit the square brackets if a keyword list is the last argument in a function call.

For example: `DB.save record, [{:use_transaction, true}, {:logging, "TRACE"}]` can be written as: `DB.save record, use_transaction: true, logging: "TRACE"`

### Maps
A map is a collection of key/value pairs. Syntax: `%{ key => value, key => value }`.
Example: `%{"name" => "Robley", "language" => "Elixir"}`

It can have atoms as keys, eg: `%{ key: value, key: value }`. Example: `%{name: "Robley", language: "Elixir"}`

If the keys are atoms, you can also use a dot notation to access them. For example if the map above was called **sample**: `IO.puts sample.name` would print `Robley` to the console. 

> Maps only allow one entry for a particular key while keyword lists allow keys to be repeated.

### Binaries
Binary literals are enclosed between `<<` and `>>`. For example `bin = <<1,2>>`

You can add modifiers to control the type and size of each individual field. For example: `bin = << 2 :: size(2), 5 :: size(4), 1 :: size(2) >>`

> Elixir uses bytes to represent UTF strings

## Names, Source Files, Conventions, Operators e.t.c.
Identifiers in Elixir are combos of upper- and lowercase ASCII xters, digits and underscores. They may end with a question mark or exclamation mark.

**Module**, **record**, **protocol**, and **behavior**, names start with an uppercase letter and are `BumpyCase` or `PascalCase`.

> By convention, source files use two-character(spaces) indentation for nesting.

## Truth
Elixir has 3 special values related to Boolean operations: `false`, `true` `nil`. **Nil** is treated as `false` in Boolean contexts.

All three of these values are aliases for atoms of the same name, so `true` is the same as the atom `:true` and `false` is `:false`.

In most contexts, any other value other than `false` and `nil` is treated as `true`. This is referred as `truthy`

## Operators

### Comparison operators:
  - **a === b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; strict equality (`so 1 === 1.0 is false`)
  - **a !== b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; strict inequality (`so 1 !== 1.0 is true`)
  - **a == b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; value equality (`so 1 == 1.0 is true`)
  - **a != b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; value inequality (`so 1 != 1.0 is false`)
  - **a > b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; normal greater than comparison (`so 2 > 1 is true`)
  - **a >= b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; normal greater than or equal to comparison (`so 2 >= 1 is true`)
  - **a < b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; normal less than comparison (`so 1 < 2 is true`)
  - **a <= b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; normal less than or equal to comparison (`so 1 <= 2 is true`)

> The rule for comparison of types is:
   **number < atom < reference < function < port < pid < tuple < map < list < binary**

### Boolean Operators
These operators expect `true` or `false` as their first argument. For example:
  - a or b
  - a and b
  - not a

#### Relaxed Boolean Operators
These operators take arguments of any type. Any value apart from `nil` or `false` is interpreted as `true`. For example:
  - **a || b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; a if a is truthy, otherwise b
  - **a && b** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; b if a is truthy, otherwise a
  - **!a** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; false if a is truthy, otherwise true

### Arithmetic Operators
They include:
  - **+** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; Addition operator
  - **-** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; Subtraction operator
  - **\*** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; Multiplication operator
  - **/** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; Division operator (`yields a floating-point result`)
  - **div** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; Division operator (`returns an integer value`)
  - **rem** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; Modulus operator (`returns the remainder`)

### Join Operators
  - **binary1 <> binary2** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; concatenates two binaries
  - **list1 ++ list2** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; concatenates two lists
  - **list1 -- list2** &nbsp;&nbsp; 👉🏽 &nbsp;&nbsp; returns elements in list1 not in list2

### The `in` Operator
`a in enum` tests if `a` is included in `enum`. For example, a list or a range
