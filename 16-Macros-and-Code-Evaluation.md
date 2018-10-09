# Macros and Code Evaluation

**DISCLAIMER: Never use a macro when you can use a function**

## Implementing an if Statement

In the case Elixir didn't have an `if` statement, let's try to implement `if` as a function:
```
defmodule My do
  def myif(condition, clauses) do
    do_clause = Keyword.get(clauses, :do, nil)
    else_clause = Keyword.get(clauses, :else, nil)

    case condition do
      val when val in [false, nil]
        -> else_clause
      _otherwise
        -> do_clause
    end
  end
end

# Sample call
My.myif 1==2, do: (IO.puts "1==2"), else: (IO.puts "1 != 2")
```

## Macros Inject Code

The Elixir compiler reads a module's source top to bottom and generate a representation of the code we find. The representation is in a nested Elixir tuple.

If we want to support macros we need a way to tell the compiler that we'd like to manipulate that tuple. We do that by using `defmacro`, `quote` and `unquote`.

When we pass parameters to a macro, Elixir doesn't evaluate them, instead it passes them as tuples representing their code.

```
defmodule Myy do
  defmacro macro(param) do
    IO.inspect param
  end
end

defmodule Test do
  require Myy

  These values represent themselves
  My.macro :atom      => :atom
  My.macro 1          => 1
  My.macro 1.0        => 1.0
  My.macro [1,2,3]    => [1,2,3]
  My.macro "binaries" => "binaries"
  My.macro { 1, 2 }   => {1,2}
  My.macro do: 1      => [do: 1]
  My.macro do         => [do: 1]
    1
  end

  And these are represented by 3 element tuples
  My.macro { 1, 2, 3, 4, 5 }

  My.macro do: { a = 1; a+a } =>
  [do:
  {:__block__,[],
  [{:=,[line: 22],[{:a,[line: 22],nil},1]},
  {:+,[line: 22],[{:a,[line: 22],nil},{:a,[line: 22],nil}]}]}]

  My.macro do
    1+2
  else
    3+4
  end
end
```

This shows us that atoms, numbers, lists (including keyword lists), binaries and tuples are represented internally as themselves. All other Elixir code is represented by a three-element tuple.

### Load Order

Macros are expanded before a program executes, so the macro defined in one module must be available as Elixir is compiling another module that uses those macros.

The `require` function tells Elixir to ensure the named module is compiled before the current one. Therefore, the macro should've been defined in a separate file to be lopaded into another.

## The Quote Function

When we pass parameters to a macro, they are not evaluated. The language comes with a function `quote` that also forces code to remain in its unevaluated form.

`quote` takes a block and returns internal representation of that block. `quote` says _interpret the content of the block that follows and return the internal representation_.

### Using the Representation As Code

When we extract the internal representaion of some code, we stop Elixir from adding it automatically to the tuples of code it is building during compilation - we've effectively created a free-standing island of code which has to be injected back into our program's internal representation.

The are two ways to achieve this. First, we can use a macro

## The Unquote Function

We can only use `unquote` inside a `quote` block. Using `unquote` inside a `quote` is a way of deferring the execution of the unquoted code. It temporarily turns off quoting and simply injects a code fragment into the sequence of code being returned by `quote`.

### Using Bindings to Inject Values

There are two ways of injecting values into quoted blocks. One is `unquote` and the other is binding. A binding is simply a keyword list of variable names and their values.

Macros are executed at compile time - this means they don't have access to values that are calculated at runtime.

## Macros are Hygienic

A macro definition has both its own scope and a scope during execution of the quoted macro body. 

A macro's definition is lexically scoped.

We can also use the function `Code.eval_quoted` to evaluate code fragments, such as those returned by `quote`.
