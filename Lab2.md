## ホスト名設定

```jsx
configure system name lab2-sr1
```

## カード設定

```jsx
configure card 1 card-type iom-1
configure card 1 level he
configure card 1 mda 1 mda-type me12-100gb-qsfp28
```

```jsx
(gl)[/]
A:admin@sros1# show card

===============================================================================
Card Summary
===============================================================================
Slot      Provisioned Type                         Admin Operational   Comments
              Equipped Type (if different)         State State
-------------------------------------------------------------------------------
1         iom-1:he                                 up    up
A         cpm-1                                    up    up/active
===============================================================================

(gl)[/]
A:admin@sros1# show mda

===============================================================================
MDA Summary
===============================================================================
Slot  Mda   Provisioned Type                            Admin     Operational
                Equipped Type (if different)            State     State
-------------------------------------------------------------------------------
1     1     me12-100gb-qsfp28                           up        up
===============================================================================
```

## ポート設定

```jsx
configure port 1/1/c1 admin-state enable
configure port 1/1/c1 connector breakout c1-100g
configure port 1/1/c1/1 admin-state enable
configure port 1/1/c1/1 description “To Peering LAN"

configure port 1/1/c2 admin-state enable
configure port 1/1/c2 connector breakout c1-100g
configure port 1/1/c2/1 admin-state enable
configure port 1/1/c2/1 description “To Private LAN"
```

```jsx
(gl)[/]
A:admin@sros1# show port 1

===============================================================================
Ports on Slot 1
===============================================================================
Port          Admin Link Port    Cfg  Oper LAG/ Port Port Port   C/QS/S/XFP/
Id            State      State   MTU  MTU  Bndl Mode Encp Type   MDIMDX
-------------------------------------------------------------------------------
1/1/c1        Up         Link Up                          conn   100GBASE-LR4*
1/1/c1/1      Up    Yes  Up      9212 9212    - netw null cgige
1/1/c2        Up         Link Up                          conn   100GBASE-LR4*
1/1/c2/1      Up    Yes  Up      9212 9212    - netw null cgige
1/1/c3        Down       Down                             conn   100GBASE-LR4*
1/1/c4        Down       Down                             conn   100GBASE-LR4*
1/1/c5        Down       Down                             conn   100GBASE-LR4*
1/1/c6        Down       Down                             conn   100GBASE-LR4*
1/1/c7        Down       Down                             conn   100GBASE-LR4*
1/1/c8        Down       Down                             conn   100GBASE-LR4*
1/1/c9        Down       Down                             conn   100GBASE-LR4*
1/1/c10       Down       Down                             conn   100GBASE-LR4*
1/1/c11       Down       Down                             conn   100GBASE-LR4*
1/1/c12       Down       Down                             conn   100GBASE-LR4*
===============================================================================
* indicates that the corresponいding row element may have been truncated.
```

## ルータインターフェース設定

```bash
configure router "Base" interface "To-Peering-LAN" port 1/1/c1/1
configure router "Base" interface "To-Peering-LAN" ipv4 primary address 192.168.2.1
configure router "Base" interface "To-Peering-LAN" ipv4 primary prefix-length 24
configure router "Base" interface "To-Peering-LAN" ipv6 address 2000:1:2::1 prefix-length 64
configure router "Base" interface "To-LAN" port 1/1/c2/1
configure router "Base" interface "To-LAN" ipv4 primary address 192.168.10.1
configure router "Base" interface "To-LAN" ipv4 primary prefix-length 30
configure router "Base" interface "To-LAN" ipv6 address 2000:1:10::1 prefix-length 64
configure router "Base" interface "system" ipv4 primary address 10.0.0.1 prefix-length 32
configure router "Base" interface "system" ipv6 address 3000::1 prefix-length 128
```

