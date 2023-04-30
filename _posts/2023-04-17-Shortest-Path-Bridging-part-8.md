**Operating and troubleshooting SPB**

SPB usually "just works", but hey - We all know that this is supposed to be true for a lot of things. Oddly enough, this seems to never be  applicable when YOU try do to something, right? Having a grasp of some basic operational commands and being able to validate the SPB config will take you a long way, and will make it easy to correct simple mistakes. This isn't by any means a comprehensive guide to troubleshooting, just some basic operational commands. 

Let's start with the backbone:

In an ALE switch, you navigate around the AOS the same way as in most other brands - You use what you know, and use "?" to see what more applies. You can choose to take a look at the configuration (like sho run commands) or what is running right now. In this case, we know that SPB is what we look for, so let's start with that:

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

Since we're checking out the backbone currently, let's focus on the ones that obviously are linked to the backbone functions: INTERFACE, NODES, BVLANS, and ADJACENCY:

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
 System Name          System Id      SourceID BridgePriority
--------------------+---------------+--------+---------------
 BCB-1       9424.e14e.4945  0xe4945  32768 (0x8000)
 BCB-2       9424.e14e.4999  0xe4999  32768 (0x8000)
 BEB-1       9424.e14f.4193  0xf4193  32768 (0x8000)
 BEB-2       9424.e14f.43c3  0xf43c3  32768 (0x8000)
 Customer-1  9424.e15a.e6a1  0xae6a1  32768 (0x8000)
 Customer-2  9424.e164.b39a  0x4b39a  32768 (0x8000)
 Customer-3  9424.e164.b3c4  0x4b3c4  32768 (0x8000)
 Customer-4  9424.e164.bb34  0x4bb34  32768 (0x8000)
 Customer-5  9424.e17c.3081  0xc3081  32768 (0x8000)
 Customer-6  9424.e17c.3135  0xc3135  32768 (0x8000)
```








Since a service in SPB is the quivalent of a broadcast domain or a VLAN, the same basic troubleshooting measures applies to a SPB environment. You typically would want to be able to check your mac-addresses on a port, or in a service:

```
show mac-learning port 1/1/51A
show mac-learning domain spb serviceid 40001
```

[Shortest Path Bridging Part 8 - Looping SPB](https://networkundertaker.com/2023/04/17/Shortest-Path-Bridging-part-9.html)

[Start page]({{ '/' | absolute_url }})