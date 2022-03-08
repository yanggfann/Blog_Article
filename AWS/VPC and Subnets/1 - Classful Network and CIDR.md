# Classful Network and CIDR

## IPv4

Internet Protocol version 4 (**IPv4**) is the fourth version of the **Internet Protocol** (IP). It is one of the **core** protocols of standards-based internetworking methods in the Internet and other packet-switched networks.

IPv4 uses a **32-bit** address space which provides 4,294,967,296 (232) unique addresses, but large blocks are reserved for special networking purposes.

### Address representation

The 32 bits of an IPv4 address are represented in 4 groups of 8 bits each. An example of this representation is 208.130.29.0.(decimalism)

## IPv6
Internet Protocol version 6 (**IPv6**) is the most recent version of the **Internet Protocol** (IP), the communications protocol that provides an identification and location system for computers on networks and routes traffic across the Internet.

IPv6 was developed by the Internet Engineering Task Force (IETF) to deal with the long-anticipated problem of IPv4 address exhaustion, and is intended to **replace IPv4**.

IPv6 uses **128-bit** addresses, theoretically allowing 2128, or approximately 3.4×1038 total addresses.

### Address representation

The 128 bits of an IPv6 address are represented in 8 groups of 16 bits each. An example of this representation is 2001:0db8:0000:0000:0000:ff00:0042:8329.(hexadecimal)

## [Classful Network](https://en.wikipedia.org/wiki/Classful_network)

A **Classful Network** is a **network addressing** architecture used in the **Internet** from 1981 until the introduction of **Classless Inter-Domain Routing** in 1993.

The method divides the IP address space for Internet Protocol version 4 (**IPv4**) into **5** address classes based on the **leading** **4** address bits.

