# Strings and Binaries

## String Literals

Elixir has two kinds of strings: `single-quoted` and `double-quoted`. They differ significantly in their internal represetation. They have the following in common:
  - Strings can hold characters in UTF-8 encoding.
  - They may contain escape sequences such as `\a`, `\b`, `\e`
  - They allow interpolation on Elixir expressions using the syntax `{...}`
  - Characters that would otherwise have a special meaning can be escaped with a backslash.
  - They support `heredocs`.

## Heredocs

Any string can span several lines. `IO.puts` always appends a newline unlike `IO.write`
The `heredoc` notation uses the tripple string delimiter (`'''` or `"""`).

> Heredocs are used extensively to add documentation to functions and modules.

## Sigils

These `~ -style` literals are called **sigils**, eg, for Regex `~r{...}`.

A sigil starts with a tilde (`~`), followed by an upper- or lowercase letter, some delimited content and perhaps some options. The delimiters can be:
```
  <...>, {...}, [...], (...), |...|, /.../, "..." and '...'.
```

The letter determines the sigil type:
```
  ~C - a character list with no escaping or interpolation
  ~c - a character list, escaped and interpolated just like a single-quoted string
  ~R - a regular expression with no escaping interpolation
  ~r - a regular expression, escaped and interpolated
  ~S - a string with no escaping or interpolation
  ~s - a string, escaped and interpolated just like a double-quoted string
  ~W - a list of whitespace-delimited words, with no escaping or interpolation
  ~w - a list of whitespace-delimited words, with escaping and interpolation
```

> ELixir does not check the nesting of delimiters.

If the opening delimiter is three single or double quotes, the sigil is treated a heredoc.

In Elixir we only call double-quoted strings **"Strings"** and single-quoted strings **"CharLists** - List of Character Codes.

Single-quoted strings are represented as a list of integer values, each value corresponding to a codepoint in the string. They are called `character lists` or `char lists`. Example:
```
str = 'wombat'
IO.puts is_list str      #=> true
IO.puts length str       #=> 6
```

`iex` prints a list of intergers as a string if it believes each number in the list is a printable character. Example:
```
[67, 65, 84]      #=>   'CAT'
```

Because a character list is a list, we can use the usual pattern matching and List functions.
```
'pole' ++ 'vault' 'polevault'
'pole' -- 'vault' 'poe'
```

## Binaries

The binary type represents a sequence of bits. A binary literal looks like `<< term,... >>`

The simplest term is just a number from `0` to `255`. The  numbers are stored as successive bytes in the binary.
```
b = << 1, 2, 3 >>
IO.puts byte_size b      #=> 3
IO.puts bit_size b       #=> 24
```

You can specify modifiers to set any term's size (in bits). This is useful when working with binary formats such as media files and network packets.
```
bt = << 1::size(2), 1::size(3) >>
IO.puts byte_size bt     #=>  1
IO.puts bit_size bt      #=>  5
```

You can store intergers, floats and other binaries in binaries.
```
int = << 1 >> <<1>>
float = <<2.5 :: float >> <<64, 4, 0, 0, 0, 0, 0, 0>>
```

> Double-Quoted Strings are Binaries

The contents of double-quoted strings (dqs) are stored as a consecutive sequence of bytes in UTF-8 encoding. This is more efficient in terms of memory and certain forms of access. It has two implications:
  - the size of the binary is not necessarily the length of the string since UTF-8 characters can take more than a single byte to represent.
  - you need to learn and work with the binary syntax alongside the list syntax in your code since you are no longer using lists.


## Strings and Elixir libraries

The `String` module defines a number of functions that work with `dqs`:
- `at(str, offset)` - returns the graphene at the given offset starting at 0. Negative offsets count from the end of the string:
  ```
  IO.puts String.at("dog", 0) "d"
  IO.puts String.at("dog", -1)    #=>  "g"
  ```

- `capitalize(str)` - converts str to lowercase then capitalizes the first character:
  ```
  IO.puts String.capitalize "ecole"       #=>  "Ecole"
  ```

- `codepoints(str)` - returns the codepoints in str:
  ```
  IO.inspect String.codepoints "Jose's dog"    #=>  ["J", "o", "s", "e", "'", "s", " ", "d", "o", "g"]
  ```

- `downcase(str)` - converts str to lowercase:
  ```
  IO.inspect String.downcase "ORDER"       #=>   "order"
  ```

- `duplicate(str, n)` - returns a string containing n copies of str:
  ```
  IO.puts String.duplicate "Ho! ", 3     #=>  "Ho! Ho! Ho!"
  ```

- `ends_with?(str, suffix | [suffixes])` - true if str ends with ny of the given suffixes:
  ```
  IO.puts String.ends_with? "string", ["elix", "stri", "ring"]   #=> true
  ```

- `first(str)` - returns the first grapheme in string:
  ```
  IO.puts String.first("hello")    #=> "h"
  ```

- `graphemes(str)` - returns the grphemes in the string:
  ```
  IO.puts String.graphemes("hello")    #=> ["h", "e", "l", "l", "o"]
  ```

- `last(str)` - returns the last grapheme in the string:
  ```
  IO.puts String.last("hello")    #=> "o"
  ```

- `length(str)` - returns the number of graphemes in string:
  ```
  IO.puts String.length("hello")    #=> 5
  ```

- `ljust(str, new_length, padding \\ "")` - returns a new string, at least new_length characters long, containing `str` left-justified and padded with padding:
  ```
  IO.puts String.ljust("cat", 5)     #=>   "cat     "
  ```

- `trim_leading(str)`, _previously `lstrip(str)`_ - removes leading whitespace from string:
  ```
  IO.puts String.lstrip("      cat")     #=> "cat"

  # The above is deprecated and you can now use the following:
  IO.puts String.trim_leading("      cat")     #=> "cat"
  ```

- `trim_leading(str, character)` - removes leading copies of character from string:
  ```
  IO.puts String.trim_leading("aaaaacat", "a")     #=> "cat"
  ```

> The above are just but a few, there are many more methods in the [official Elixir Docs](https://hexdocs.pm/elixir/String.html#functions)


## Binaries and Pattern Matching

The first rule of binaries is `"if in doubt, specify the type of each field"`.

Available types are `binary`, `bits`, `bitstring`, `bytes`, `float`, `integer`, `utf8`, `utf16`, and `utf32`. You can also add qualifiers:
  - size(n): The size in bits of the field.
  - signed and unsigned: For integer fields, should it be interpreted as signed?
  - endiannes: big, little or native

Use hyphens to separate multiple attributes for a field. Eg:
```
<< length::unsigned-integer-size(12), flags::bitstring-size(4) >> = data
```

### String processing with binaries

When we process lists, we use patterns that split the head from the rest of the list. With binaries that hold strings, we can do the same kind of trick.

We have to specify the type of the head(UTF-8), and make sure the tail remains a binary.

The parallels with list processing are clear, but the differences are significant. Rather than use `[ head | tail ]`, we use `<< head::utf-8, tail:binary >>`. And rather than terminate when we reach the empty list, `[]`, we look for an empty binary.
