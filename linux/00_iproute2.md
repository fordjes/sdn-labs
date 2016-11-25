---
date: "2016-11-20"
draft: false
weight: 10
title: "Lab 01 - Linux Networking"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

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

0. Let's begin with a basic legacy command used with **net-tools**. The *ifconfig* command will display all conected network interfaces, the argument *a* will include even those interfaces that are a state of DOWN.

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

 - **Q1: What is the MAC address of ens3?**
    - A1:
 - **Q2: What is ens3?**
    - A2: eth0 - This is a physical interface on the machine you are on (beachhead).
  - Q3: What is ens4?**
    - A3: eth1 - This is a physical interface on the machine you are on (beachhead). This interface will be used to safely execute basic Linux networking commands in this lab.

0. The correlary of ifconfig -a is ip addr show. Run that command now:

    `student@beachhead:~$` `ip addr show`

0. So both commands worked, but to practice good habits, we should use *ip addr show*.

  - **Q1: Why is this *new* way better?**
    - A1: The legacy networking commands are being depricated. In future releases of Linux, it is possibly that legacy commands (such as *ifconfig*) will no longer be supported. There are many limitations with legacy networking commands, such as the inability to create virtual ethernet adapters.

0. Now let's check out the display returned with an updated **iproute2** command. The *ip link* command will display all connected network interfaces. The *ip link show* command will fetch characteristics regarding the link layer devices currently available. It is immaterial to *ip link* whether the device is in use by any other higher layer protocols (L3 - IP). The *ip link* tool provides two verbs: *show* and *set*. The *ip link set* command might be used to: de/activate an interface, change link layer state flags, change the MTU, the name of the interface, and even manipulate the Ethernet broadcast address.

    `student@beachhead:~$` `ip link show`

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
        link/ether fa:16:3e:59:a0:b8 brd ff:ff:ff:ff:ff:ff
    3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether fa:16:3e:ac:cb:ff brd ff:ff:ff:ff:ff:ff
    ```
    
  - **Q1: What is the above MAC address**
    - A1:

0. Now let's try to assign an IPv4 address to a network interface. To start, we'll use the legacy (net-tools) command *ifconfig*.

    `student@beachhead:~$` `sudo ifconfig ens4 172.16.2.10 netmask 255.255.255.0`

0. There won't be any special output if it worked, so run legacy *ifconfig* to confirm that the IP address was applied to the ens4 interface.

  `student@beachhead:~$` `ifconfig`
    
  - **Q1: Was an IP address added to en4?**
    - A1:

0. To remove the IP address using the legacy command, *ifconfig*, use the following command:

  `student@beachhead:~$` `sudo ifconfig ens4 0.0.0.0`

0. There won't be any special output if it worked, so run legacy *ifconfig* to confirm that the IP address was applied to the ens4 interface.

  `student@beachhead:~$` `ifconfig -a`

  - **Q1: Does ens4 still have an IP address?**
    - A1:

0. Let's do the same thing we just did (apply an L3 address to ens4), but with the new iproute2 toolkit. For this we want the command *ip addr add*.

  `student@beachhead:~$` `sudo ip addr add 172.16.2.10/24 dev ens4`  

0. Display the L2 and L3 information with the updated *

  `student@beachhead:~$` `ip addr show`

0. It is possible to assign multiple IPv4 addresses to a network interface. First, we'll use the legacy 
#### 4. Assign Multiple IPv4 addresses to a Network Interface
**net-tools:**  (Not shown, possible using alias, but ugly)

**iproute2:**   
  `$ sudo ip addr add 172.16.2.13/24 dev ens4`  
  `$ sudo ip addr add 172.16.2.14/24 dev ens4`  
  `$ sudo ip addr add 172.16.2.15/24 dev ens4`  
  
#### 5. Remove an IPv4 address from a Network Interface

**iproute2:**  
  `$ sudo ip addr del 172.16.2.13/24 dev ens4`
  `$ sudo ip addr del 172.16.2.13/24 dev ens4`
  `$ sudo ip addr del 172.16.2.13/24 dev ens4`

  - **Q1: Why would you want to run multiple IP addresses on the same interface?**
    - A1: Oftentime services will operate on the same default port. Examples include, Apache & WSGI (OpenStack Keystone), or perhaps two SIP servers on the same machine (5060).
    
0. Assign an IPv6 address to a Network Interface

**net-tools:**  
   `$ sudo ifconfig ens4 inet6 add 2002:0db5:0:f102::1/64`  
   `$ sudo ifconfig ens4 inet6 add 2003:0db5:0:f102::1/64`  

**iproute2:** 
   `$ sudo ip -6 addr add 2002:0db5:0:f102::1/64 dev ens4`  
   `$ sudo ip -6 addr add 2003:0db5:0:f102::1/64 dev ens4`  

#### 8. Show IPv6 address(es) of a Network Interface

**net-tools:**  
   `$ ifconfig ens4`  

**iproute2:** 
   `$ ip -6 addr show dev ens4`  

#### 9. Remove an IPv6 address from a Network Interface
**net-tools:**  
   `$ sudo ifconfig ens4 inet6 del 2002:0db5:0:f102::1/64`  

**iproute2:** 
   `$ sudo ip -6 addr del 2002:0db5:0:f102::1/64 dev ens4`  

#### 10. Change the MAC Address of a Network Interface

0. First a legacy *net-tools* command.

  `student@beachhead:~$` sudo ifconfig ens4 hw ether 08:00:27:75:2a:66`
   
  - **Q1: What is this process called?**
    - A1: MAC Address Override
  - **Q2: Why would you want to modify a MAC address?**
    - A2: Replacing a machine and would prefer to drop in a perfect copy of the one you are replacing (not to bust the ARP table).

