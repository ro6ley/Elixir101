# Functions

Elixir is a functional language.

## Anonymous functions
An anonymous function is created using the `fn` keyword:
```
fn
  parameter-list -> body
  parameter-list -> body...
end

# Examples
sum = fn (a, b) -> a + b end
IO.puts sum.(1,2)
```

We use a `dot` before the parenthesis when calling anonymous functions, even if the function takes no arguments. Eg:
```
greeting = fn () -> IO.puts "Hello" end
greeting.()
```

You can omit the parenthesis in a function declaration. Eg:
```
f1 = fn a, b -> a * b end
f1.(5,6)
f2 = fn -> IO.puts 99 end
f2.()
```

Elixir uses pattern matching to bind variables to function arguments. Example of a function to swap values:
```
swap = fn {a, b} -> {b, a} end
```

Other examples of anonymous functions:
```
list_concat = fn (a, b) -> a ++ b end
list_concat.([:a, :b], [:c, :d]) #=> [:a, :b, :c, :d]

sum = fn a, b, c -> a + b + c end
sum.(1,2,3)

pair_tuple_to_list = fn {a, b} -> [a, b] end
```

A single function definition can allow you to define different implementations depending on the type and contents of the arguments passed.
> Note: You cannot select based on a number of arguments, each definition must have the same number od parameters.
Example:
```
handle_open = fn
  {:ok, file} -> "Read data: {IO.read(file, :line)}"
  {_, error} -> "Error: {:file.format_error(error)}"
  end
```

Elixir does string interpolation - the contents of `{...}` are evaluated then then the result is substituted back in.

Fizzbuzz example:
```
fizz_buzz = fn
  {0, 0, _} -> "FizzBuzz"
  {0, _, _} -> "Fizz"
  {_, 0, _} -> "Buzz"
  {_, _, x} -> x
end

IO.puts fizz_buzz.({0,1,2}) #=> "Fizz"
IO.puts fizz_buzz.({0,0,2}) #=> "FizzBuzz"
IO.puts fizz_buzz.({1,0,2}) #=> "Buzz"
IO.puts fizz_buzz.({3,4,6}) #=> x
```

New `rem` function example:
```
new_rem = fn (n) -> IO.puts fizz_buzz.({rem(n, 3), rem(n, 5), n}) end

new_rem.(10) #=>
new_rem.(11) #=>
new_rem.(12) #=>
new_rem.(13) #=>
new_rem.(14) #=>
new_rem.(15) #=>
new_rem.(16) #=>
new_rem.(17) #=>
```

Functions can return functions. Example:
```
inception_fun = fn -> fn -> "Hello" end end
IO.puts inception_fun.().() Hello
```
Calling `inception_fun.()` returns the inner function. Calling `inception_fun.().()` will evaluate the inner function and `Hello` is returned.

We can use parenthesis to make the inner function more obvious, example:
```
fun_a = fn -> (fn -> "Hello" end) end
```

We can call the outer function and bind the result to a different variable. Example:
```
fun1 = fn -> fn -> "Hello" end end
fun2 = fun1.()
fun2.() Hello - inner function call
```

Functions remember their original environmwnt.

Functions in Elixir automatically carry with them the bindings of variables in the scope in which they are defined. In the case of an inner function accessing a variable in the parent scope, when the inner function is defined, it inherits the scope of the parent and carries the binding of the variables around with it.

This is a **CLOSURE** - the scope encloses the bindings of its variables.

Example:
```
prefix = fn string -> (fn new_string -> string <> " " <> new_string end) end
mrs = prefix.("Mrs")
IO.puts mrs.("Smith") "Mrs Smith"
IO.puts prefix.("Elixir").("Rocks") "Elixir Rocks"
```

### Passing Functions As Arguments
Functions are just values so they can be passed to other functions as arguments
```
times_2 = fn n -> n * 2 end
apply = fn (fun, val) -> fun.(val) end
IO.puts apply.(times_2, 12) #=> 24
```

The built in `Enum` modules has a function called `map` takes two arguments, `a collection and a function`. Example:
```
Enum.map([1,2,3,4,5], fn elem -> elem > 2.5 end) #=> [false, false, true, true, true]
```

## The & Notation
This is a shortcut to creating anonumous functions. The placeholders `&1, &2, ...` correspond to the first, second and subsequent parameters of the function.

So `&(&1 + &2)` will be converted to `p1, p2 -> p1 + p2`. Example:
```
add_one = &(&1 + 1) same as add_one = fn n -> n + 1 end
IO.puts add_one.(44) 45

speak = &(IO.puts(&1))
speak.("Hello")
```

Because [] and {} are operators in Elixir, literal lists and tules can also be turned into functions, example:
```
divrem = &{ div(&1, &2), rem(&1, &2) }
divrem.(13, 5)
```

## Function capturing
This enables us create lambda/anonymous functions out of named functions.

You can give it the name and the arity of an existing function and it will return an anonymous function that calls it. The arguments that you pass to the anonymous function will be passed to the named function. Example:
```
say = &(IO.puts(&1))
say.("Why me Lord?") #=> Why me Lord?
```

> NOTE: The capture operator won't error if the named function does not exist upon capture, it will error when invoked.
