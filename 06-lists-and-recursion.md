# Lists and Recursion

## Heads and Tails
A list can either be empty or consist of a head and a tail. The head contains a value and the tail is a list - this is a recursive definition.
```
An empty list: []
```

We split the head and tail using the `pipe` | character, therefore,
```
[3] === [3 | []]

[1,2] === [1 | [2]] === [1 | [2 | []]]
```

The pipe operator can be used in matching, i.e:
```
[head | tail] = [1, 2, 3, 4]
IO.puts head      #=> 1
IO.inspect tail   #=> [2, 3, 4]
```

The length of an empty list is 0, the length of a list is 1 plus the length of its tail.
```
defmodule MyList do
  def len([]), do: 0
  def len([_head | tail]), do: 1 + len(tail)     #=> head is unused, use the _notation

  def square([]), do: []
  def square([head | tail]), do: [ head * head | square(tail) ]

  def add_1([]), do: []
  def add_1([head | tail]), do: [head+1 | add_1(tail)]

  def map([], _func), do: []
  def map([head | tail], func), do: [func.(head) | map(tail, func)]

  def sum([], total \\ 0), do: total
  def sum([head | tail], total), do: sum(tail, head + total)

  def new_sum([]), do: 0
  def new_sum([head | tail]), do: head + new_sum(tail)

  def mapsum([], _func), do: 0
  def mapsum([head | tail], func) do
    func.(head)  + mapsum(tail, func)
  end
end

IO.puts MyList.len([])                      #=> 0
IO.puts MyList.len([11, 12, 13, 14, 15])    #=> 4

IO.inspect MyList.square([])                #=> []
IO.inspect MyList.square([4, 5, 6])         #=> [16, 25, 36]

IO.inspect MyList.add_1([])                 #=> []
IO.inspect MyList.add_1([4, 6, 8])          #=> [5, 7, 9]

IO.inspect MyList.map([], &(&1 + 1))            #=> []
IO.inspect MyList.map([0, 1, 2, 3], &(&1 + 1))  #=> [1, 2, 3, 4]

IO.puts MyList.sum([])                       #=> 0
IO.puts MyList.sum([1, 2, 3, 4])             #=> 10

IO.puts "Without using an accumulator:"
IO.puts MyList.new_sum([1, 2, 3, 4])

IO.puts MyList.mapsum([1,2,3], &(&1 * &1))
```

For the sum function that gets the total of values in an array, remembering to include the total is a bit tacky, so we can hide it out - our module will have a public function that takes a list and calls private functions to do the work as shown below:
```
defmodule MyList1 do
  def sum(list), do: _sum(list, 0)

  private methods
  defp _sum([], total), do: total
  defp _sum([head | tail], total), do: _sum(tail, head + total)
end
```

We chose to give our helper functions a preceding underscore. You can also name your helper functions in the format `do_xxx`

### Generalizing Our Sum Function

We can have a function that takes in a collection, an initial value and a function and reduces the collection into a value. i.e. reduce(collection, initial_value, func)
```
defmodule Reducer do
  def reduce([], value, _), do: value
  def reduce([head | tail], value, func) do
    reduce(tail, func.(head, value), func)
  end

  def return_max([]), do: 0
  def return_max([head | tail], a) do
    if a > head do
      return_max(tail, a)
    else
      return_max(tail, head)
    end
  end
end

IO.puts Reducer.reduce([], 0, &{&1 + &2})
IO.puts Reducer.reduce([1, 2, 3], 1, &(&1 * &2))
```

The join operator `|` supports multiple values to its left so you can match multiple individual elements as the head. Example:
```
defmodule Swapper do
  def swap([]), do: []
  def swap([a, b | tail]), do: [b, a | swap(tail)]
  def swap([_]), do: "Can't swap a list with an odd number of elements."
end
```

### The List Module in Action

Concatenate lists using `++`
```
[1, 2, 3] ++ [4, 5, 6]      #=> [1, 2, 3, 4, 5, 6]
```

Flatten lists:
```
List.flatten([[[1], 2], [[[3]]]])    #=> [1, 2, 3]
```

Folding (like reduce, but can chose direction):
```
List.foldl([1,2,3], "", fn value, acc -> "{value}({acc})" end)  #=> 3(2(1()))

List.foldr([1,2,3], "", fn value, acc -> "{value}({acc})" end)  #=> 1(2(3()))
```

Merging lists and splitting them apart:
```
l = List.zip([[1,2,3], [:a,:b,:c], ["cat", "dog"]])   #=> [{1, :a, "cat"}, {2, :b, "dog"}]

List.unzip(l)                                         #=> [[1, 2], [:a, :b], ["cat", "dog"]]
```

Accessing tuples within lists:
```
kw = [{:name, "Dave"}, {:likes, "Programming"}, {:where, "Dallas", "TX"}]   #=> [{:name, "Dave"}, {:likes, "Programming"}, {:where, "Dallas", "TX"}]

List.keyfind(kw, "Dallas", 1)                                               #=> {:where, "Dallas", "TX"}

List.keyfind(kw, "TX", 2)                                                   #=> {:where, "Dallas", "TX"}

List.keyfind(kw, "TX", 1)                                                   #=> nil

List.keyfind(kw, "TX", 1, "No city called TX")                              #=> "No city called TX"

kw = List.keydelete(kw, "TX", 2)                                            #=> [name: "Dave", likes: "Programming"]

kw = List.keyreplace(kw, :name, 0, {:first_name, "Dave"})                   #=> [first_name: "Dave", likes: "Programming"]
```