```jsx
(ex)[/]
A:admin@sros1# /show router interface

===============================================================================
Interface Table (Router: Base)
===============================================================================
Interface-Name                   Adm       Opr(v4/v6)  Mode    Port/SapId
   IP-Address                                                  PfxState
-------------------------------------------------------------------------------
To-LAN                           Up        Up/Up       Network 1/1/c2/1
   192.168.2.1/30                                              n/a
   2000:1:2::1/64                                              PREFERRED
   fe80::5054:ff:fecd:b502/64                                  PREFERRED
To-Peering-LAN                   Up        Up/Up       Network 1/1/c1/1
   192.168.1.1/24                                              n/a
   2000:1:1::1/64                                              PREFERRED
   fe80::5054:ff:fe0b:f301/64                                  PREFERRED
system                           Up        Up/Up       Network system
   10.0.0.1/32                                                 n/a
   2001::1/128                                                 PREFERRED
-------------------------------------------------------------------------------
Interfaces : 3
===============================================================================
```

## NTP設定

```jsx
configure system time zone non-standard name "jst"
configure system time zone non-standard offset "09:00"
configure system time ntp admin-state enable
configure system time ntp server 192.168.1.254 router-instance "Base" prefer true
```

```jsx
(ex)[/]
A:admin@sros1# /show system ntp servers

===============================================================================
NTP Active Associations
===============================================================================
State                     Reference ID    St Type  A  Poll Reach     Offset(ms)
    Router         Remote
-------------------------------------------------------------------------------
reject                    45.76.218.37    3  srvr  -  64   ......YY  1463.483
    Base           192.168.1.254
===============================================================================

===============================================================================
NTP Clients
===============================================================================
vRouter                                                    Time Last Request Rx
    Address
-------------------------------------------------------------------------------
===============================================================================
```

## OSPFv2/OSPFv3 設定

```jsx
configure router "Base" ospf 0 admin-state enable
configure router "Base" ospf 0 area 0.0.0.0 interface "To-LAN" interface-type point-to-point
configure router "Base" ospf 0 area 0.0.0.0 interface "system" passive

configure router "Base" ospf3 0 admin-state enable
configure router "Base" ospf3 area 0.0.0.0 interface "To-LAN" interface-type point-to-point
configure router "Base" ospf3 area 0.0.0.0 interface "system" passive
```

```jsx
(ex)[/]
A:admin@sros1# show router ospf interface

===============================================================================
Rtr Base OSPFv2 Instance 0 Interfaces
===============================================================================
If Name               Area Id         Designated Rtr  Bkup Desig Rtr  Adm  Oper
-------------------------------------------------------------------------------
system                0.0.0.0         0.0.0.0         0.0.0.0         Up   Wait
To-LAN                0.0.0.0         0.0.0.0         0.0.0.0         Up   PToP
-------------------------------------------------------------------------------
No. of OSPF Interfaces: 2
===============================================================================

(ex)[/]
A:admin@sros1# show router ospf neighbor

===============================================================================
Rtr Base OSPFv2 Instance 0 Neighbors
===============================================================================
Interface-Name                   Rtr Id          State      Pri  RetxQ   TTL
   Area-Id
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
No. of Neighbors: 0
===============================================================================
```

```jsx
(ex)[/]
A:admin@sros1# show router "Base" ospf3 interface

===============================================================================
Rtr Base OSPFv3 Instance 0 Interfaces
===============================================================================
If Name               Area Id         Designated Rtr  Bkup Desig Rtr  Adm  Oper
-------------------------------------------------------------------------------
system                0.0.0.0         0.0.0.0         0.0.0.0         Up   Wait
To-LAN                0.0.0.0         0.0.0.0         0.0.0.0         Up   PToP
-------------------------------------------------------------------------------
No. of OSPF Interfaces: 2
===============================================================================

(ex)[/]
A:admin@sros1# show router "Base" ospf3 neighbor

===============================================================================
Rtr Base OSPFv3 Instance 0 Neighbors
===============================================================================
Interface-Name                   Rtr Id          State      Pri  RetxQ   TTL
   Area-Id
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
No. of Neighbors: 0
===============================================================================
```

## iBGP eBGP設定

