Erlang, Elixir, BEAM, OTP, and Phoenix
---

# Introduction
* Erlang is a functional programming language
* Developed in the 1980s at Ericsson for telecom applications
* First commercial project launched in 1994
* AXD301 switch with 1.7 million lines of Erlang code
  - Reported 99.9999999% availability (~31 ms downtime per year)
---
#### Telephony requirements for Erlang
* Telecom applications are highly concurrent by nature
  - A single switch handles 10s to 100s of thousands of simultaneous transactions
* Switching networks are distributed
* Must have as close to 100% uptime as possible
  - Can't crash without automatic recovery
  - Software must be changeable on the fly
  - A network must be able to tolerate failure of individual switches
* Soft real-time guarantees
---
#### High level implementation
* Model as finite state machines that undergo state transitions in response to protocol messages
* An FSM is a lightweight process in memory
* Most processes waiting until event triggers state transition
* Upon receipt, execute small amount of code
  - Change state
  - Send message
  - Wait for next event
* One process should not be able to crash the system, or affect other processes
---
#### Who uses Erlang?
* WhatsApp (2 million connections on a single node)
* Facebook chat backend
* Amazon SimpleDB
* Discord
* League of Legends
---
#### BEAM Architecture
* Schedulers
* Processes
* Distribution
---
#### BEAM: Schedulers
* BEAM has a thread per logical core
* 1 thread = 1 scheduler
* Periodically (20-40k reductions) a master scheduler is chosen
  - Does internal load balancing to utilize all cores efficiently
---
#### BEAM: Schedulers
* A scheduler is a set of 3/4 priority queues
  - Max , high, normal, low
* Queues are implemented as work stealing deques
* An executing process runs for 2k reductions before being rescheduled
  - 2k reductions < ~1ms
---
#### BEAM: Processes
<img src="./memes/process_necromancer.jpg" style="width: 450px;"/>
---
#### BEAM: Processes
* Completely isolated, independent unit of execution
* Have their own stack and heap
* Communicate via message passing
  - This means that data is copied when passed
---
# Processes are fully pre-emptive
* Two closest languages/frameworks are Go and Akka
  - Go can't pre-empt inside of loops
  - Akka uses a cooperative scheduler
---
#### Elixir History
* First released in 2011
* Created by José Valim who was heavily involved in the Ruby and Rails community
* Inspired by his difficulties making Rails more concurrent

---
# Elixir Philosophy
Concurrent programming should not be difficult
---
#### Elixir Overview
* Takes a strong inspiration from Ruby for syntax, and Lisp for structure
  - Syntax is not homoiconic on the surface
  - But elixir allows quoting and exposes its AST for macros
* Dynamically typed, functional language, immutable data structures
* Like JVM languages, can call Erlang libraries with no penalty
* Polymorphism via dynamic dispatch
* No loops, only recursion
---
#### Value types
* Integers
* Floats
* Atoms (constant whose name is its own value)
  - `:my_atom`
  - `true, false, nil` also atoms
* Ranges
  - `start..end`
* Regular expressions
  - `~r{regexp}`
---
#### Collection types
* Tuples
  - `{:ok, 42, "next"}`
* Linked lists
  - ```elixir
  list = [1,2,3,4]
  hd(list) # 1
  tl(list) # [2,3,4]```
* Binaries (strings are UTF-8 encoded binaries)
* Maps
  - `%{key => val, key => val}`
---
#### System types
* PIDs
  - Process ID, references local or remote process.
* Port
  - For interprocess communication with non-erlang systems
