# OTP: 

## OTP: Servers

**OTP === Open Telecom Platform**

It was initially used to build telephone exchanges and switches.

`OTP` is actually a bundle that includes Erlang, a database (called Mnesia), and an innumerable number of libraries. It also defines a structure for your applications.

##3 Some OTP Definitions

OTP defines systems in terms of hierarchies of `aplications`. An application consists of one or more processes. These processes follow one of a small number of OTP conventions, called `behaviors`. 

Each implementation of one of these behaviors will run in its own process (and may have addidional associated processes).

### GenServer
Let's look at the server behavior called **GenServer**.

A special behavior called `supervisor` monitors the health of these processes and implements strategies for restarting them if needed.

### An OTP Server

When we write an `OTP server`, we write a module containing one or more callback functions with standard names. OTP will invoke the appropriate callback to hande a particular situation.

#### Our First OTP Server

You pass it a number when you start it up, and that becomes the current state of the server. We are going to use `mix`.

The `call` function calls a server and waits for a reply. Sometimes you wont want to wait because there's no reply coming back. In such moments, use the GenServer `cast` function.

Just like `call` is passed to the `handle_call` function, `cast` is passed to `handle_cast` function.

To recompile from source use the `r` command within the iex session. Example:
```
iex> r Sequence.Server
```

Even though you have recompiled the code, the old version is still running. The VM doesn't hot swap code until you explicitly access it by module name.

You can enable debugging by tracing a server's execution when using `start_link`, example: 
```
GenServer.start_link(Sequence.Server, 100, [debug: [:trace]])
```

We can also include `:statistics` in the debug list to ask a server to keep some basic statistics.

The Erlang `sys` module is your interface to the world of system messages.

### GenServer Callbacks

`GenServer` is an `OTP protocol`. OTP works by assuming your module defines a number callback functions. When you add the line `use GenServer` to a module, Elixir creates default implementations of the six callback functions of a `GenServer`. All we have to do is override the ones where we add our own behavior.

The methods include:
  - init(start_arguments)
  - handle_call(request, from, state)
  - handle_cast(request, state)
  - handle_info(info, state)
  - terminate(reason, state)
  - code_change(from_version, state, extra)
  - format_status(reason, [pdict, state])

#### Naming a Process

Local naming of processes is possible. We assign a name that is unique for all OTP processes on the server, and then we use that name instead of the PID. 

To create a locally named process:
```
{ :ok, pid } = GenServer.start_link(Sequence.Server, 100, name: :seq)
```

## OTP: Supervisors

Elixir supervisors perform process monitoring and restarting.

### Supervisors and Workers

An Elixir `supervisor` has just one purpose - it manages one or more worker processes. A supervisor is a process that uses the OTP supervisor behavior.

It is given a list of processes to monitor and told what to do if a process dies, and how to prevent restart loops.

### Managing Process State Across Restarts

Supervisors Are the Heart of Reliability.

> OTP has been said to be used to build systems with 99.999999999% reliability.

## OTP: Applications

In the OTP world an application is a bundle of code that comes with a descriptor. The descriptor tells the runtime what dependencies the code has, what global names it registers, etc.

### The Application Specification File

Mix will refer to a file called `name.app` where `name` is the name of your application. This file is called an `application specification` and is used to define your application to the runtime environment. 

Mix creates this file automatically from the informantion in `mix.exs` combined with information it gleans from compiling your application.

### Hot Code Swapping

OTP programs and any ELixir program can in fact update their code while they are running. 

OTP provides a release-management framework that handles hot code swapping.

First, the real deal is swapping state, not code. However, server processes maintain state and likely changes to the server code will change the structure of the state they hold. 

So OTP provides a standard server callback that lets server inherit the state from a prior version of itself.