```jsx
    /configure router "Base" autonomous-system 65000
    /configure router "Base" bgp router-id 10.0.0.2
    /configure router "Base" bgp ebgp-default-reject-policy import false
    /configure router "Base" bgp ebgp-default-reject-policy export false
    /configure router "Base" bgp multipath max-paths 2
    /configure router "Base" bgp multipath ebgp 2
    /configure router "Base" bgp multipath ibgp 2
    /configure router "Base" bgp group "ddos" peer-as 65000
    /configure router "Base" bgp group "ddos" local-as as-number 65000
    /configure router "Base" bgp group "eBGP-Peering" type external
    /configure router "Base" bgp group "eBGP-Peering" peer-as 65001
    /configure router "Base" bgp group "eBGP-Peering" family ipv4 true
    /configure router "Base" bgp group "eBGP-Peering" family ipv6 true
    /configure router "Base" bgp group "eBGP-Peering" import policy ["any"]
    /configure router "Base" bgp group "eBGP-Peering" export policy ["any"]
    /configure router "Base" bgp group "iBGP-Peering" type internal
    /configure router "Base" bgp group "iBGP-Peering" peer-as 65000
    /configure router "Base" bgp group "iBGP-Peering" family ipv4 true
    /configure router "Base" bgp group "iBGP-Peering" family ipv6 true
    /configure router "Base" bgp group "iBGP-Peering" import policy ["any"]
    /configure router "Base" bgp group "iBGP-Peering" export policy ["any"]
    /configure router "Base" bgp neighbor "10.0.0.2" group "iBGP-Peering"
    /configure router "Base" bgp neighbor "10.0.0.2" local-address 10.0.0.1
    /configure router "Base" bgp neighbor "10.0.0.2" next-hop-self true
    /configure router "Base" bgp neighbor "192.168.2.3" admin-state enable
    /configure router "Base" bgp neighbor "192.168.2.3" group "eBGP-Peering"
    /configure router "Base" bgp neighbor "2000:1:2::3" group "eBGP-Peering"
    /configure router "Base" bgp neighbor "2000:1:2::3" local-address 2000:1:2::1
    /configure router "Base" bgp neighbor "2000:1:10::2" group "iBGP-Peering"
    /configure router "Base" bgp neighbor "2000:1:10::2" local-address 2000:1:10::1
```

