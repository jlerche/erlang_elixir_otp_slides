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
![Javascript has ninjas, Erlang has necromancers](memes/process_necromancer.jpg)
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
#### Elixir: History
* First released in 2011
* Created by José Valim who was heavily involved in the Ruby and Rails community
* Inspired by his difficulties making Rails more concurrent
* The language takes a strong inspiration from Ruby for syntax, and Lisp for structure
