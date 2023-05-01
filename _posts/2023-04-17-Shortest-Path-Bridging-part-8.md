**Operating the backbone in SPB**

SPB usually "just works", but hey - We all know that this is supposed to be true for a lot of things. Oddly enough, this seems to never be  applicable when YOU try do to something, right? Having a grasp of some basic operational commands and being able to validate the SPB config will take you a long way, and will make it easy to correct simple mistakes. This isn't by any means a comprehensive guide to troubleshooting, just some basic operational commands. 

Let's start with the backbone:

In an ALE switch, you navigate around the AOS the same way as in most other brands - You use what you know, and use "?" to see what more applies. You can choose to take a look at the configuration (like show run commands) or what is running right now. In this case, we know that SPB is what we look for, so let's start what's running right now:

```
BRANCH-USR-Core:: show spb ?
                       ^
                       ISIS IPVPN6 IPVPN
 (Shortest Path Bridging Command Set)
```

Ok, SPB is based on ISIS, so let's add that:

```
Branch-USR-Core:: show spb isis ?
                            ^
                            UNICAST-TABLE SPF SERVICES
                            RAPID-LSP-CONVERGE-TABLE RAPID-LSP-CONVERGE-INFO
                            NODES MULTICAST-TABLE MULTICAST-SOURCES-SPF
                            MULTICAST-SOURCES INGRESS-MAC-FILTER INTERFACE
                            INFO DATABASE BVLANS ADJACENCY
 (Shortest Path Bridging Command Set)
```

Since we're checking out the backbone currently, let's focus on the ones that obviously are linked to the backbone functions: INTERFACE, NODES, ADJACENCY, and BVLANS:

```
BRANCH-USR-Core:: show spb isis interface
SPB ISIS Interfaces:
                                    Oper   Admin   Link      Hello   Hello  Circ
 Interface       Level   CircID     state  state   Metric    Intvl   Mult   Type
---------------+-------+----------+------+-------+---------+-------+------+------------
 1/1/47          L1      1          DOWN   UP      4000      9       3       Pt-to-Pt
 1/1/48          L1      2          UP     UP      4000      9       3       Pt-to-Pt
 1/1/49A         L1      3          DOWN   UP      10        9       3       Pt-to-Pt
 1/1/50A         L1      4          UP     UP      10        9       3       Pt-to-Pt
 1/1/51A         L1      5          DOWN   UP      1000      9       3       Pt-to-Pt
```

Here, we can see all interfaces currently configured for the SPB backbone, L1 is the type for SPB-ISIS interfaces, we see the circuit-id, and that some are down, some are up, but all are enabled. We also see the link metric, and that hello messages are sent every 9 seconds, and adjacency is declared lost if 3 checks in a row fails. The circuit type is point-to-point.

```
BRANCH-USR-Core:: show spb isis nodes
SPB ISIS Nodes:
 System Name      System Id      SourceID BridgePriority
--------------------+---------------+--------+---------------
 BCB-1            9424.e14e.4945  0xe4945  32768 (0x8000)
 BCB-2            9424.e14e.4999  0xe4999  32768 (0x8000)
 BEB-1            9424.e14f.4193  0xf4193  32768 (0x8000)
 BEB-2            9424.e14f.43c3  0xf43c3  32768 (0x8000)
 Customer-1       9424.e15a.e6a1  0xae6a1  32768 (0x8000)
 Customer-2       9424.e164.b39a  0x4b39a  32768 (0x8000)
 Customer-3       9424.e164.b3c4  0x4b3c4  32768 (0x8000)
 Customer-4       9424.e164.bb34  0x4bb34  32768 (0x8000)
 BRANCH-USR-Core  9424.e17c.3081  0xc3081  32768 (0x8000)
 Customer-5       9424.e17c.3135  0xc3135  32768 (0x8000)
```

The above command shows us all the SPB nodes in total that the BRANCH-USR-Core has discovered, including itself.
System name is what it is, system Id is the unique node-id in the SPB domain (BMAC).
SourceID is a 20-bit identifier used for mapping the origin of BUM traffic (only relevant in tandem multicast mode, check out this post about multicast modes).
The bridge priority is used as a tie breaker during path computation. By manipulating the bridge priority, you can create a similar hierarchy as you would with STP, with lowest prio in the core etc.

```
BRANCH-USR-Core:: show spb isis adjacency
SPB ISIS Adjacency:
 System
 (Name : SystemId)             Type   State   Hold    Interface
-------------------------------------+------+-------+------+----------
 Customer-2   : 9424.e164.b39a  L1    UP        22      1/1/48
 BEB-2        : 9424.e14f.43c3  L1    UP        19     1/1/54A
```

