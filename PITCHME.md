Erlang, Elixir, BEAM, and OTP
---

# Introduction
* Erlang is a functional programming language
* Developed in the 1980s at Ericsson for telecom applications
* First commercial project launched in 1994
* AXD301 switch with 1.7 million lines of Erlang code
  - Reported 99.9999999% availability (~31 ms downtime per year)
---
# Telephony requirements for Erlang
* Telecom applications are highly concurrent by nature
  - A single switch handles 10s to 100s of thousands of simultaneous transactions
* Switching networks are distributed
* Must have as close to 100% uptime as possible
  - Can't crash without automatic recovery
  - Software must be changeable on the fly
  - A network must be able to tolerate failure of individual switches
* Soft real-time guarantees
