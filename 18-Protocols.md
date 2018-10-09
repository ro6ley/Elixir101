# Protocols - Polymorphic Functions

A protocol is a little like the `behaviours` in that it defines the functions that must be provided to achieve something. Protocol's impementation can be placed completely outside the module.

## Defining a protocol

A protocol is defined using `defprotocol` keyword and can include module- and function-level documentation. A protocol definition will include one or more function definitions but they will not have bodies - they are there simply to declare the interface that the protocol requires. Example:
```
defprotocol Inspect do
  def inspect(thing, opts)
end
```

## Implementing protocols

The `defimpl` macro lets you give Elixir the implementation of a protocol for one or more types. Example:
```
defimpl Inspect, for: PID  do
  def inspect(pid, _opts) do:
    "PID" <> iolist_to_binary(pid_to_list(pid))
  end
end
```
## The Available Types

You can define implementaions for one or more of the following types: `Any`, `Atom`, `BitString`, `FLoat`, `Function`, `Integer`, `List`, `PID`, `Port`, `Record`, `Reference` and `Tuple`.

## Protocols and Structs

`inspect` recognizes structs

The built in `Access` protocol defines the `[]` operator for accessing members of a collection.

The `Enumerable` protocol is the basis of all the functions in the `Enum` modules.
