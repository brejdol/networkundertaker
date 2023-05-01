**Operating services in SPB**

Below you'll find commands and the output from them, regarding services:

```
BRANCH-USR-Core:: show service spb
Legend: * denotes a dynamic object
SPB Service Info
  SystemId : 9424.e17c.3081,   SrcId : 0xc3081,    SystemName : SH-USR-Core

                            SAP     Bind                    MCast
ServiceId   Adm  Oper Stats Count   Count   Isid      BVlan Mode     (T/R)
-----------+----+----+-----+-------+-------+---------+-----+--------------
521         Up   Up    N    8       4       521       4002  Tandem   (1/1)
522         Up   Up    N    2       4       522       4002  Tandem   (1/1)
523         Up   Up    N    6       4       523       4002  Tandem   (1/1)
524         Up   Up    N    0       4       524       4002  Tandem   (1/1)
525         Up   Up    N    0       4       525       4002  Tandem   (1/1)
901         Up   Up    N    0       4       901       4002  Tandem   (1/1)
902         Up   Up    N    0       4       902       4002  Tandem   (1/1)
1000        Up   Up    N    0       4       1000      4001  Tandem   (1/1)
```
The command "show service spb" (output was truncated) lists all configured spb services in your local node, starting with the service-ID (the locally significant name, not the global isid), the administrative and operational status, if stats are enabled (you can enable more statistics in a service), the SAP count (the number of access ports that is configured). the bind count (how many nodes that are advertising this service), the isid (the globally unique id), which BVLAN the service is mapped to, and the multicast replication mode for the service.
Running the same command on a BCB node would list 0 services, since it just switch the BVLANs.

```
BRANCH-USR-Core:: show spb isis services
Legend: * indicates locally configured ISID
SPB ISIS Services Info:
                      System
    ISID      BVLAN   (Name : BMAC)                           MCAST(T/R)
------------+-------+----------------------------------------+-----------
*     1000     4001   BEB-1               : 94:24:e1:4f:41:93  both
*     1000     4001   BEB-2               : 94:24:e1:4f:43:c3  both
*     1000     4001   Customer-1          : 94:24:e1:5a:e6:a1  both
*     1000     4001   BRANCH-USR-Core     : 94:24:e1:7c:30:81  both
*     1000     4001   Customer-5          : 94:24:e1:7c:31:35  both
```

The output from "show spb isis services" (output was truncated) lists all nodes that advertise the service (i.e: has the service enabled), and also shows the globally unique isid, the BVLAN the service is mapped to, as well as the remode node name and BMAC.
The asterix at the start of the line indicates that the service is also active in your local node, for clarity. The local node is also in the list.

```
BRANCH-USR-Core:: show service spb 1000
SPB Service Detailed Info
  Service Id       : 1000,                 Description      : mgmnt-san          ,
  ISID             : 1000,                 BVlan            : 4001,
  Multicast-Mode   : Tandem,               Tx/Rx Bits       : 1/1,
  Admin Status     : Up,                   Oper Status      : Up,
  Stats Status     : No,                   Vlan Translation : Y,
  Service Type     : SPB,                  Allocation Type  : Static,
  MTU              : 9194,                 VPN IP-MTU       : 1500,
  SAP Count        : 0,                    SDP Bind Count   : 4,
  RemoveIngressTag : No,                   Option           : None,
  Mgmt Change      : 04/27/2023 09:27:34,  Status Change    : 04/27/2023 09:27:34
```
The command "show service spb 1000" gives you more specific info about that particular service. Note the MTU that is lowered from the standard 9216 in an ALE switch due to the SPB underlay and service overhead in the link layer.