```jsx
(ex)[/]
A:admin@sros1# /show router "Base" bgp summary
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
BGP Admin State         : Up          BGP Oper State              : Up
Total Peer Groups       : 1           Total Peers                 : 4
Total VPN Peer Groups   : 0           Total VPN Peers             : 0
Current Internal Groups : 0           Max Internal Groups         : 0
Total BGP Paths         : 19          Total Path Memory           : 6992

Total IPv4 Remote Rts   : 0           Total IPv4 Rem. Active Rts  : 0
Total IPv6 Remote Rts   : 0           Total IPv6 Rem. Active Rts  : 0
Total IPv4 Backup Rts   : 0           Total IPv6 Backup Rts       : 0
Total LblIpv4 Rem Rts   : 0           Total LblIpv4 Rem. Act Rts  : 0
Total LblIpv6 Rem Rts   : 0           Total LblIpv6 Rem. Act Rts  : 0
Total LblIpv4 Bkp Rts   : 0           Total LblIpv6 Bkp Rts       : 0
Total Supressed Rts     : 0           Total Hist. Rts             : 0
Total Decay Rts         : 0

Total VPN-IPv4 Rem. Rts : 0           Total VPN-IPv4 Rem. Act. Rts: 0
Total VPN-IPv6 Rem. Rts : 0           Total VPN-IPv6 Rem. Act. Rts: 0
Total VPN-IPv4 Bkup Rts : 0           Total VPN-IPv6 Bkup Rts     : 0
Total VPN Local Rts     : 0           Total VPN Supp. Rts         : 0
Total VPN Hist. Rts     : 0           Total VPN Decay Rts         : 0

Total MVPN-IPv4 Rem Rts : 0           Total MVPN-IPv4 Rem Act Rts : 0
Total MVPN-IPv6 Rem Rts : 0           Total MVPN-IPv6 Rem Act Rts : 0
Total MDT-SAFI Rem Rts  : 0           Total MDT-SAFI Rem Act Rts  : 0
Total McIPv4 Remote Rts : 0           Total McIPv4 Rem. Active Rts: 0
Total McIPv6 Remote Rts : 0           Total McIPv6 Rem. Active Rts: 0
Total McVpnIPv4 Rem Rts : 0           Total McVpnIPv4 Rem Act Rts : 0
Total McVpnIPv6 Rem Rts : 0           Total McVpnIPv6 Rem Act Rts : 0

Total EVPN Rem Rts      : 0           Total EVPN Rem Act Rts      : 0
Total L2-VPN Rem. Rts   : 0           Total L2VPN Rem. Act. Rts   : 0
Total MSPW Rem Rts      : 0           Total MSPW Rem Act Rts      : 0
Total RouteTgt Rem Rts  : 0           Total RouteTgt Rem Act Rts  : 0
Total FlowIpv4 Rem Rts  : 0           Total FlowIpv4 Rem Act Rts  : 0
Total FlowIpv6 Rem Rts  : 0           Total FlowIpv6 Rem Act Rts  : 0
Total FlowVpnv4 Rem Rts : 0           Total FlowVpnv4 Rem Act Rts : 0
Total FlowVpnv6 Rem Rts : 0           Total FlowVpnv6 Rem Act Rts : 0
Total Link State Rem Rts: 0           Total Link State Rem Act Rts: 0
Total SrPlcyIpv4 Rem Rts: 0           Total SrPlcyIpv4 Rem Act Rts: 0
Total SrPlcyIpv6 Rem Rts: 0           Total SrPlcyIpv6 Rem Act Rts: 0

===============================================================================
BGP Summary
===============================================================================
Legend : D - Dynamic Neighbor
===============================================================================
Neighbor
Description
                   AS PktRcvd InQ  Up/Down   State|Rcv/Act/Sent (Addr Family)
                      PktSent OutQ
-------------------------------------------------------------------------------
192.168.1.2
                65001       0    0 00h05m29s Active
                            0    0
192.168.1.3
                65001       0    0 00h05m29s Active
                            0    0
2000:1:1::2
                65001       0    0 00h00m51s Active
                            0    0
2000:1:1::3
                65001       0    0 00h00m51s Active
                            0    0
-------------------------------------------------------------------------------
```

```jsx
(ex)[/]
A:admin@sros1# show router "Base" bgp neighbor "192.168.1.2" received-routes
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP IPv4 Routes
===============================================================================
Flag  Network                                            LocalPref   MED
      Nexthop (Router)                                   Path-Id     IGP Cost
      As-Path                                                        Label
-------------------------------------------------------------------------------
No Matching Entries Found.
===============================================================================

(ex)[/]
A:admin@sros1# show router "Base" bgp neighbor "192.168.1.3" received-routes
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP IPv4 Routes
===============================================================================
Flag  Network                                            LocalPref   MED
      Nexthop (Router)                                   Path-Id     IGP Cost
      As-Path                                                        Label
-------------------------------------------------------------------------------
No Matching Entries Found.
===============================================================================

(ex)[/]
A:admin@sros1# show router "Base" bgp neighbor "2000:1:1::2" received-routes
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP IPv4 Routes
===============================================================================
Flag  Network                                            LocalPref   MED
      Nexthop (Router)                                   Path-Id     IGP Cost
      As-Path                                                        Label
-------------------------------------------------------------------------------
No Matching Entries Found.
===============================================================================

(ex)[/]
A:admin@sros1# show router "Base" bgp neighbor "2000:1:1::3" received-routes
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP IPv4 Routes
===============================================================================
Flag  Network                                            LocalPref   MED
      Nexthop (Router)                                   Path-Id     IGP Cost
      As-Path                                                        Label
-------------------------------------------------------------------------------
No Matching Entries Found.
===============================================================================
```

