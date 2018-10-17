# Modules and Named Functions

## Modules
In Elixir named functions must be written inside modules. Example:
```
defmodule Times do
  def double(n) do
    n * 2
  end

  def triple(n), do: n * 3

  def quadruple(n), do: double(n) |> double()
  
  # The above can also be written as:
  # This is also the actual underlying syntax
  
  def new_double(n), do: n * 2
  
  # You can pass multiple lines to the do clause using parenthesis as follows:
  def another_double(n), do: (
    IO.puts n
    IO.puts "You entered {n}"
  )
end
```

The `defmodule` can also be rewritten as:
```
defmodule Times_, do: (def new_time(n), do: n * 2)

IO.puts Times.quadruple 4
```

## Named Functions
Named functions use pattern matching to bind their parameter list to the passed arguments. We write function definition multiple times, each time with its own parameter list.

When you call a named function Elixir tries to match your arguments with the parameter list of the first definition , if it cannot match them, it tries the next definition of the same function to see if it matched, and so on.

```
defmodule Factorial do
  def of(0), do: 1
  def of(n) when n > 0 do
    n * of(n-1)
  end
end

IO.puts Factorial.of 7
```

> Note: The order of clauses matters since Elixir tries functions from the top down and executes the first match.

Examples:
Function sum(n) that calculates the sum of numbers from 1 to n
```
defmodule NewMath do
  # Sum
  def sum(0), do: 0
  def sum(1), do: 1
  def sum(n) when n > 0 do
    n + sum(n-1)
  end

  # GCD
  def gcd(x, 0), do: x
  def gcd(x, y), do: gcd(y, rem(x, y))
end

IO.puts NewMath.sum(4)
IO.puts NewMath.gcd(6,3)
```

### Guard Clauses
These are predicates that are attached to a function definition using one or more `when` keywords. When doing pattern matching, Elixir first does the conventional parameter-based match and then evaluates any `when` predicates executing the function only if at least one predicate is true. Example:
```
defmodule Guard do
  def what_is(x) when is_number(x) do
    IO.puts "{x} is a number."
  end
  def what_is(x) when is_list(x) do
    IO.puts "{x} is a list"
  end
  def what_is(x) when is_atom(x) do
    IO.puts "{x} is an atom boi"
  end
end

Guard.what_is(99)
Guard.what_is([1,2,3])
Guard.what_is(:a)
```

#### Guard Clause limitations
You can only use a subset of Elixir expressions whe writing Guard clauses:
  - Comparison operators: `==, !=, ===, !==, <, >, <=, >=`
  - Boolean and negation operators: `or`, `and`, `not`, `!`. `||` and `&&` are not allowed
  - Arithmetic operators: +, -, /, *
  - The `in` operator
  - Join operators: `<>` and `++`
  - Type-check functions that are inbuilt: `is_atom`, `is_binary`, `is_float` etc.
  - Other functions: `abs(number)`, `bit_size(bitstring)` etc.

### Default Parameters
The syntax for giving default parameters is `param \\ default_value`. You can add a function head with no body that contains the default parameters, and use the regular parameters for the rest.

```
defmodule GuessModule do
  def guess(actual, range) do
  end

  def guess(actual, range \\ 1..10) do
  ...
  end
end
```
### Private Functions
These are functions that can only be called within the module that declares them. They are defined using the `defp` macro.

Private functions with multiple heads just like `def` but you can't have some private and some public, they all have to be private when you have different parameters.

### |> - THe Pipe Operator
This `|>` operator takes the result of the expression on its letf and inserts it as the first parameter of the function invocation to its right.

`val |> f(a, b)` is basically the same as calling `f(val, a, b)`

> NOTE: YOu should always use parentheses around function parameters in pipelines.

## Modules

Modules provide namespaces for things you define. They act as wrappers for named functions, macros, structs, protocols & other modules.

If we want to reference a function defined in a module from outside that module, we need to prefix the reference with the module's name, i.e `Mod.func`

To access a function in a nested module from the outside scope, prefix it with the all the module names.

Module nesting in Elixir is an illusion - all modules are defined at the top level. When we define a module inside another, Elixir simply prepends the outer module name to the inner module name, putting a dot between the two. This means we can directly define a nested module, i.e
```
defmodule Mx.Tasks.DocTst do
  def run do
    "This function is inside a nested module"
  end
end

Mx.Tasks.DocTst.run
```

### Directives for modules

Elixir has three directive that simplify working with modules and all three are `lexically scoped` - it starts at the point the directive is encountered and stops at the end of the enclosing scope - module or function.

#### The Import Directive

The `import` directive brings a module's functions and/or macros into the current scope. When you import a module's function, you will be able to call it without having to specify the module name.

The full syntax of import is: 
```
import Module [, only: | except: ]
```
The second parameter lets you control which funnctions or macros are imported. You write `only:` or `except:` followed by a list of `name: arity` pairs. Example: 
```
import List, only: [flatten: 1, duplicate: 2]
```
You can also give `only:` the atoms `:functions` or `:macros` and import will bring only functions or macros.

#### The Alias Directive

The `alias` directive creates an alias for a module. Example:
```
defmodule Example do
  def func do
    alias Mix.Tasks.Doctest, as: Doctest
    doc = Doctest.setup
    doc.run(Doctest.defaults)
  end
end
```

The `as:` parameter defaults to the last part of the module name.

#### The Require Directive

You `require` a module if you want to use the macros defined in that module. `require` directive ensures that the given module is loaded before your code tries to use any of the macros it defines.

#### Directives in Summary:
> - **import** directive brings in functions to be used inside your module. **use** directive brings in functions to be used AND exposes them publicly on your module.
> - **import** directive brings all the Functions and Macros of Module un-namespaced into your module.
> - **require** directive allows you to use macros of Module but does not import them. (_Functions of Module are always available namespaced._)
> - **use** directive first **requires** module and then calls the **\_\_using\_\_** macro on Module
> - In short, in Elixir, you don't need to **import** modules. All public functions can be accessed by full-qualified **MODULE.FUNCTION** syntax e.g. `String.trim()`. We use **import** whenever we want to easily access functions or macros from other modules without using the full-qualified **MODULE.FUNCTION** syntax.
> - You don't need to **import** a module to use its functions, but you need to **require a module to use its macros**


### Modules Attributes

Modules have associated metadata. Each item of metadata is called an `attribute` of the module and is identified by a name. Inside a module, you can access these attributes by prefixing the name with an `@ sign`.

You can give an attribute a value using the syntax: `@name value`. This only works only at the top level of a module - you cannot set an attribute value inside a function definition, you can access them inside the functions though. Example:
```
defmodule Exmpl do
  @author "Robley"
  def get_author do
    @author
  end
end

IO.puts "Example was written by #{Exmpl.get_author}"
```

You can set the same atribute multiple times in a module - if you access that attribute in that module the value you will see will be the value in effect when the function is defined.

> Note: Attributes should be used for configuration and metadata - as constants

## Module Names: ELixir, Erlang, and Atoms

Internally, module names are just atoms. When you write a name starting with an uppercase letter, such as `IO`, Elixir converts it internally into an atom called `Elixir.IO`.

```
is_atom IO            #=> true
to_string IO          #=> "Elixir.IO"
:"Elixir.IO" === IO   #=> true
```

We can call functions like `:"Elixir.IO".puts 123`

## Calling a function in an Erlang library

To call an Erlang module name in Elixir we simply change the Erlang module name to an Elixir atom. Example, the `io` would be called as `:io.format`