---
#### Anonymous functions
* AKA unnamed functions
* Can be passed as arguments (they're also a type)
* Parenthesis optional
```elixir
iex> add = fn (a, b) -> a + b end
Function<12.118419387/2 in :erl_eval.expr/5>
iex> add.(1,2)
3
```
---
#### Named functions
```elixir
defmodule MyModule do
  def say_hello(name) do
    IO.puts "Hello #{name}"
  end
end

iex> MyModule.say_hello("Joe")
Hello Joe
```
---
#### Pattern matching
* Instead of assigning a variable, think of as binding a variable
* `a = 1` doesn't mean we're assigning `1` to `a`
```elixir
iex> a = 1
1
iex> 1 = a
1
```
---
#### Pattern matching
```elixir
iex> [1, a, 3] = [1, 2, 3]
[1, 2, 3]
iex> a
2
```
---
#### Pattern matching
Ignore values with `_`
```elixir
iex> [a, _, _] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```
---
#### Pattern matching
Match on previously bound value with pin operator `^`
```elixir
iex> a =1
1
iex> [^a, 2, 3] = [1, 2, 3]
[1, 2, 3]
```
---
#### Pattern matching
We can pattern match in function signatures
```elixir
defmodule Factorial do
  def fac(0), do: 1
  def fac(x), do: x * fac(x-1)
end
```
---
#### Guards
Can specify conditions on the arguments
```elixir
defmodule Factorial do
  def fac(0), do: 1
  def fac(n) when n > 0 do
    n * fac(n-1)
  end
end
```
---
#### Conditionals
* Erlang has no `if`
* Elixir has `if`, `else`, and `unless` implemented as a macro
* These macros rely on `case` which uses patter matching
---
#### Case
```elixir
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
"This clause will match and bind x to 2 in this clause"
```
---
#### Case with guards
```elixir
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Would match, if guard condition were not satisfied"
...> end
"Will match"
```
---
#### Recursion
```elixir
defmodule Recursion do
  def map([head|tail], f) do
     [f.(head)|map(tail, f)]
  end
end
```
---
#### Recursion: Tail call optimization
```elixir
defmodule RecursionTCO do
  def map(list, f) do
     p_map(list, f, [])
  end

  defp p_map([], _, acc) do
    Enum.reverse(acc)
  end

  defp p_map([head|tail], f, acc) do
    p_map(tail, f, [f.(head)|acc])
  end
end
```
---
#### The pipe operator `|>`
Takes output of previous expression, and uses it as input in the first argument of the supplied function

```elixir
iex> 1..3 |> Enum.map(&to_string/1) |> Enum.each(&IO.puts/1)
1
2
3
:ok
```
---
#### Processes
```elixir
iex> pid = spawn fn -> 1 + 1 end
PID<0.43.0>
iex> Process.alive?(pid)
false
```
`spawn/1` takes a function and returns a pid.
---
#### Processes
We can retrieve the PID of the current process with `self/0`
```elixir
iex> self()
PID<0.41.0>
iex> Process.alive?(self())
true
```
---
#### Send and receive
```elixir
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```
---
#### Sending between processes
```elixir
iex> parent = self()
PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from PID<0.48.0>"
```
---
# Processes are the fundamental unit of concurrency
---
A process can be used for:
  * Holding state to be shared with other processes
  * Executing code in parallel/concurrently
  * Maintaining pools of other processes
---
In order to do any of that, we need the process to stick around
---
But how to do that?

<img src="./memes/thinking-face.png" />
---
<img src="./memes/elmo_recursion.png" />
---
```elixir
defmodule Infinite.Process do
  def loop do
    receive do
      {sender_pid, :hello} when is_pid(sender_pid) ->
        send sender_pid, "world"
        loop
      {sender_pid, _} when is_pid(sender_pid) ->
        send sender_pid, :error
        loop
      _ ->
        loop
    end
  end
end
```
---
```elixir
iex> pid = spawn(Infinite.Process, :loop, [])
PID<0.89.0>
iex> send(pid, {self(), :hello})
{PID<0.81.0>, :hello}
iex> flush()
"world"
:ok
iex> send(pid, {self(), :foo})
{PID<0.81.0>, :foo}
iex> flush()
:error
:ok
```
---
This is such a common design pattern that Erlang provides an abstraction for it, and others, in a common library
---
# OTP
---
* With a message passing model, accessing data is not straightforward
* Requires sending a message to a process
* Message must respect an interface provided by the process
* Then wait for reply, or don't
---
This sounds suspiciously like a client-server interaction
---
OTP provides the following, among others:
* GenServer
* Supervisor
* Application
* ETS
---
#### GenServer
Client-server model is used for resource management where several clients want to share a common resource
* Some kind of data, aka state
* Trigger a task or function
---
* The various abstractions in OTP are called behaviours
* Essentially an interface with required functions to be implemented
* GenServer has 6 required callbacks
* The `use GenServer` macro expands and provides convenient default implementations
---
* The most used callbacks are `handle_call` and `handle_cast`
* `handle_call` is synchronous
  - Sends a message and blocks until reply
  - Must reply with a tuple like `{:reply, response, state}`
* `handle_cast` is asynchronous
  - Fire and forget
  - Replies with `{:noreply, state}`
* Both have the genserver's `state` as arguments
---
```elixir
defmodule Stack do
  use GenServer

  def handle_call(:pop, _from, [h|t]) do
    {:reply, h, t}
  end

  def handle_cast({:push, item}, state) do
    {:noreply, [item | state]}
  end
end
```
---
```elixir
iex> {:ok, pid} = GenServer.start_link(Stack, [:hello])
{:ok, PID<0.88.0>}
iex> GenServer.call(pid, :pop)
:hello
iex> GenServer.cast(pid, {:push, :world})
:ok
iex> GenServer.call(pid, :pop)           
:world
```
---
* This is nice, but a little unwieldy
* It's common practice to implement an API for processes to use when they want to make a call
* This could look something like the following
---
```elixir
def start_link(initial_state) do
  GenServer.start_link(__MODULE__, initial_state)
end

def push(pid, item) do
  GenServer.cast(pid, {:push, item})
end

def pop(pid) do
  GenServer.call(pid, :pop)
end
```
---
## Note:
* `__MODULE__` returns the name of the current module.
* `cast` doesn't guarantee the server received the message
* `handle_info` callback is for handling messages not sent via `GenServer.call`or `GenServer.cast`
---
#### A brief interlude on bugs, exceptions, etc
* Processes are completely isolated and share nothing
* When a process ends, its memory is reclaimed by the VM
* If code in a process causes an exception, it doesn't affect the rest of the code running in the VM
* Processes can send messages upon exiting
* It's not surprising then that a common meme in the Erlang world is "Let it crash"
---
* Does _not_ mean that errors aren't ever handled
* But with pattern matching and message passing, you can be pretty confident that your code is running what you expect
* If not, kill the process and let the calling process handle the `:kill` message
* Process isolation is for handling side effects
---
# Supervisors
---
#### Supervisors
* Specialized processes for monitoring other processes
* Register a list of child processes
* Can restart or stop any of its child processes
* Child processes implement OTP behaviours
* Can register a supervisor as a child
* Leads to supervision trees
  - Just swap `worker` for `supervisor`
---
```elixir
import Supervisor.Spec

children = [
  worker(Stack, [], [name: MyStack])
]
```
then in `iex`
```elixir
{:ok, pid} = Supervisor.start_link(children, strategy: :one_for_one)
```
---
#### Supervisor strategies
* `:one_for_one`: only restart failed child process
* `:one_for_all`: restart all child processes in event of a failure
* `:rest_for_one`: restart failed process and any process started after it
* `:simple_one_for_one`: for dynamically spawned children. Supervisor has one child 'template' that other spawned processes base off of
---
#### Restart values
* `:permanent`: child is always restarted - default
* `:temporary`: child is never restarted
* `:transient`: child is restarted only if abnormal termination
---
# Applications
---
An application is a component implementing some specific functionality, that can be started and stopped as a unit, and which can be re-used in other system.
---
In practice, this is usually a supervision tree with various genserver and supervisor children.
---
```elixir
defmodule MyApp do
  use Application

  def start(_type, _args) do
    MyApp.Supervisor.start_link()
  end
end
```

Can also define a `stop` callback
---
Can include the application in the project dependencies and have it started on load.
---
ETS
---
#### ETS
* Erlang Term Storage
* Robust in-memory store for erlang types
* Organized by tables, which are owned by processes
---
#### Tables types
* `set`: one value per key, keys are unique. default table type
* `ordered_set`: Similar to `set` but ordered by term
* `bag`: many objects per key, but only one instance of each object per key
* `duplicate_bag`: many objects per key, duplicates allowed
---
# ETS tables are accessible by any process

# ETS doesn't have transactions
---
* Therefore, allowing reads from any process is fine, but writes should be done through the controlling process

* If transactions and more robust features are needed, then Mnesia offers that, and ships with the Erlang runtime
---
Phoenix
---
#### Phoenix
* Web MVC framework (though slightly changing in upcoming release)
* Layer built on top of a set of modules
  - Cowboy: the webserver, included as project dependency
  - Plug: specification for web application components and adapters for web servers
    - Like Rack/Sinatra for ruby or `net/http` for go, kinda
  - Ecto: database wrapper and provides integrated query language
---
#### Phoenix logical components
* The Endpoint
* The Router
* Controllers
* Views
* Templates
* Channels
* PubSub
---
#### Endpoint
* Handles all aspects of requests up until the point where the router takes over
* Provides core set of plugs to apply to all requests
* Dispatches requests into designated router
---
#### Router
* Parses incoming requests and dispatches to controller
* Provides helpers to generate route paths or urls to resources
* Defines named pipelines through which requests can pass through
---
#### Controllers
* Provide functions, called actions, to handle requests
* Actions
  - Prepare data and pass into views
  - Invoke rendering via views
  - Perform redirects
---
#### Views
* Render templates
* Act as presentation layer
* Defined helper functions, available in templates, to decorate data for presentation
---
#### Templates
* Just like any other templates in other frameworks
* Are actually compiled functions
---
#### Channels
* Manage websockets
* Analogous to controllers but allow bi-directional communication with persistent connections
* Ships with a javascript client for easy integration in templates or JS libraries like React.
---
#### PubSub
* Underlies the channel layer and allows clients to subscribe to topics
* Abstracts the underlying pubsub adapter for third-party pubsub integration
  - Uses a `:pg2` and `:ets` implementation
---
# Distribution
---
Erlang's big advantage is that it automates away the tedium of writing distributed applications
---
* Connecting an erlang node to another automatically clusters them with all the other connected nodes
* Erlang abstracts RPCs between nodes
  - To spawn a process on another node just use functions from the `Node` library
  - Nodes are atoms
  - `iex> Node.spawn(:"foo@hostname", fn x -> do_the_thing(x))`
  - Returns a pid
* Erlang `:rpc` library for more functionality
---
#### Handling failures
* A configuration file can be used to specify on which node an application is to run on
* Can list other nodes for the application to launch the application if the master node goes down
* Can specify priority orders within nodes so a higher priority node that comes back takes over control
---
#### Example
* master, slave1, slave2 running `myapp`
* master goes down, slave1 takes over
* slave1 goes down, slave2 takes over
* if slave1 comes back up, slave2 keeps running `myapp`
* master comes back up and takes over `myapp`
---
#### `:pg2`

`:pg2` allows us to create, join, and query groups of processes across a cluster
---
* Can acess a group of processes by common name
* Send a message to one, some, or all group members
* If a member terminates, it is automatically removed from the group
---
How does Phoenix Channels use this do distributed PubSub?
---
Simple, have a PubSub server per node join a pg2 group
---
How does pg2 work?
---
#### `pg2` toolbox
* GenServer
* ETS
* `:global`
* Node and Process Monitoring
---
#### `pg2` GenServer
* Each node has a `pg2` server process running
* Central interaction point for `pg2` between each node
---
#### `pg2` ETS
* Store process groups and their memberships
* Reads can happen from any process, but writes are serialized through `pg2` genserver
---
#### `:global`
* `:global` provides a function `trans/2`
  - Acquires a lock across entire cluster using any term as key
  - Then runs a provided function
* Combining `trans/2` with genserver `multi_call/3` allows us to call all processes registered with a given name within the cluster
  - `pg2` ensures that only one process across the entire cluster can modify any given group at a time
---
#### Node and Process monitoring
* `:net_kernal.monitor_nodes/1` allows calling process to register for notifications about nodes connecting and disconnecting from the cluster
* When the `pg2` server receives a notification that a new node has connected, it merges the groups and memberships between itself and the new member's pg2 server.
* `pg2` registers a monitor for each process which joins a group, if monitor reports that the process is down, process membership is removed from the local data
---
#### Observations
* `pg2` uses global locks
  - Modifying group memberships is a globally locked operation
  - Requires multiple network round-trips
  - Can lead to lock overhead in groups with large memberships
* `pg2` is a distributed database
---
#### `pg2` CAP theorem
* `pg2` is AP: available and partition-tolerant
* Cluster partitions will only see groups and memberships from nodes that are reachable
* It is eventually consistent, it automatically heals from partitions
* Easy to distribute thanks to monitors, and conflicts are easily resolved by merging
---
#### Websocket performance
* In 2012 WhatsApp published ~2million tcp connections on 20 core 100gb servers
* In 2015 Elixir/Phoenix team published ~2million websocket connections using Channels using 40 (logical core?) 84gb memory
* 2016 thanks to improvements in Erlang/OTP 18, 2.3 million websocket connections on 20 logical core 64gb Digital Ocean droplet using Docker.
* ~1750ws/gb/core