The command above shows us all adjacencies that the local node have established to other nodes. Again, we see the node name, system ID (BMAC) as well as the type (always L1 for SPB IS-IS).
We also see state, the hold time (seconds until the adjacency is declared lost if no "hello" message is recieved), and the interface where the adjacency is formed.

```
BRANCH-USR-Core:: show spb isis bvlans
SPB ISIS BVLANS:
                                   Services  Num    Tandem     Root Bridge
 BVLAN   ECT-algorithm     In Use  mapped    ISIDS  Multicast  (Name : MAC Address)
-------+-----------------+-------+---------+------+----------+----------------------------------------
  4000  00-80-c2-01       YES     NO            0  SGMODE
  4001  00-80-c2-02       YES     YES          23  SGMODE
  4002  00-80-c2-03       YES     YES          27  SGMODE
  4003  00-80-c2-04       YES     YES          31  SGMODE
  4004  00-80-c2-05       YES     YES          16  SGMODE

BVLANs:     5
```
The command "show spb isis bvlans" shows us the number of BVLANs, the ECT-algorithm used, if the BVLAN is used, and have any services mapped to it. BVLAN 4000 have no services mapped, since it is used for the IS-IS control plane messaging. The other BVLANs have quite a few services mapped, and all are using SG-mode multicast replication. The root bridge is only visible for the BVLANs that use (*,G)tandem replication.

```
BRANCH-USR-Core:: show spb isis unicast-table
SPB ISIS Unicast MAC Table:
        Destination                              Outbound
 BVLAN  (Name : MAC Address)                     Interface
------+----------------------------------------+-----------
  4000  BCB-1               : 94:24:e1:4e:49:45     1/1/48
  4000  BCB-2               : 94:24:e1:4e:49:99     1/1/48
  4000  BEB-1               : 94:24:e1:4f:41:93     1/1/48
  4000  BEB-2               : 94:24:e1:4f:43:c3    1/1/54A
  4000  Customer-1          : 94:24:e1:5a:e6:a1    1/1/54A
  4000  Customer-2          : 94:24:e1:64:b3:9a     1/1/48
  4000  Customer-3          : 94:24:e1:64:b3:c4     1/1/48
  4000  Customer-4          : 94:24:e1:64:bb:34     1/1/48
  4000  Customer-5          : 94:24:e1:7c:31:35     1/1/48
  4001  BCB-1               : 94:24:e1:4e:49:45     1/1/48
  4001  BCB-2               : 94:24:e1:4e:49:99     1/1/48
  4001  BEB-1               : 94:24:e1:4f:41:93     1/1/48
  4001  BEB-2               : 94:24:e1:4f:43:c3    1/1/54A
  4001  Customer-1          : 94:24:e1:5a:e6:a1    1/1/54A
  4001  Customer-2          : 94:24:e1:64:b3:9a     1/1/48
  4001  Customer-3          : 94:24:e1:64:b3:c4     1/1/48
  4001  Customer-4          : 94:24:e1:64:bb:34     1/1/48
  4001  Customer-5          : 94:24:e1:7c:31:35     1/1/48
  4002  BCB-1               : 94:24:e1:4e:49:45     1/1/48
  4002  BCB-2               : 94:24:e1:4e:49:99     1/1/48
  4002  BEB-1               : 94:24:e1:4f:41:93     1/1/48
  4002  BEB-2               : 94:24:e1:4f:43:c3    1/1/54A
  4002  Customer-1          : 94:24:e1:5a:e6:a1    1/1/54A
  4002  Customer-2          : 94:24:e1:64:b3:9a     1/1/48
  4002  Customer-3          : 94:24:e1:64:b3:c4     1/1/48
  4002  Customer-4          : 94:24:e1:64:bb:34     1/1/48
  4002  Customer-5          : 94:24:e1:7c:31:35     1/1/48
  4003  BCB-1               : 94:24:e1:4e:49:45     1/1/48
  4003  BCB-2               : 94:24:e1:4e:49:99     1/1/48
  4003  BEB-1               : 94:24:e1:4f:41:93     1/1/48
  4003  BEB-2               : 94:24:e1:4f:43:c3    1/1/54A
  4003  Customer-1          : 94:24:e1:5a:e6:a1    1/1/54A
  4003  Customer-2          : 94:24:e1:64:b3:9a     1/1/48
  4003  Customer-3          : 94:24:e1:64:b3:c4     1/1/48
  4003  Customer-4          : 94:24:e1:64:bb:34     1/1/48
  4003  Customer-5          : 94:24:e1:7c:31:35     1/1/48
  4004  BCB-1               : 94:24:e1:4e:49:45     1/1/48
  4004  BCB-2               : 94:24:e1:4e:49:99     1/1/48
  4004  BEB-1               : 94:24:e1:4f:41:93     1/1/48
  4004  BEB-2               : 94:24:e1:4f:43:c3    1/1/54A
  4004  Customer-1          : 94:24:e1:5a:e6:a1    1/1/54A
  4004  Customer-2          : 94:24:e1:64:b3:9a     1/1/48
  4004  Customer-3          : 94:24:e1:64:b3:c4     1/1/48
  4004  Customer-4          : 94:24:e1:64:bb:34     1/1/48
  4004  Customer-5          : 94:24:e1:7c:31:35     1/1/48

MAC Addresses: 45
```