```
BRANCH-USR-Core:: show service access
Legend: (~)Internal User Port Loopback (-)ERP Ring

Port       Link  SAP     SAP     Vlan
Id        Status Type    Count   Xlation L2Profile                        Description
---------+------+-------+-------+-------+--------------------------------+---------------------------------
1/1/1     Up     Manual  56      Y       def-access-profile
1/1/2     Up     Manual  55      Y       def-access-profile
1/1/3     Up     Manual  22      Y       def-access-profile
1/1/4     Down   Manual  43      Y       def-access-profile
1/1/5     Up     Manual  1       Y       def-access-profile
1/1/6     Up     Manual  2       Y       def-access-profile
1/1/7     Up     Manual  2       Y       def-access-profile
```

The output from "show service access" (output was truncated) shows us ports, link status, SAP-type (you can also allocate services dynamically, just as with VLANs), SAP count (the amount of services on a port), if vlan-translation is enabled, and the L2 profile (here, you can for example filter out STP and other L2 protocols).

```
BRANCH-USR-Core:: show service spb 1100 ports
Legend: (*)Dyn unicast (+)Remote Mcast (#)Local Mcast (-)ERP Ring
SPB Service 1100 Info
  Admin : Up,        Oper  : Up,     Stats      : N,         Mtu     : 9194,   VlanXlation : Y,
  ISID  : 1100,      BVlan : 4001,   MCast-Mode : Tandem,    Tx/Rx   : 1/1,    RemoveIngTag: N,
  VRF   : Default,                   IPv4 MTU   : 1500,      IPv4 Int: service-1100
  VRF   : Default,                   IPv6 MTU   : 1500,      IPv6 Int: service-1100


                                       Sap Trusted:Priority/         Sap Description /
Identifier             Adm  Oper Stats Sdp SystemId:BVlan   Intf     Sdp SystemName
----------------------+----+----+-----+--------------------+--------+--------------------------------
sap:1/1/1:1100         Up   Up    N           Y:x           1/1/1       -
sap:1/1/2:1100         Up   Up    N           Y:x           1/1/2       -
sap:1/1/32:1100        Up   Down  N           Y:x           1/1/32      -
sap:1/1/33:1100        Up   Up    N           Y:x           1/1/33      -
sdp:32954:1100*        Up   Up    Y    9424.e15a.e6a1:4001  1/1/54A  Customer-1
sdp:33081:1100*        Up   Up    Y    9424.e14e.4945:4001  1/1/48   BCB-1
sdp:33082:1100*        Up   Up    Y    9424.e14e.4999:4001  1/1/48   BCB-2
sdp:33095:1100*        Up   Up    Y    9424.e164.b39a:4001  1/1/48   Customer-2
sdp:33119:1100*        Up   Up    Y    9424.e14f.43c3:4001  1/1/54A  BEB-2
sdp:33134:1100*        Up   Up    Y    9424.e164.b3c4:4001  1/1/48   Customer-3
sdp:33140:1100*        Up   Up    Y    9424.e14f.4193:4001  1/1/48   BEB-1
sdp:33141:1100*        Up   Up    Y    9424.e17c.3135:4001  1/1/48   Customer-5
sdp:33149:1100*        Up   Up    Y    9424.e164.bb34:4001  1/1/48   Customer-4

Total Ports: 15
```

The output from "show service spb 1100 ports" (output was truncated) gives us both local and all remote ports (SDP = service distribution point) for a service. We also see the administrative and operational status, the system-id (BMAC) and the interfaces and names of the remote nodes. SDP ports consists of a randomly generated number:service-id, and always have a "*" after them because the SDP id-number is generated by the SPB control plane.

