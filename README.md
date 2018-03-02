# cldemo-evpn-centralized

This Github repository contains the configuration files necessary for setting up EVPN using Cumulus Linux and FRR on the Reference Topology.  It includes the VXLAN Routing with the Centralized Architecture

This demo is equivalent to the Centralized Architecture Deployment scenario in the [Cumulus VXLAN Routing and EVPN whitepaper](https://cumulusnetworks.com/learn/web-scale-networking-resources/whitepapers/Cumulus-Networks-White-Paper-EVPN.pdf) 

The flatfiles in this repository will set up a BGP unnumbered underlay along with an EVPN overlay between the leafs, spines and exit router (i.e. border leaf).  The servers will have a basic IPv4 with bonding (MLAG) to the leafs.  Server01 and 03 are in one VLAN/VXLAN, and servers 02 and 04 are in a different VLAN/VXLAN.  VTEPS are configured on all leaf switches and the exit switch with all VNIs.  Exit01 and Exit02 are configured as the centralized VXLAN Routers and are running MLAG between them. The SVIs are configured on the Exit routers and  the VRR virtual address is advertised as the default gateway to the Leaf switches via the EVPN BGP extended community. 

The internet router is advertising a default route to the exit router.  The internet router has IP address 172.16.1.1 as a loopback address.


Quickstart: Run the demo
------------------------

Before running this demo, install VirtualBox and Vagrant. The currently supported versions of VirtualBox and Vagrant can be found on the [cldemo-vagrant](https://github.com/CumulusNetworks/cldemo-vagrant) page.  

    git clone https://github.com/cumulusnetworks/cldemo-vagrant
    cd cldemo-vagrant
    vagrant up oob-mgmt-server oob-mgmt-switch 
    vagrant up leaf01 leaf02 leaf03 leaf04 spine01 spine02 exit01 exit02 internet server01 server02 server03 server04
    vagrant ssh oob-mgmt-server
    sudo su - cumulus
    git clone https://github.com/CumulusNetworks/cldemo-evpn-centralized
    cd cldemo-evpn-centralized
    ansible-playbook run-demo.yml
    ssh server01
    ping 172.16.1.1 


Software in Use
------------------------
**Spines, Leafs, Exit and Internet:**
      Cumulus v3.5.0

**On Servers:**
Ubuntu 16.04


## Topology ##

This demo runs on a spine-leaf topology with four attached hosts. The ansible playbook run-demo.yml requires an out-of-band management network that provides access to eth0 on all of the in-band devices. 

![EVPN Demo Topology](https://github.com/CumulusNetworks/cldemo-evpn-centralized/blob/master/cldemo-evpn-centralized.png)

 
## Viewing the Results ##




 

Viewing the Results {WIP}
-------


View the EVPN Routing Table on a Leaf:

   

     cumulus@leaf01:mgmt-vrf:~$ net show bgp evpn route
     BGP table version is 17, local router ID is 10.0.0.11
     Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
     Origin codes: i - IGP, e - EGP, ? - incomplete
     EVPN type-2 prefix: [2]:[ESI]:[EthTag]:[MAClen]:[MAC]:[IPlen]:[IP]
     EVPN type-3 prefix: [3]:[EthTag]:[IPlen]:[OrigIP]
     Network          Next Hop            Metric LocPrf Weight Path
     Route Distinguisher: 10.0.0.11:2
     *> [2]:[0]:[0]:[48]:[44:38:39:00:00:03]
                     10.0.0.112                         32768 i                           
     *> [2]:[0]:[0]:[48]:[44:38:39:00:00:03]:[32]:[10.1.3.101]
                     10.0.0.112                         32768 i
     *> [2]:[0]:[0]:[48]:[44:38:39:00:00:03]:[128]:[fe80::4638:39ff:fe00:3]
                     10.0.0.112                         32768 i
     *> [2]:[0]:[0]:[48]:[46:38:39:00:00:03]
                     10.0.0.112                         32768 i
     *> [2]:[0]:[0]:[48]:[46:38:39:00:00:17]
     *> [3]:[0]:[32]:[10.0.0.112]
                     10.0.0.112                         32768 i
     Route Distinguisher: 10.0.0.11:3
     **-SNIP-**
     Route Distinguisher: 10.0.0.41:2
     *  [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.41                              0 65020 65041 25253 i
     *> [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.41                              0 65020 65041 25253 i
     Route Distinguisher: 10.0.0.42:2
     *  [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.42                              0 65020 65042 25253 i
     *> [5]:[0]:[0]:[0.0.0.0]
                        10.0.0.42                              0 65020 65042 25253 i
                                    
                                   
 





Check the Kernel routing table on the leaf in the VRF:

    cumulus@leaf01:mgmt-vrf:~$ ip route show vrf vrf1
    default  proto bgp  metric 20
    nexthop via 10.0.0.42  dev vlan4001 weight 1 onlink
    nexthop via 10.0.0.41  dev vlan4001 weight 1 onlink
    unreachable default  metric 4278198272
    10.1.3.0/24 dev vlan13  proto kernel  scope link  src 10.1.3.11
    10.1.3.0/24 dev vlan13-v0  proto kernel  scope link  src 10.1.3.1
    10.1.3.103 via 10.0.0.134 dev vlan4001  proto bgp  metric 20 onlink
    10.2.4.0/24 dev vlan24  proto kernel  scope link  src 10.2.4.11
    10.2.4.0/24 dev vlan24-v0  proto kernel  scope link  src 10.2.4.1
    10.2.4.104 via 10.0.0.134 dev vlan4001  proto bgp  metric 20 onlink
   

Ping from Server01 to Server04 on (different VLANs and racks), and ping from server01 to Internet Router

 

               
                               
    cumulus@server01:~$ ping 10.2.4.104
    PING 10.2.4.104 (10.2.4.104) 56(84) bytes of data.
    64 bytes from 10.2.4.104: icmp_seq=1 ttl=62 time=6.56 ms
    64 bytes from 10.2.4.104: icmp_seq=2 ttl=62 time=2.63 ms
    64 bytes from 10.2.4.104: icmp_seq=3 ttl=62 time=2.17 ms
    ^C
    --- 10.2.4.104 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 2.178/3.792/6.563/1.969 ms
    PING 172.16.1.1 (172.16.1.1) 56(84) bytes of data.
    64 bytes from 172.16.1.1: icmp_seq=1 ttl=62 time=2.53 ms
    64 bytes from 172.16.1.1: icmp_seq=2 ttl=62 time=2.92 ms
    









    
    



