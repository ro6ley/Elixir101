# Linking Modules: Behaviors and Use

## Behaviors

An Elixir behavior is nothing more than a list of functions. A module that declares that it implements a particular behavior must implement all of the associated functions.

A behavior is a little like an `interface in Java`, i.e. a set of function signatures that a module has to implement. A module uses it to declare that it implements a particular interface.

## Defining Behaviors

We define a behavior using the ELixir Behaviour module, combined with `defcallback` definitions. Example:
```
defmodule URI.Parser do
  @moduledoc """
  Defines the behavior for each URI.Parser.
  Check URI.HTTP for a possible implementation.
  """
  use Behaviour

  @doc """
  Responsible for parsing extra URL information
  """
  defcallback parse(uri_info :: URI.Info.t) :: URI.Info.t

  @doc """
  Responsible for returning the default port
  """
  defcallback default_port() :: integer
end
```

## Declaring Behaviours

Having defined the behaviour, we can declare that some other module implements it using the `@behaviour` attribute:
```
defmodule URI.HTTP do
  @behaviour URI.Parser

  def default_port(), do: 80
  def parse(info), do: info

end
```

Behaviours give us a way of both documenting and enforcing the public functions that a module should implement.

## Use and __using__

You pass `use` a module along with an optional argument, and it invokes the function or macro `__using__` in that module, passing it the argument.