**iproute2:** 
   `$ sudo ip link set dev ens4 address 08:00:27:75:2a:67`

0. So now that we've brought manipulation of interfaces a bit more into focus, let's focus on the IP routing table. This next command, which is legacy net-tools, will allow us to view the table.

    `student@beachhead:~$` `route -n`  

    ```
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    0.0.0.0         172.16.1.1      0.0.0.0         UG    0      0        0 ens3
    169.254.169.254 172.16.1.1      255.255.255.255 UGH   0      0        0 ens3
    172.16.1.0      0.0.0.0         255.255.255.0   U     0      0        0 ens3
    ```

0. Network statistics are also available via the legacy net-tools command *netstat*. The following command is popular legacy for returning routing information.

    `student@beachhead:~$` `netstat -rn`  

    ```
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    0.0.0.0         172.16.1.1      0.0.0.0         UG        0 0          0 ens3
    169.254.169.254 172.16.1.1      255.255.255.255 UGH       0 0          0 ens3
    172.16.1.0      0.0.0.0         255.255.255.0   U         0 0          0 ens3
    ```

0. Now examine the information returned by the updated iproute2 *ip route* command.

    `student@beachhead:~$` `ip route show`
    
    ```
    default via 172.16.1.1 dev ens3 onlink
    172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.5
    ```
    
0. Legacy manipulation of this table was done with the net-tools command *route*. Try adding a new route for traffic on the 192.168.1.0/24 network.

    `student@beachhead:~$` `sudo route add -net 192.168.1.0 netmask 255.255.255.0 dev ens4`

0. With legacy manipulation of the routing table, we can remove a route as well.

    `$ sudo route del -net 192.168.1.0 netmask 255.255.255.0 dev ens4`  

0. Using the new iproute2 toolkit command ip route, we can add a new route for traffic going to the 192.168.1.0/24 network (just as we did with the legacy command *route add*).

  `student@beachhead:~$` `sudo ip route add 192.168.1.0/24 dev ens4`

0. Remove the route we just created using the iproute2 toolkit.

  `student@beachhead:~$` `sudo ip route del 192.168.1.0/24 dev ens4`

0. Let's examine some socket (port) statistics (active/listening TCP/UDP sockets). We'll start with the legacy *net-tools* that you might be familiar with. Use the man page for the *netstat* command to answer the following questions.

  `student@beachhead:~$` `man netstat`

  - **Q1: What does the n *flag* do?**
    - A1: This causes only numbers to be shown.
  - **Q2: What does the o *flag* do?**
    - A2: This causes information regarding networking timers to be displayed. 
  - **Q3: What does the t *flag* do?**
    - A3: This causes only information regarding tcp to be displayed.

0. Use the legacy net-tools, *netstat* to display socket information.
  
  `student@beachhead:~$` `netstat -not`

0. The updated analogous *iproute2* command is *ss*. 

  `student@beachhead:~$` `ss -not`  

0. 

**net-tools:**  
   `$ arp -an`  

0. **iproute2:**

  `student@beachhead:~$` `ip neigh`  

  - **Q1: Why doesn't the ip neigh command include flags?**
    - A1: The updated iproute2 command gives us the correct information we want to view the first time.

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


#### Additional Learning / References

The following is a list of pages we thought might be helpful for our students to know about:

* [iproute2 wiki](https://en.wikipedia.org/wiki/Iproute2)

* [net-tools deprecated - mailing list](https://lists.debian.org/debian-devel/2009/03/msg00780.html)

* [iproute2 cheatsheet](http://baturin.org/docs/iproute2/)

* [ip route Management](http://linux-ip.net/html/tools-ip-route.html)

* [Predictable Interface Naming](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)
