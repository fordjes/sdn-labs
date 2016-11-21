---
date: "2016-11-27"
draft: false
weight: 10
title: "Lab 01 - Linux Networking"
---
[Click here to find out more about Alta3 Research's Openstack Training](https://alta3.com/courses/openstack)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

## TODO: This lab is BROKEN
- add prompts
- remove/edit sudo ifconfig eth1 up/down
- this lab doesn't work on beachhead
- (took Hilary about 20 minutes; I got mostly errors)

### Lab Objective
The objective of this lab is to learn how to interface with various networking componets in the Linux kernel. Casual Linux users are likely to recognize many legacy net-tool commands (ifconfig, route, arp, iptunnel, nameif, ipmaddr, netstat). These commands have been obsoleted by the collection of command-line tools known as iproute2.

| Legacy (net-tools)   | Obsoleted by (iproute2) | Use case                   |
|----------------------|-------------------------|----------------------------|
| ifconfig             | ip addr, ip link, ip -s | Address and link config    |
| route                | ip route                | Routing tables             |
| arp                  | ip neigh                | Neighbors                  |
| iptunnel             | ip tunnel               | Tunnels                    |
| nameif               | ifrename                | Rename network interfaces  |
| ipmaddr              | ip maddr                | Multicast                  |
| netstat              | ip -s, ss, ip route     | Network statistics         |

### Procedure

0. From your remote desktop, open a terminal session, and move to the student home directory.

    `student@beachhead:/$` `cd`

#### Display connected network interfaces - ifconfig vs ip link show

0. Let's begin with a basic legacy command used with **net-tools**. The *ifconfig* command will display all conected network interfaces.

    `student@beachhead:~$` `ifconfig -a`

    ```
    ens3  Link encap:Ethernet  HWaddr fa:16:3e:6f:47:52  
          inet addr:172.16.1.4  Bcast:172.16.1.255  Mask:255.255.255.0
          inet6 addr: fe80::f816:3eff:fe6f:4752/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:196654 errors:0 dropped:0 overruns:0 frame:0
          TX packets:144021 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:539590824 (539.5 MB)  TX bytes:11101898 (11.1 MB)

    ens4  Link encap:Ethernet  HWaddr fa:16:3e:2b:a7:95  
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

    lo    Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:174 errors:0 dropped:0 overruns:0 frame:0
          TX packets:174 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:12904 (12.9 KB)  TX bytes:12904 (12.9 KB)
    ```

0. Now let's check out the display returned with an updated **iproute2** command. The *ip link show* command will display all connected network interfaces. The *ip link show* command obsoletes the *ifconfig* command.

    `student@beachhead:~$` `ip link show`

    ```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:59:a0:b8 brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether fa:16:3e:ac:cb:ff brd ff:ff:ff:ff:ff:ff
    ```

#### Assign IPv4 address to a Network Interface - ifconfig vs ip addr

0. 

**net-tools:**  
   `$ sudo ifconfig eth1 10.0.0.1/24`

**iproute2:**  
   `$ sudo ip addr add 10.0.0.1/24 dev eth1`  

#### 4. Assign Multiple IPv4 addresses to a Network Interface
**net-tools:**  (Not shown, possible using alias, but ugly)

**iproute2:**   
  `$ sudo ip addr add 10.0.0.1/24 broadcast 10.0.0.255 dev eth1`  
  `$ sudo ip addr add 10.0.0.2/24 broadcast 10.0.0.255 dev eth1`  
  `$ sudo ip addr add 10.0.0.3/24 broadcast 10.0.0.255 dev eth1`  
  
#### 5. Remove an IPv4 address from a Network Interface

**net-tools:**  
   `$ sudo ifconfig eth1 0`  

**iproute2:**  
   `$ sudo ip addr del 10.0.0.1/24 dev eth1`  

#### 6. Show IPv4 Address(es) of a Network Interface

**net-tools:** Shows only the primary address, not secondary IP addresses.  
   `$ ifconfig eth1`  

**iproute2:**  Shows all IP addresses, primary and secondary.  
   `$ ip addr show dev eth1`  

#### 7. Assign an IPv6 address to a Network Interface

**net-tools:**  
   `$ sudo ifconfig eth1 inet6 add 2002:0db5:0:f102::1/64`  
   `$ sudo ifconfig eth1 inet6 add 2003:0db5:0:f102::1/64`  

**iproute2:** 
   `$ sudo ip -6 addr add 2002:0db5:0:f102::1/64 dev eth1`  
   `$ sudo ip -6 addr add 2003:0db5:0:f102::1/64 dev eth1`  

#### 8. Show IPv6 address(es) of a Network Interface

**net-tools:**  
   `$ ifconfig eth1`  

**iproute2:** 
   `$ ip -6 addr show dev eth1`  

#### 9. Remove an IPv6 address from a Network Interface
**net-tools:**  
   `$ sudo ifconfig eth1 inet6 del 2002:0db5:0:f102::1/64`  

**iproute2:** 
   `$ sudo ip -6 addr del 2002:0db5:0:f102::1/64 dev eth1`  

#### 10. Change the MAC Address of a Network Interface

**net-tools:** deactivate the interface first!  
   `$ sudo ifconfig eth1 hw ether 08:00:27:75:2a:66`  

**iproute2:**  deactivate the interface first!  
   `$ sudo ip link set dev eth1 address 08:00:27:75:2a:67`

#### 11. View the IP Routing Table

**net-tools:**  
   `$ route -n`  
   `$ netstat -rn`  

**iproute2:**  
   `$ ip route show`  

#### 12. Add or Modify a Default Route

**net-tools:**  
   `$ sudo route add default gw 192.168.1.2 eth0`  
   `$ sudo route del default gw 192.168.1.1 eth0`  

**iproute2:**  
   `$ sudo ip route add default via 192.168.1.2 dev eth0`  
   `$ sudo ip route replace default via 192.168.1.2 dev eth0`

#### 13. Add or Remove a Static Route
**With net-tools:**  
   `$ sudo route add -net 172.16.32.0/24 gw 192.168.1.1 dev eth0`  
   `$ sudo route del -net 172.16.32.0/24`  
   
**iproute2:**  
   `$ sudo ip route add 172.16.32.0/24 via 192.168.1.1 dev eth0`  
   `$ sudo ip route del 172.16.32.0/24`

#### 14. View Socket Statistics (active/listening TCP/UDP sockets)

**net-tools:**  
   `$ netstat`  
   `$ netstat -l`
   
**iproute2:**  
   `$ ss`  
   `$ ss -l`  

#### 15. View the ARP Table

**net-tools:**  
   `$ arp -an`  

**iproute2:**  
   `$ ip neigh`  

#### 16. Add or Remove a Static ARP Entry

**net-tools:**  
   `$ sudo arp -s 192.168.1.100 00:0c:29:c0:5a:ef`  
   `$ sudo arp -d 192.168.1.100`  

**iproute2:**  
   `$ sudo ip neigh add 192.168.1.100 lladdr 00:0c:29:c0:5a:ef dev eth0`  
   `$ sudo ip neigh del 192.168.1.100 dev eth0`

All of the future labs will use iproute2 commands.

0. `ifconfig` and `ip address`

  * What is interface names are connected to your instance?
  * What is the ip address of your instance?
  * What is the network mask of the interface?
  * What is the mac address of the interface? 

0. `route` and `ip route`

  * What is the default gateway?
  * What is networks are connected to your instance?

0. `netstat -nt` and `ss -nt`

  * What ports are open on your instance?
  * What services are listening on your instance?

## Predictable interface naming

Ubuntu and RHEL now ship with systemd which manages among other things network interfaces.
Starting with version 197 the way that interface names are generate will change.
 
The goal is to have predicability following these naming mechanisms:

0. Names incorporating Firmware/BIOS provided index numbers for on-board devices (example: eno1)
0. Names incorporating Firmware/BIOS provided PCI Express hotplug slot index numbers (example: ens1)
0. Names incorporating physical/geographical location of the connector of the hardware (example: enp2s0)
0. Names incorporating the interfaces's MAC address (example: enx78e7d1ea46da)
0. Classic, unpredictable kernel-native ethX naming (example: eth0)

The reason for this change was that systems with multiple interfaces were not gaurenteed to get the same names after reboot:

> The classic naming scheme for network interfaces applied by the kernel is to simply assign names beginning with "eth0", "eth1", ... to all interfaces as they are probed by the drivers. As the driver probing is generally not predictable for modern technology this means that as soon as multiple network interfaces are available the assignment of the names "eth0", "eth1" and so on is generally not fixed anymore and it might very well happen that "eth0" on one boot ends up being "eth1" on the next. This can have serious security implications, for example in firewall rules which are coded for certain naming schemes, and which are hence very sensitive to unpredictable changing names.

[Read more](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)

#### Resources

* [iproute2 wiki](https://en.wikipedia.org/wiki/Iproute2)
* [net-tools deprecated - mailing list](https://lists.debian.org/debian-devel/2009/03/msg00780.html)
* [iproute2 cheatsheet](http://baturin.org/docs/iproute2/)
* [ip route management](http://linux-ip.net/html/tools-ip-route.html)
