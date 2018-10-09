# Control Flow

## If and Unless

`If` and `Unless` take two parameters: a `condition` and a `keyword list`, which can contain the keys `do:` and `else:`. If the `condition` is `truthy`, then `if` expression evaluates the code associated with the `do:` key; otherwise it evaluates the `else:` code. Either branch may be absent, example:
```
IO.puts if 1 == 1, do: "true part", else: "false part"  #=> "true part"

IO.puts if 1 == 2, do: "true part", else: "false part"  #=> "else part"
```

This can also be written as:
```
if 1 == 1 do
  "true part"
else
  "false part"
end
```

`Unless` is similar:
```
IO.puts unless 1 == 1, do: "error", else: "ok"  #=> "ok"

# OR

unless 1 == 2 do
  "OK"
else
  "error"
end
```

## Cond

The `cond` macro lets you list out a series of conditions, each with associated code. It executes the code corresponding to the first truthy conditions.

Fizzbuzz example:
```
defmodule FizzBuzz do
  def upto(n) when n > 0, do: _upto(1, n, [])

  defp _upto(_current, 0, result), do: Enum.reverse result

  defp _upto(current, left, result) do
    next_answer =
      cond do
        rem(current, 3) == 0 and rem(current, 5) ->
          "FizzBuzz"
        rem(current, 3) == 0 ->
          "Fizz"
        rem(current, 5) == 0 ->
          "Buzz"
        true ->
          current
      end
    _upto(current+1, left-1, [ next_answer | result ])
  end
end
```

## Case

Case lets you test a value against a set of patterns, executes the code associated with the first one that matches, are retursn the value of that code. The patterns may include guard clauses. Example:
```
case File.open("case.ex") do
  {:ok, file} ->
    IO.puts "First line: {IO.read(file, :line)}"
  {:error, reason} ->
    IO.puts "Failed to open file: {reason}"
end
```

We can emply guard clauses with case to refine the patterns used when matching functions:
```
defmodule Bouncer do
  dave = %{ name: "Dave", age: 27 }

  case dave do
    person = %{age: age} when is_number(age) and age >= 21 ->
      IO.puts "You are cleared to enter the FooBar, {person.name}"

    _ ->
      IO.puts "Sorry, no admission"
  end
end
```

## Raising Exceptions

Exceptions in Elixir are not control-flow structures, they are intended for things that should never happen in normal operation.

Raise an exception with `raise` function - at its simplest, you pass it a string and it generates an exception of type `RunTimeError`. Eg:
```
raise "Giving up"       #=> "** (RuntimeError) Giving up"
```

You can also pass the type of the exception, along with other optional attributes. All exceptions implement at least the message attribute.
```
raise RuntimeError, message: "Override message"
```

> A trailing exclamation mark, i.e., `File.open!` means that the function will raise an exception on error.