Classes A, B, and C provide **[unicast](https://en.wikipedia.org/wiki/Unicast)** addresses for networks of three different network sizes. Class D is for **[multicast](https://en.wikipedia.org/wiki/Multicast)** networking and the class E address range is reserved for future or experimental purposes.

### Classful Network definition

|Class|Leading bits|Size of network number bit field|Size of rest bit field|Number of networks|Number of hosts for per network|Addresses per network|Total addresses in class|Start address|End address|Default subnet mask in dot-decimal notation|CIDR notation|Scope of using|
|----|----|----|----|----|----|----|----|----|----|----|----|----|
|Class A|0|8|24|128 (2^7)|16,777,214|16,777,216 (2^24)|2,147,483,648 (2^31)|0.0.0.0|127.255.255.255|255.0.0.0|/8|A large network of a lot of hosts|
|Class B|10|16|16|16,384 (2^14)|65,534|65,536 (2^16)|1,073,741,824 (2^30)|128.0.0.0|191.255.255.255|255.255.0.0|/16|A network with a medium number of hosts|
|Class C|110|24|8|2,097,152 (2^21)|254|256 (2^8)|536,870,912 (2^29)|192.0.0.0|223.255.255.255|255.255.255.0|/24|Small Local Area Network|
|Class D (multicast)|1110|not defined|not defined|not defined|not defined|268,435,456 (228)|224.0.0.0|239.255.255.255|not defined|/4|Leave it to the Internet Architecture Board (IAB) to use multicast addresses|
|Class E (reserved)|1111|not defined|not defined|not defined|not defined|268,435,456 (228)|240.0.0.0|255.255.255.255|not defined|not defined|Reserved for search, Internet experimentation and development only|

Number of **networks** = 2^(Size of network number bit field - Leading bits)

Number of hosts for **per network** = (2^Size of rest bit field) - 2 (e.g. (2^24) -2 = 16,777,214)

`0.0.0.0` is a special address, indicating the host on the local network

#### Bit-wise representation
In the following bit-wise representation,

- n indicates a bit used for the network ID.
- H indicates a bit used for the host ID.
- X indicates a bit without a specified purpose.
```
Class A
0.  0.  0.  0   = 00000000.00000000.00000000.00000000
127.255.255.255 = 01111111.11111111.11111111.11111111
                  0nnnnnnn.HHHHHHHH.HHHHHHHH.HHHHHHHH

Class B
128.  0.  0.  0 = 10000000.00000000.00000000.00000000
191.255.255.255 = 10111111.11111111.11111111.11111111
                  10nnnnnn.nnnnnnnn.HHHHHHHH.HHHHHHHH

Class C
192.  0.  0.  0 = 11000000.00000000.00000000.00000000
223.255.255.255 = 11011111.11111111.11111111.11111111
                  110nnnnn.nnnnnnnn.nnnnnnnn.HHHHHHHH

Class D
224.  0.  0.  0 = 11100000.00000000.00000000.00000000
239.255.255.255 = 11101111.11111111.11111111.11111111
                  1110XXXX.XXXXXXXX.XXXXXXXX.XXXXXXXX

Class E
240.  0.  0.  0 = 11110000.00000000.00000000.00000000
255.255.255.255 = 11111111.11111111.11111111.11111111
                  1111XXXX.XXXXXXXX.XXXXXXXX.XXXXXXXX
```

#### Why it needs minus 2 when calculating the number of hosts for per network?

The network address (network number) and broadcast address cannot be assigned to the host for per network.

## [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)

### Why need CIDR?

This architecture change extended the addressing capacity of the Internet but did not prevent **IP address exhaustion**. 

The problem was that many sites needed larger address blocks than a Class C network provided, and therefore they received a Class B block, **which was in most cases much larger than required**. Due to the rapid growth of the Internet, the pool of unassigned Class B addresses (214, or about 16,000) was rapidly being depleted. 

Starting in 1993, classful networking was replaced by Classless Inter-Domain Routing (CIDR), in an attempt to solve this problem.

### Classless Inter-Domain Routing(CIDR)

Classless Inter-Domain Routing is a method for allocating IP addresses and for IP routing.

CIDR encompasses several concepts. It is based on variable-length subnet masking (VLSM) which allows the specification of arbitrary-length prefixes. 

**IPv4 CIDR** blocks are identified using a syntax similar to that of IPv4 addresses: a dotted-decimal address, followed by a slash, then a number **from 0 to 32**, i.e., a.b.c.d/n. 

An IP address is part of a CIDR block and is said to match the CIDR prefix if the initial n bits of the address and the CIDR prefix are the same. An IPv4 address is 32 bits so an **n-bit CIDR** prefix leaves 32 − n bits unmatched, meaning that **2^(32−n) IPv4 addresses** match a given n-bit CIDR prefix.

### IPv4 CIDR blocks

|Address format a.b.c.d/n|Number of hosts for per network = 2^(32-n) - 2|Addresses per network = 2^(32-n)|
|----|----|----|
|a.b.c.d/32|single-host network|1|
|a.b.c.d/31|point-to-point links|2|
|a.b.c.d/30|2|4|
|a.b.c.d/29|6|8|
|a.b.c.d/28|14|16|
|a.b.c.d/27|30|32|
|a.b.c.d/26|62|64|
|a.b.c.d/25|126|128|
|a.b.c.d/24|254|256|
|a.b.c.d/23|510|512|
|a.b.c.d/22|1,022|1,024|
|a.b.c.d/21|2,046|2,048|
|a.b.c.d/20|4,094|4,096|
|a.b.c.d/19|8,190|8,192|
|a.b.c.d/18|16,382|16,384|
|a.b.c.d/17|32,766|32,768|
|a.b.c.d/16|65,534|65,536|
|a.b.c.d/15|131,070|131,072|
|a.b.c.d/14|262,142|262,144|
|a.b.c.d/13|524,286|524,288|
|a.b.c.d/12|1,048,574|1,048,576|
|a.b.c.d/11|2,097,150|2,097,152|
|a.b.c.d/10|4,194,302|4,194,304|
|a.b.c.d/9|8,388,606|8,388,608|
|a.b.c.d/8|16,777,214|16,777,216|
|a.b.c.d/7|33,554,430|33,554,432|
|a.b.c.d/6|67,108,862|67,108,864|
|a.b.c.d/5|134,217,726|134,217,728|
|a.b.c.d/4|268,435,454|268,435,456|
|a.b.c.d/3|536,870,910|536,870,912|
|a.b.c.d/2|1,073,741,822|1,073,741,824|
|a.b.c.d/1|2,147,483,646|2,147,483,648|
|a.b.c.d/0|4,294,967,294|4,294,967,296|

In common usage, the first address in a subnet, **all binary zero in the host identifier**, **is reserved for referring to the network itself**, while the last address, **all binary one in the host identifier**, is **used as a broadcast address for the network**; this reduces the number of addresses available for hosts by 2. 

As a result, a /31 network, with one binary digit in the host identifier, would be unusable, as such a subnet would provide no available host addresses after this reduction. RFC 3021 creates an exception to the "host all ones" and "host all zeros" rules to make **/31 networks usable for point-to-point links**. **/32 addresses (single-host network)** must be accessed by explicit routing rules, as there is no room in such a network for a gateway.

In routed subnets larger than /31 or /32, **the number of available host addresses is usually reduced by two**, namely the largest address, which is reserved as the broadcast address, and the smallest address, which identifies the network itself.
