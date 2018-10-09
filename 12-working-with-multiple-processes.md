# Working with Multiple Processes

Elixir uses the `actor` model of concurrency. An `actor` is an independent process that shares nothing with any other processes. You can spawn new processes, send them messages, and receive messages back.

Elixir uses process support in Erlang - these processes will run across all your CPUs (just like native processes), but they have little overhead.

## A Simple Process

Here is a module that defines a function we'd like to run in a separate process:
```
defmodule SpawnBasic do
  def greet do
    IO.puts "Hello"
  end
end
```

To run it in a separate process:
```
spawn(SpawnBasic, :greet, [])
```

The `spawn` function kicks off a new process. It returns a Process Identifier - `PID` which uniquely identifies the process it creates. When `spawn` is called it creates a new process to run the code we specify.

## Sending Messages Between Processes

We send a message using the `send` function. It takes a `PID` and the `message` to send (an Elixir value which we also call a `term`) on the right.

We wait for messages using `recieve`. In a way is acts like a `case` with the message body as the parameter. Inside the receiving block, you can specify any number of patterns and associated actions.

Updated example that sends and receives messages:
```
defmodule Spawn1 do
  def greet do
    receive do                                 #  handle message reception
      {sender, msg} ->
        send sender, { :ok, "Hello, {msg}"}    #  send the response to the sender
        greet                                  #  to handle multiple messages
    end
  end
end
```

here's a client:
```
pid = spawn(Spawn1, :greet, [])    # spawn a process and get its id
send pid, {self, "World!"}         # send a message to the spawned process

receive do                         # receive the message received and print it
  { :ok, message } ->
    IO.puts message
end
```

## Handling Multiple Messages

Let's try sending a second message:
```
defmodule Spawn2 do
  def greet do
    receive do
      { sender, message } ->
        send sender, { :ok, "Hello, {message }"}
        greet
    end
  end
end
```

Here's a client
```
pid_new = spawn(Spawn2, :greet, [])
send pid, {self, "Elixir!"}

receive do
  { :ok, message } ->
    IO.puts "Got: {message}"
end

send pid, {self, "Kermit!"}
receive do
  { :ok, message } ->
    IO.puts "Got: {message}"
  after 500 ->
    IO.puts "The greeter has gone away."
end
```

## Recursion, Looping, and the Stack

The recursive `greet` function above ends up calling itself when it receives a message. In many languages, that adds a new frame to the stack. After a large number of messages, you might run out of memory.

This does not happen in Elixir, as it implements `tail-call optimization` - if the very last thing a function does is call itself, there's no need to make the call. Instead, the runtime can simply jump back to the start of the function. If the recursive call has arguments, then these replace the original parameters as the loop occurs.

## Process Overhead

Elixir process have very low overhead.

## Linking Two Processes

Processes can be linked, and when they are linked, each can receive information when the other exits. The `spawn_link` call spawns a process and links it to the caller in the operation.

```
defmodule Link2 do
  import :timer, only: [ sleep: 1 ]

  def sad_function do
    sleep 500
    exit(:boom)
  end

  def run do
    spawn_link(Link2, :sad_function, [])
    receive do
      msg ->
        IO.puts "Msg received: {msg}."
    after 1000 ->
      IO.puts "Nothing happened as far as I am concerned."
    end
  end
end

Link2.run (EXIT from PID<0.73.0>) :boom
```

From the example above, the child process died and killed the entire application.

That's the default behaviour of linked processes - when one exits abnormally, it kills the other.

Elixir uses the `OTP` framework for contructing process trees, and `OTP` includes the concept of `process supervision`.

However, you can tell Elixir to convert exit signals from a linked process into a message you can handle:
```
defmodule Link3 do
  import :timer, only: [ sleep: 1 ]
  def sad_function do
    sleep 500
    exit(:boom)
  end
  def run do
    Process.flag(:trap_exit, true)             #  trap the exit
    spawn_link(Link3, :sad_function, [])
    receive do
      msg ->
        IO.puts "MESSAGE RECEIVED: {inspect msg}"
    after 1000 ->
        IO.puts "Nothing happened as far as I am concerned"
    end
  end
end

Link3.run
```

## Monitoring a process

`Monitoring` lets a process spawn another and get notified of its termination, but without reverse notification - it is one way only.

When you monitor a process you get a `:DOWN` message when it exits or fails, or if it doesn't exist.

You can use `spawn_monitor` to turn on monitoring when you spawn a process, or you can use `Process.monitor` to monitor an existing process.

## Parallel Map - The "Hello, World" of Erlang

A regular map returns the list that results from applying a function to each element of a collection. The `parallel` version does the same, but it applies the function to each element in a separate process.

```
defmodule Parallel do
  def pmap(collection, fun) do
    me = self
    collection
    |> Enum.map(fn (elem) ->
          spawn_link fn -> (send me, {self, fun.(elem)}) end
        end)
    |> Enum.map(fn (pid) ->
          receive do {^pid, result } -> result end
        end)
  end
end

IO.inspect Parallel.pmap(1..10, &(&1 * &1))
```

### A Fibonacci Server

This example program's task is to calculate `fin(n)` for a list of `n`, where `fib(n)` is the nth Fibonacci number. The twist is that we'll write our program to calculate different Fibonacci numbers in parallel.

To do this we'll write a trivial server process that does the calculation and a scheduler that assigns work to a calculation process when it becomed free.

When the calculator is ready for the next number, it sends a `:ready` msg to the scheduler. If there's still work to do, the scheduler sends it to the calculator in a `:fib` message, otherwise it sends a `:shutdown`. The calc receives a given Fibonacci number and sends back the answer.

```
defmodule FibSolver do
  def fib(scheduler) do
    send scheduler, { :ready, self }
    receive do
      { :fib, n, client } ->
        send client, { :answer, n, fib_calc(n), self }
        fib(scheduler)
      { :shutdown } ->
        exit(:normal)
    end
  end

  # very inefficient, deliberately
  defp fib_calc(0), do: 0
  defp fib_calc(1), do: 1
  defp fib_calc(n), do: fib_calc(n-1) + fib_calc(n-2)
end


defmodule Scheduler do
  def run(num_processes, module, func, to_calculate) do
    (1..num_processes)
      |> Enum.map(fn(_) -> spawn(module, func, [self]) end)
      |> schedule_processes(to_calculate, [])
  end

  defp schedule_processes(processes, queue, results) do
    receive do
      { :ready, pid } when length(queue) > 0 ->
        [ next | tail ] = queue
        send pid, {:fib, next, self}
        schedule_processes(processes, tail, results)
      { :ready, pid } ->
        send pid, {:shutdown}
        if length(processes) > 1 do
          schedule_processes(List.delete(processes, pid), queue, results)
        else
          Enum.sort(results, fn {n1, _}, {n2, _} -> n1 <= n2 end)
        end
      { :answer, number, result, _pid } ->
        schedule_processes(processes, queue, [ {number, result } | results ])
    end
  end
end

# driving the scheduler
to_process = [ 37, 37, 37, 37, 37, 37 ]

Enum.each 1..10, fn num_processes ->
  {time, result} = :timer.tc(Scheduler, :run,
                              [num_processes, FibSolver, :fib, to_process])

  if num_processes == 1 do
    IO.puts inspect result
    IO.puts "\n       time (s)"
  end
  :io.format "~2B       ~.2f~n", [num_processes, time/1000000.0]
end
```
