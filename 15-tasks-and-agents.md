# Tasks and Agents

## Tasks

An Elixir task is a function that runs in the background.
```
defmodule Fib do
  def of(0), do: 0
  def of(1), do: 1
  def of(n), do: Fib.of(n-1) + Fib.of(n-2)
end

IO.puts "Start the task"
worker = Task.async(fn -> Fib.of(20) end)
IO.puts "Do sth else"

IO.puts "Wait for the task"
result = Task.await(worker)

IO.puts "The result is {result}"
```

The call to `Task.async` creates a separate process that runs the given function. The return value of `async` is the task descriptor - actually a PID and a ref, that we'll use to identify the task later.

Once the task is running, the code continues with other work, when it wants to get the function's value, it calls the `Task.await`, passing it the task descriptor. This waits for our task to finish and returns its value.

We can also pass `Task.async` the name of a module and function along with any arguments. Example:
```
worker = Task.async(Fib, :of, [20])
result = Task.await(worker)
```

## Tasks and Supervision

Tasks are implemented as `OTP servers`, which means we can add them to our application's supervision tree. We can link the task to a currently supervised process through `start_link` instead of async. We can also run the task directly from a supervisor.

## Agents

An `Agent` is a background process that maintains state. This state can be accessed at different places within process or node, or across multiple nodes.

The initial state is set by a func we pass in when we start the agent.
