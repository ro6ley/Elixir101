# Nodes - The Key to Ditributing Services

A node is simply a running Erlang VM (BEAM). The BEAM handles its own events, process scheduling, memory, naming services, and interprocess communication. A node can connect to other nodes - in the same computer, across a LAN or across the internet.

## Naming Nodes

We can set the name of a node when we start it, with `iex` we use the `--name` or `--sname` option, examples:
```
# Setting a quaified name
iex --name robley@fp.local

# Setting a shortname
iex --sname robley
```

We can spawn processes in other nodes using the node name. For example, if we have another node called `node_two@Mufasa`, to spawn a process on it:
```
Node.spawn(:"node_two@Mufasa", func)
```

The response is a `PID` and the `node name`.

The PID contents (EG: **PID<9964.106.0>**) are broken down as follows:
  - The first field in a PID is the node number. '0' on a local node, '9964' on our remote node in our example.
  - The last two digits are the low and high bits of the process ID.

The process was created on node_one, so it inherits its process hierarchy from node_one. Part of that hierarchy is sth caled the `group leader` which, among other things, determines where IO.puts sends its output.

We start on `node_one`, run a process on `node_two` and the output appears on `node_one`.

## Nodes, Cookies, and Security

Before a node will let another connect, it checks that the remote node has permission. It does that by comparing the node's `cookie` with its own cookie. As an admin of a distributed ELixir system, you need to create a cookie and then make sure all nodes use it.

A cookie can be passed to an iex session through the `--cookie` flag, example:
```
iex --sname cookie --cookie soco
```

When Erlang starts, it looks for an `.erlang.cookie` file in your home dir, if that file does not exist, Erlang creates it and stores a random string in it. It uses it as the cookie for any node that the user starts.

> When connecting nodes over a public network - the cookie is transmitted in plain text.

## Naming Your Processes

If you want to register a callback process on one node and an event-generating process on another, just give the callback PID to the generator.

For the callback to find the generator, the generator can register its PID, giving it a name. The callback on the other node can look up the generator name, using the PID that comes back to send messages to it.

## I/Os, PIDs, and Nodes

`Input` and `Output` in the Erlang VM are performed using `I/O servers` - these are simply Erlang process that implement a low-level message interface.

In Elixir you identify an open file or device by the PID of its I/O server.

> Nodes Are the Basis of Distribution