In this output in the example above, we can see the egressing interface when sending traffic to that node. The mac-address is the system-id, so we're only seeing nodes mac-addresses. Paths can and will differ between BVLANs due to their different SPT algorithms.

```
SH-USR-Core:: show spb isis spf bvlan 4000
SPB ISIS Path Table:
 Destination                              Outbound   Next Hop                                SPB     Num
 (Name : BMAC)                            Interface  (Name : BMAC)                           Metric  Hops
----------------------------------------+----------+----------------------------------------+------+------
 BCB-1               : 94:24:e1:4e:49:45     1/1/48  Customer-2          : 94:24:e1:64:b3:9a  12000    3
 BCB-2               : 94:24:e1:4e:49:99     1/1/48  Customer-2          : 94:24:e1:64:b3:9a   8000    2
 BEB-1               : 94:24:e1:4f:41:93     1/1/48  Customer-2          : 94:24:e1:64:b3:9a  16010    5
 BEB-2               : 94:24:e1:4f:43:c3    1/1/54A  BEB-2               : 94:24:e1:4f:43:c3   1000    1
 Customer-1          : 94:24:e1:5a:e6:a1    1/1/54A  BEB-2               : 94:24:e1:4f:43:c3  11000    2
 Customer-2          : 94:24:e1:64:b3:9a     1/1/48  Customer-2          : 94:24:e1:64:b3:9a   4000    1
 Customer-3          : 94:24:e1:64:b3:c4     1/1/48  Customer-2          : 94:24:e1:64:b3:9a  12000    3
 Customer-4          : 94:24:e1:64:bb:34     1/1/48  Customer-2          : 94:24:e1:64:b3:9a   8000    2
 Customer-5          : 94:24:e1:7c:31:35     1/1/48  Customer-2          : 94:24:e1:64:b3:9a  16000    4

SPF Path count: 9
```

This command "show spb isis spf bvlan <bvlan-id>" gives use the resulting paths of the current shortest path forwarding tree calculations in your local node. We see the destination + its BMAC, and the next hop node + its BMAC, the SPB total metric for the path, and the total number of hops until the traffic reaches its destination.

**Checking the configuration**

```
BRANCH-USR-Core:: show configuration snapshot spb
! SPB-ISIS:
spb isis bvlan 4000 ect-id 1
spb isis bvlan 4001 ect-id 2
spb isis bvlan 4002 ect-id 3
spb isis bvlan 4003 ect-id 4
spb isis bvlan 4004 ect-id 5
spb isis control-bvlan 4000
spb isis interface port 1/1/43 metric 4000
spb isis interface port 1/1/45 metric 4000
spb isis interface port 1/1/46 metric 4000
spb isis interface port 1/1/47 metric 10000
spb isis interface port 1/1/48 metric 4000
spb isis interface port 1/1/49A
spb isis interface port 1/1/54A
spb isis interface port 1/1/56A
spb isis admin-state enable
spb ipvpn bind vrf hyperv isid 20000 gateway 100.64.90.1 all-routes
```

The above command is the equivalent to "sho run ..." in a Cisco, you are checking the configuration. Without any options, it will show the configuration in clear text. In the above case, it will show everything that regards SPB. Show configuration snapshot can be used with just about any keyword in the configuration, like:

```
show configuration snapshot interface
show configuration snapshot ip interface
show configuration snapshot ipv6 interface
show configuration snapshot ospf
show configuration snapshot bgp
```

And you can always use the question mark to see all options on every level.

With these seven commands, you can quickly get a very good grasp of the current state of your backbone topology. Now, we will take a look on how to operate services.


[Shortest Path Bridging Part 9 - Operating services in SPB](https://networkundertaker.com/2023/04/30/Shortest-Path-Bridging-part-9.html)

[Start page]({{ '/' | absolute_url }})