```jsx
(ex)[/]
A:admin@sros1# show router fib 1 summary ipv4

===============================================================================
FIB Summary
===============================================================================
                              Active
-------------------------------------------------------------------------------
Static                        0
Direct                        3
Host                          0
BGP                           0
BGP VPN                       0
EVPN-IFF                      0
EVPN-IFL                      0
BGP LABEL                     0
OSPF                          0
OSPFv3                        0
ISIS                          0
RIP                           0
RIP_NG                        0
RIB-API                       0
LDP                           0
Aggregate                     0
ARP-ND                        0
Sub Mgmt                      0
VPN Leak                      0
TMS                           0
NAT                           0
Managed                       0
Periodic                      0
Video                         0
ESM-Broadcast                 0
-------------------------------------------------------------------------------
Total Installed               3
-------------------------------------------------------------------------------
Current Occupancy             1%
Overflow Count                0
Suppressed by Selective FIB   0
Occupancy Threshold Alerts
    Alert Raised 0 Times;
===============================================================================
```

## 経路ポリシー

```jsx
configure policy-options as-path "PEERING" expression "65001"
configure policy-options as-path-group "BOGON" entry 10 expression ".* [64496-64511] .*"
configure { policy-options community "LARGE-PEER" member "65100:100" }
configure { policy-options community "SMALL-PEERS" member "65200:200" }
configure { policy-options prefix-list "AS65xx-prefixes" prefix 10.100.100.0/24 type longer }

configure policy-options policy-statement "EXT-AS-IMPORT" entry-type named
configure policy-options policy-statement "EXT-AS-IMPORT" named-entry "Routes-AS65001" from as-path name "PEERING"
configure policy-options policy-statement "EXT-AS-IMPORT" named-entry "Routes-AS65001" from as-path group "BOGON"

configure policy-options policy-statement "EXT-AS-IMPORT" named-entry "Routes-AS65001" action action-type accept
```

```jsx
(ex)[/]
A:admin@sros1# show router bgp policy-test plcy-or-long-expr "EXT-AS-IMPORT" family ipv4 prefix 0.0.0.0/0 longer neighbor 192.168.1.2
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP IPv4 Routes
===============================================================================
      Network                                            LocalPref   MED
      Nexthop                                            Path-Id     Label
      As-Path
-------------------------------------------------------------------------------
No Matching Entries Found.
===============================================================================
```

## cflowd設定

```jsx
configure cflowd overflow 10
configure cflowd active-flow-timeout 30
configure cflowd inactive-flow-timeout 10
configure cflowd sample-profile 1 sample-rate 100
configure cflowd collector 192.168.2.254 port 5000 description "Neighbor collector"
configure cflowd collector 192.168.2.254 port 5000 autonomous-system-type peer
configure cflowd collector 192.168.2.254 port 5000 version 8
configure cflowd collector 192.168.2.254 port 5000 aggregation protocol-port true
configure cflowd collector 192.168.2.254 port 5000 aggregation source-destination-prefix true
configure cflowd collector 192.168.2.254 port 2000 description "v9collector"
configure cflowd collector 192.168.2.254 port 2000 template-set mpls-ip
configure cflowd collector 192.168.2.254 port 2000 version 9
configure router "Base" interface "To-Peering-LAN" cflowd-parameters sampling unicast type interface
```

```jsx
(ex)[/]
A:admin@sros1# show cflowd status

===============================================================================
Cflowd Status
===============================================================================
Cflowd Admin Status  : Enabled
Cflowd Oper Status   : Enabled
Cflowd Export Mode   : Automatic
Active Flow Timeout  : 30 seconds
Inactive Flow Timeout: 10 seconds
Template Retransmit  : 600 seconds
Cache Size           : 65536 entries
Overflow             : 10%
Aggregation Summary  : protocol-port source-destination-prefix
VRtr If Index Context: global
Analyze GRE          : Disabled
Analyze L2TP         : Disabled
Analyze IPV4overV6   : Disabled

Active Flows         : 0
Dropped Flows        : 0
Total Pkts Rcvd      : 0
Total Pkts Dropped   : 0
Overflow Events      : 0
                                         Raw Flow Counts  Aggregate Flow Counts
Flows Created                                          0                      0
Flows Matched                                          0                      0
Flows Flushed                                          0                      0

==============================================================================
Sample Profile Info
==============================================================================
Profile Id     Sample Rate
------------------------------------------------------------------------------
    1                  100

===============================================================================
Version Info
===============================================================================
Version Status                   Sent                 Open               Errors
-------------------------------------------------------------------------------
    5   Disabled                    0                    0                    0
    8   Enabled                     0                    0                    0
    9   Enabled                     0                    0                    0
   10   Disabled                    0                    0                    0
===============================================================================
```