```

BRANCH-USR-Core:: show service mesh-sdp SPB
Legend: * denotes a dynamic object
SPB Mesh-SDP Info
SvcId    SdpId            Isid      FarEnd SysId:BVlan   Oper Intf     FarEnd SystemName
--------+----------------+---------+--------------------+----+--------+------------------
521      32955:521*       521       9424.e15a.e6a1:4002  Up   1/1/54A  Customer-1
521      33120:521*       521       9424.e14f.43c3:4002  Up   1/1/54A  BEB-2
521      33142:521*       521       9424.e14f.4193:4002  Up   1/1/48   BEB-1
521      33143:521*       521       9424.e17c.3135:4002  Up   1/1/48   Customer-5
522      32955:522*       522       9424.e15a.e6a1:4002  Up   1/1/54A  Customer-1
522      33120:522*       522       9424.e14f.43c3:4002  Up   1/1/54A  BEB-2
522      33142:522*       522       9424.e14f.4193:4002  Up   1/1/48   BEB-1
522      33143:522*       522       9424.e17c.3135:4002  Up   1/1/48   Customer-5
523      32955:523*       523       9424.e15a.e6a1:4002  Up   1/1/54A  Customer-1
523      33120:523*       523       9424.e14f.43c3:4002  Up   1/1/54A  BEB-2
523      33142:523*       523       9424.e14f.4193:4002  Up   1/1/48   BEB-1
523      33143:523*       523       9424.e17c.3135:4002  Up   1/1/48   Customer-5
524      32955:524*       524       9424.e15a.e6a1:4002  Up   1/1/54A  Customer-1
```

With the command "show service mesh-sdp SPB" (output was truncated), we see all remote SDPs for all services in the SPB domain.

**MAC-learning**

A service is the equivalent to a broadcast domain, and when troubleshooting on L2, you usually want to check you mac-address-tables:

```
BCB-1:: show mac-learning domain spb
Legend: Mac Address: * = address not valid,

        Mac Address: & = duplicate static address,

   Domain    Vlan/SrvcId[ISId/vnId]     Mac Address           Type          Operation          Interface
------------+----------------------+-------------------+------------------+-------------+-------------------------
       
 Total number of Valid MAC addresses above = 0
```

Ah, yes, that's right... If you run that command on a backbone core bridge, it will show you zero mac-addresses, since it doesn't learn any customer mac-addresses, it just switches the BVLANs...

```
BRANCH-USR-Core:: show mac-learning domain spb
Legend: Mac Address: * = address not valid,

        Mac Address: & = duplicate static address,

   Domain    Vlan/SrvcId[ISId/vnId]     Mac Address           Type          Operation          Interface
------------+----------------------+-------------------+------------------+-------------+-------------------------
       SPB                1100:1100   00:09:0f:09:0a:01            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   00:09:0f:09:c8:02            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   00:09:0f:09:c8:03            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   2c:fa:a2:40:85:cf            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   e8:e7:32:51:71:32            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   fc:aa:14:e7:28:fe            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   fc:aa:14:e7:36:e4            dynamic    servicing            sap:1/1/1:1100
       SPB                1100:1100   2c:fa:a2:0a:d9:04            dynamic    servicing            sap:1/1/2:1100
       SPB                1100:1100   2c:fa:a2:43:e6:48            dynamic    servicing            sap:1/1/2:1100
       SPB                1100:1100   e8:e7:32:51:49:34            dynamic    servicing            sap:1/1/2:1100
       SPB                1100:1100   e8:e7:32:85:95:8b            dynamic    servicing            sap:1/1/2:1100
       SPB                1100:1100   96:4d:ed:b9:08:16            dynamic    servicing           sap:1/1/33:1100
       SPB                1100:1100   2c:fa:a2:57:b2:71            dynamic    servicing            sdp:32954:1100
       SPB                1100:1100   70:4c:a5:f6:dc:52            dynamic    servicing            sdp:32954:1100
       SPB                1100:1100   72:5d:77:2a:7f:28            dynamic    servicing            sdp:32954:1100
```

But if you do that in a BEB instead, you will see all mac-addresses, both local and remote. If you want to check a certain service, use the "show mac-learning domain spb isid XXXXXX" instead.

Next up, we will loop the living daylights out of SPB.

[Shortest Path Bridging Part 10 - Messing up SPB](https://networkundertaker.com/2023/05/01/Shortest-Path-Bridging-part-10.html)

[Start page]({{ '/' | absolute_url }})