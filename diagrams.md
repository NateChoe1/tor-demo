# The problem

* Anonymous internet connections are hard
* In most cases, a client and a server can be correlated by parties other than the client

---

# Normal HTTP

        +----------+           +----------+           +----------+
        |          |    TCP    |          |    TCP    |          |
        |  client  | <-------> | attacker | <-------> |  server  |
        |          |           |          |           |          |
        +----------+           +----------+           +----------+
        
        Knows:                 Knows:                 Knows:
          client id              client id              client id
          server id              server id              server id
          content                content                content

* Man in the middle knows everything
* Server knows everything

---

# HTTPS

        +----------+           +----------+           +----------+
        |          |    TLS    |          |    TLS    |          |
        |  client  | <-------> | attacker | <-------> |  server  |
        |          |           |          |           |          |
        +----------+           +----------+           +----------+
        
        Knows:                 Knows:                 Knows:
          client id              client id              client id
          server id              server id              server id
          content                                       content

* Man in the middle still knows which client is talking to which server
* Server still knows everything

---

# HTTPS + VPN

        +----------+       +----------+       +----------+       +----------+       +----------+
        |          |  TLS  |          |  TLS  |          |  TLS  |          |  TLS  |          |
        |  client  | <---> | attacker | <---> |   VPN    | <---> | attacker | <---> |  server  |
        |          |       |          |       |          |       |          |       |          |
        +----------+       +----------+       +----------+       +----------+       +----------+
        
        Knows:             Knows:             Knows:             Knows:             Knows:
          client id          client id          client id          server id          server id
          server id          vpn id             server id          vpn id             vpn id
          content                                                                     content

* Random attackers on the wire are thwarted
* Server doesn't know the client id
* No given party knows all the details except the client
* This system still places trust on the VPN

---

# HTTPS + 2 VPNs

        +----------+       +----------+       +----------+       +----------+
        |          |  TLS  |          |  TLS  |          |  TLS  |          |
        |  client  | <---> |   VPN    | <---> |   VPN    | <---> |  server  |
        |          |       |          |       |          |       |          |
        +----------+       +----------+       +----------+       +----------+
        
        Knows:             Knows:             Knows:             Knows:
          client id          client id          server id          server id
          server id          next vpn id        prev vpn id        final vpn id
          content                                                  content

* No party knows both the client and the server
* Thanks to encryption, no parties have to be trusted

---

# Tor

* Thousands of relays moving traffic around
* Users form a "circuit" of three relays to route data
* No given relay can correlate a client to a server
* Each user using Tor is indistinguishable

---

# Onion services

* What if the server wants to remain anonymous as well?

---

# Onion services

* Server generates a public/private key pair
* Server selects three relays to act as "introduction points"

        +---------+  +---------+  +---------+
        | intro 1 |  | intro 2 |  | intro 3 |
        +---------+  +---------+  +---------+
             ^            ^            ^
             |------------+------------+
             |       Tor circuit
        +----+----+
        | server  |
        +---------+

---

# Onion services

* Server uploads these relays to a distributed hash table called the directory

        +---------+  +---------+  +---------+  +---------+
        | intro 1 |  | intro 2 |  | intro 3 |  |directory|
        +---------+  +---------+  +---------+  +---------+
             ^            ^            ^            ^
             |------------+------------+            |
             |       Tor circuit                    |
        +----+----+                                 |
        | server  |---------------------------------
        +---------+         Tor circuit

---

# Onion services

* Client obtains the public key and queries the directory for the introduction points

        +---------+           Tor circuit
        | client  |---------------------------------
        +---------+                                 |
                                                    |
                                                    |
                                                    v
        +---------+  +---------+  +---------+  +---------+
        | intro 1 |  | intro 2 |  | intro 3 |  |directory|
        +---------+  +---------+  +---------+  +---------+
             ^            ^            ^
             |------------+------------+
             |       Tor circuit
        +---------+
        | server  |
        +---------+

---

# Onion services

* Client selects a relay to act as the rendezvous

        +---------+                 Tor circuit
        | client  |----------------------------------------------
        +---------+                                              |
                                                                 |
                                                                 |
                                                                 v
        +---------+  +---------+  +---------+  +---------+  +----------+
        | intro 1 |  | intro 2 |  | intro 3 |  |directory|  |rendezvous|
        +---------+  +---------+  +---------+  +---------+  +----------+
             ^            ^            ^
             |------------+------------+
             |       Tor circuit
        +---------+
        | server  |
        +---------+

---

# Onion services

* Client notifies server of the rendezvous through an intro point

        +---------+                 Tor circuit
        | client  |----------------------------------------------
        +---------+                                              |
             | Tor circuit                                       |
              ------------+                                      |
                          v                                      v
        +---------+  +---------+  +---------+  +---------+  +----------+
        | intro 1 |  | intro 2 |  | intro 3 |  |directory|  |rendezvous|
        +---------+  +---------+  +---------+  +---------+  +----------+
             ^            ^            ^
             |------------+------------+
             |       Tor circuit
        +---------+
        | server  |
        +---------+

---

# Onion services

* Server connects to rendezvous point

        +---------+                 Tor circuit
        | client  |----------------------------------------------
        +---------+                                              |
                                                                 |
                                                                 |
                                                                 v
        +---------+  +---------+  +---------+  +---------+  +----------+
        | intro 1 |  | intro 2 |  | intro 3 |  |directory|  |rendezvous|
        +---------+  +---------+  +---------+  +---------+  +----------+
             ^            ^            ^                         ^
             |------------+------------+                         |
             |       Tor circuit                                 |
        +---------+                                              |
        | server  |----------------------------------------------
        +---------+                 Tor circuit

---

# Onion services

* Client and server communicate through the rendezvous

        +---------+                 Tor circuit
        | client  |----------------------------------------------
        +---------+                                              |
                                                                 |
                                                                 |
                                                                 v
        +---------+  +---------+  +---------+  +---------+  +----------+
        | intro 1 |  | intro 2 |  | intro 3 |  |directory|  |rendezvous|
        +---------+  +---------+  +---------+  +---------+  +----------+
             ^            ^            ^                         ^
             |------------+------------+                         |
             |       Tor circuit                                 |
        +---------+                                              |
        | server  |----------------------------------------------
        +---------+                 Tor circuit

* Thanks to the rotating rendezvous, no one relay is overloaded
* Because everything happens through Tor circuits, nobody knows anything

---

# Using Tor

* Tor is mainly used through the Tor browser
* Tor browser can be downloaded from [the Tor website](https://www.torproject.org/)
* If that website is censored, there's a [Github repo with releases](https://github.com/torproject/torbrowser-releases/releases)
* For other tasks such as ssh, [Torsocks](https://support.torproject.org/glossary/torsocks/) can route all the network traffic of a program through Tor