## RPKI設定

```jsx
configure router "Base" origin-validation rpki-session 192.168.2.5 admin-state enable
configure router "Base" origin-validation rpki-session 192.168.2.5 local-address 192.168.2.1
configure router "Base" origin-validation rpki-session 192.168.2.5 port 8282
configure router "Base" bgp group "eBGP-Peering" origin-validation ipv4 true
configure router "Base" bgp group "eBGP-Peering" origin-validation ipv6 true
configure router "Base" bgp best-path-selection origin-invalid-unusable true
```

```jsx
(ex)[/]
A:admin@sros1# show router origin-validation rpki-session detail

===============================================================================
RPKI Session Information
===============================================================================
IP Address         : 192.168.1.5
Description        : (Not Specified)
-------------------------------------------------------------------------------
Port               : 8282               Oper State         : connect
Uptime             : 0d 00:00:00        Flaps              : 0
Active IPv4 Records: 0                  Active IPv6 Records: 0
Admin State        : Up                 Local Address      : 192.168.1.1
Hold Time          : 600                Refresh Time       : 300
Stale Route Time   : 3600               Connect Retry      : 120
Serial ID          : 0                  Session ID         : 0
===============================================================================
No. of Sessions    : 1
===============================================================================
```

## BGP FlowSpec 設定

```jsx
configure router "Base" bgp group "ddos" peer-as 65000
configure router "Base" bgp group "ddos" local-as as-number 65000
configure router "Base" bgp neighbor "192.168.2.4" group "ddos"
configure router "Base" bgp neighbor "192.168.2.4" type internal
configure router "Base" bgp neighbor "192.168.2.4" peer-as 65000
configure router "Base" bgp neighbor "192.168.2.4" family ipv4 true
configure router "Base" bgp neighbor "192.168.2.4" family ipv6 false
configure router "Base" bgp neighbor "192.168.2.4" family flow-ipv4 true
configure router "Base" bgp neighbor "192.168.2.4" family flow-ipv6 false
configure router "Base" bgp neighbor "192.168.2.4" family ipv4 ipv6 flow-ipv4 flow-ipv6 true
configure filter ip-filter "FSPEC-filter" default-action accept
configure filter ip-filter "FSPEC-filter" filter-id 99
configure filter ip-filter "FSPEC-filter" embed flowspec offset 1000 router-instance "Base"
configure router "Base" interface "To-Peering-LAN" ingress filter ip "FSPEC-filter"
```

```jsx
(ex)[/]
A:admin@sros1# show router bgp routes flow-ipv4
===============================================================================
 BGP Router ID:10.0.0.1         AS:65000       Local AS:65000
===============================================================================
 Legend -
 Status codes  : u - used, s - suppressed, h - history, d - decayed, * - valid
                 l - leaked, x - stale, > - best, b - backup, p - purge
 Origin codes  : i - IGP, e - EGP, ? - incomplete

===============================================================================
BGP FLOW IPV4 Routes
===============================================================================
Flag  Network             Nexthop                 LocalPref       MED
      As-Path                                                     IGP Cost
-------------------------------------------------------------------------------
No Matching Entries Found.
===============================================================================
```

```jsx
(ex)[/]
A:admin@sros1# show filter ip "FSPEC-filter"

===============================================================================
IP Filter
===============================================================================
Filter Id           : 99                           Applied        : Yes
Scope               : Template                     Def. Action    : Forward
Type                : Normal
Shared Policer      : Off
System filter       : Unchained
Radius Ins Pt       : n/a
CrCtl. Ins Pt       : n/a
RadSh. Ins Pt       : n/a
PccRl. Ins Pt       : n/a
Entries             : 0
Description         : (Not Specified)
Filter Name         : FSPEC-filter
-------------------------------------------------------------------------------
Filter Match Criteria : IP
-------------------------------------------------------------------------------
No Match Criteria Found
===============================================================================
```

