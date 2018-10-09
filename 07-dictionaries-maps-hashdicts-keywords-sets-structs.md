# Dictionaries: Maps, HashDicts, Keywords, Sets and Structs

A dictionary is a data type that associates keys with values.

### How to choose between Maps, HashDicts and Keywords?

The following are the control questions:
- Will I want more than one entry with the same key? - Use `Keyword`
- Do I need to guarantee that the elements are ordered? - Use `Keyword`
- Do I want to pattern match against the contents? - Use `Map`
- Will I be storing more than a few hundred entries in it? - Use `HashDict`

## Dictionaries

Maps and hashdicts both implement the `Dict` behavior. The `Keyword` largely does to but with some differences to allow for duplicate keys.

### Pattern matching

This helps us find out if our maps include certain keys or values. Example:
```
person = %{name: "Dave", height: 1.88}

%{name: a_name} = person           #=> same as %{:name => a_name} = person
IO.puts a_name                     #=> "Dave"
```
In action:
```
people = [
  %{ name: "Grumpy", height: 1.24 },
  %{ name: "Dave", height: 1.88 },
  %{ name: "Dopey", height: 1.32 },
  %{ name: "Shaquille", height: 2.16 },
  %{ name: "Sneezy", height: 1.28 }
]

#=> Feeding a list of maps to a comprehension:

for person = %{name: name, height: height} <- people,
  height < 1.5,
  do: IO.inspect person

defmodule HotelRoom do
  def book(%{name: name, height: height}) when height > 1.5 do
    IO.puts "Need an extra long bed for {name}"
  end

  def book(%{name: name, height: height}) when height < 1.3 do
    IO.puts "Need shower controls for {name}"
  end

  def book(person) do
    IO.puts "Need regular bed for {person.name}"
  end
end

people |> Enum.each(&HotelRoom.book/1)
```

> Pattern matching cant bind keys.
> 
> Maps do not allow you to bind a value to a key during pattern matching.

#### Updating a Map

With maps we can add new `key/value` entries and update existing entries without traversing the entire structure.

But as with all values in Elixir, a map is **IMMUTABLE** so the result of an update is a new map.
The simplest way to UPDATE a map is with this syntax:
```
  new_map = %{old_map | key => value, ... }
```
This creates a new map that is a copy of the old, but the values associated with the keys on the right of the pipe character are updated.
```
m = %{a: 1, b: 2, c: 3}
m1 = %{ m | b: "two", c: "three"}
IO.inspect m1
m2 = %{ m | a: "one"}
IO.inspect m2
```

To add a new key to a map, use the `Dict.put_new/3` function:
```
m3 = Dict.put_new(m, :d, 4)
IO.inspect m3
```

## Structs

A `struct` is simply a module that wraps a limited form of a map. This is because the keys must be atoms, and the maps don't have `Dict` or `Access` capabilities.

The name of the module becomes the name of the map tyoe. The macro `defstruct` is used to define the maps characteristics. Example:
```
defmodule Subscriber do
  defstruct name: "", paid: false, over_18: true

  def c_i do
    s1 = %Subscriber{}
    s2 = %Subscriber{ name: "Robley"}
    s3 = %Subscriber{ name: "Mary", paid: true}
    %Subscriber{name: a_name} = s2

    IO.inspect s1
    IO.inspect s2
    IO.inspect s3.name
    IO.puts a_name

    Updating
    s4 = %Subscriber{ s1 | name: "He had no name before"}

    IO.inspect s4
  end
end

Subscriber.c_i
```

The syntax for creating a struct is the same as for creating a map, you just add the module name between `%` and `{`.

You can access the fields in a struct using the dot notation or pattern matching.

Updates also follow suit.

Structs are wrapped in a module since you may likely want to add struct-specifig behavior.

#### Another way to access structs

You can access maps using `some_map[:key]` but this is not possible with structs since Maps implement the **Access protocol** (_which defines the ability to access feilds using square bracket notation_) and structs do not. 

You can add this ability to your structs using the `@derive` directive. Example:
```
defmodule Attendee do
  @derive Access
  defstruct name: "", paid: false, over_18: false

  def a_c do
    a = %Attendee{name: "First person"}

    IO.puts a[:name]
  end

end

Attendee.a_c
```

### Nested Dictionary Structures

Values can be dictionaries too. Example:
```
defmodule Customer do
  defstruct name: "", company: ""
end

defmodule BugReport do
  defstruct owner: %{}, details: "", severity: 1

  def m_i do
    report = %BugReport{owner: %Customer{name: "Robley", company: "Andela"},
              details: "Failure"}

    IO.inspect report 
    IO.puts report.owner.name Robley

    Updating a nested value using `put_in`
    IO.inspect put_in(report.owner.name, "Gori")

    `update_in` function allows us to apply a function to a value
    IO.inspect update_in(report.owner.name, &("Mr. " <> &1))

    Other functions include `get_in` and `get_and_update_in`
  end
end

BugReport.m_i
```

### Nested Accessors and Nonstructs

The nested accessor functions use the Access protocol to strip part and reassemle data structures.

## Dynamic (Runtime) Nested Accessors

The nested accessor we've seen so far are macros - they operate at compile time. As a result they have several limitations:
  - The number of keys you pass a particular call is static
  - You can't pass the set of keys as parameters between functions

To overcome this, `get_in`, `put_in`, `update_in` and `get_and_update_in` can all take a list of keys as a separate parameter. Adding this parameter changes them from macros to function calls, so they become dynamic.

|                   | Macro            |  Function           |
|-------------------|------------------|---------------------|
|`get_in`           | no               | (dict, keys)        |
|`put_in`           | (path, value)    | (dict, keys, value) |
|`update_in`        | (path, fn)       | (dict, keys, fn)    |
|`get_and_update_in | (path, fn)       | (dict, keys, fn)    |

Simple example
```
nested = %{
  buttercup: %{
    actor: %{
      first: "robin",
      last: "wright"
    },
    role: "princess"
  },
  wrestley: %{
    actor: %{
      first: "carey",
      last: "ewes"
    },
    role: "farm boy"
  },
}

IO.inspect get_in(nested, [:buttercup])

IO.inspect get_in(nested, [:buttercup, :actor])

IO.inspect get_in(nested, [:buttercup, :actor, :first])

IO.inspect put_in(nested, [:wrestley, :actor, :last], "elwes")
```

The dynamic versions of both `get_in` and `get_and_update_in` support a cool cool trick - if you pass in a function as a key, that function is invoked to return the corresponding values.

## Sets

There is currently just one implementation of sets, the `HashSet`. Examples:
```
set1 = Enum.into 1..5, HashSet.new

IO.inspect set1                         #=> HashSet<[2, 3, 4, 1, 5]>

IO.puts Set.member? set1, 3             #=> true

set2 = Enum.into 3..8, HashSet.new

IO.inspect set2                         #=> HashSet<[7, 6, 3, 4, 5, 8]>

IO.inspect Set.union set1, set2         #=> HashSet<[7, 6, 4, 1, 8, 2, 3, 5]>

IO.inspect Set.difference set1, set2    #=> HashSet<[1, 2]>

IO.inspect Set.difference set2, set1    #=> HashSet<[6, 7, 8]>

IO.inspect Set.intersection set1, set2  #=> HashSet<[3, 4, 5]>
```