## uRPF設定

```jsx
configure router "Base" interface "To-Peering-LAN" ipv4 urpf-check mode loose
configure router "Base" interface "To-Peering-LAN" ipv6 urpf-check mode loose
```

## ACL設定

```jsx
configure filter match-list port-list "AS7xx-Ports" port 179
configure filter match-list port-list "AS7xx-Ports" range start 30000 end 64000
configure filter ip-filter "AS700-ALLOW" filter-id 700
configure filter ip-filter "AS700-ALLOW" entry 10 match protocol tcp
configure filter ip-filter "AS700-ALLOW" entry 10 action accept
configure filter ip-filter "AS700-ALLOW" default-action accept
configure filter ipv6-filter "AS-IPv6-ALLOW" filter-id 800
configure filter ipv6-filter "AS-IPv6-ALLOW" entry 10 match next-header tcp
configure filter ipv6-filter "AS-IPv6-ALLOW" entry 10 action accept
configure filter ipv6-filter "AS-IPv6-ALLOW" entry 20 match next-header tcp
configure filter ipv6-filter "AS-IPv6-ALLOW" entry 20 action accept
configure filter ipv6-filter "AS-IPv6-ALLOW" default-action accept
configure router "Base" interface "To-Peering-LAN" ingress filter ip "AS700-ALLOW"
configure router "Base" interface "To-Peering-LAN" ingress filter ipv6 "AS-IPv6-ALLOW"
```

```jsx
(ex)[/]
A:admin@sros1# show filter ip "AS700-ALLOW"

===============================================================================
IP Filter
===============================================================================
Filter Id           : 700                          Applied        : Yes
Scope               : Template                     Def. Action    : Drop
Type                : Normal
Shared Policer      : Off
System filter       : Unchained
Radius Ins Pt       : n/a
CrCtl. Ins Pt       : n/a
RadSh. Ins Pt       : n/a
PccRl. Ins Pt       : n/a
Entries             : 1
Description         : (Not Specified)
Filter Name         : AS700-ALLOW
-------------------------------------------------------------------------------
Filter Match Criteria : IP
-------------------------------------------------------------------------------
Entry               : 10
Description         : (Not Specified)
Log Id              : n/a
Src. IP             : 0.0.0.0/0
Src. Port           : n/a
Dest. IP            : 0.0.0.0/0
Dest. Port          : n/a
Protocol            : tcp
Dscp                : Undefined
ICMP Type           : Undefined                    ICMP Code      : Undefined
Fragment            : Off                          Src Route Opt  : Off
Sampling            : Off                          Int. Sampling  : On
IP-Option           : 0/0                          Multiple Option: Off
Tcp-flag            : (Not Specified)
Option-pres         : Off
Egress PBR          : Disabled
Primary Action      : Forward
Ing. Matches        : 0 pkts
Egr. Matches        : 0 pkts

===============================================================================
```

## ロギング設定

```jsx
/configure log file "1" rollover 10
/configure log file "1" retention 10
/configure log log-id "90" admin-state enable
/configure log log-id "90" source main true
/configure log log-id "90" { destination snmp }
/configure log log-id "91" admin-state enable
/configure log log-id "91" source main true
/configure log log-id "91" destination syslog "syslog_server"
/configure log log-id "92" source main true
/configure log log-id "92" source security true
/configure log log-id "92" source change true
/configure log log-id "92" destination file "1"
/configure log snmp-trap-group "90" trap-target "1" address 192.168.2.254
/configure log snmp-trap-group "90" trap-target "1" version snmpv2c
/configure log snmp-trap-group "90" trap-target "1" notify-community "public"
/configure log syslog "syslog_server" address 192.168.2.254
/configure log syslog "syslog_server" facility local4
/configure log syslog "syslog_server" hostname value "SR1"
```
