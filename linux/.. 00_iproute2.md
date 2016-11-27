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
    - A1: The MAC address of ens3 on this example is *fa:16:3e:6f:47:52*, yours will be unique.
  - **Q2: What is ens3?**
    - A2: eth0 - This is a physical interface on the machine you are on (beachhead). We have it named uniquely because of an issue caused by [Predictable Interface Naming](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/).
  - **Q3: What is ens4?**
    - A3: eth1 - This is a physical interface on the machine you are on (beachhead). This interface will be used to safely execute basic Linux networking commands in this lab.

0. The corollary of *ifconfig -a* is *ip addr show*. Run that command now:

  `student@beachhead:~$` `ip addr show`

0. So both commands worked, but to practice good habits, we should use *ip addr show*.

  - **Q1: Did the *ip addr show* command require any flags?**
    - A1: No. It displayed all of the information we wanted (regardless of interfaces being up or down), without any arguments. This is one of many improvements with *iproute2* tools.
  - **Q2: What if I like the old legacy *net-tools* commands? Can I keep using them?**
    - A2: The legacy networking commands are being depricated. In future releases of Linux, it is possibly that legacy commands (such as *ifconfig*) will no longer be supported. There are many limitations with legacy networking commands, such as the inability to create virtual ethernet adapters.
  - **Q3: What is the IPv4 address of ens4?**
    - There are no IP addresses applied.
  - **Q4: What is the IPv4 address of ens3?**
    - A4: 172.16.1.4
    
0. Now let's check out the display returned with an updated **iproute2** command.

  >
  The *ip link* command will display all connected network interfaces. The *ip link show* command will fetch characteristics regarding the link layer devices currently available. It is immaterial to *ip link* whether the device is in use by any other higher layer protocols (L3 - IP). The *ip link* tool provides two verbs: *show* and *set*. The *ip link set* command might be used to: de/activate an interface, change link layer state flags, change the MTU, the name of the interface, and even manipulate the Ethernet broadcast address.

  `student@beachhead:~$` `ip link show`

  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether fa:16:3e:59:a0:b8 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether fa:16:3e:ac:cb:ff brd ff:ff:ff:ff:ff:ff
  ```

0. Answer the following questions:

  - **Q1: What happens if I start messing with the *ens3* interface?**
    - A1: The *ens3* interface provides connectivity to your lab environment. Here's a great analogy. You're standing on a branch called *ens3*. That branch is 100' in the air, hanging off a redwood tree in [Muir Woods National Monument](https://en.wikipedia.org/wiki/Muir_Woods_National_Monument). You don't want to mess with *ens3*.
  - **Q2: What is the MAC on ens4?**
    - A2: The MAC address on the *ens4* interface, on the above example, is *fa:16:3e:ac:cb:ff*. Yours will be unique. 
  - **Q3: Is it okay to tamper with ens4?**
    - A3: Yep! That is why we created *ens4*. All of the labs reference modifications to *ens4* only, so as long as you don't modify any commands in the lab, you'll be fine. 

0. Now let's try to assign an IPv4 address to a network interface. To start, we'll use the legacy (net-tools) command *ifconfig*.

  `student@beachhead:~$` `sudo ifconfig ens4 172.16.2.10 netmask 255.255.255.0`

0. There won't be any special output if it worked, so run legacy *ifconfig* to confirm that the IP address was applied to the ens4 interface.

  `student@beachhead:~$` `ifconfig`
    
  - **Q1: Was an IP address added to en4?**
    - A1: Yes. You should see that the IP address 172.16.2.10 was applied to the interface ens4.

0. To remove the IP address using the legacy command, *ifconfig*, use the following command:

  `student@beachhead:~$` `sudo ifconfig ens4 0.0.0.0`

0. There won't be any special output if it worked, so run legacy *ifconfig* to confirm that the IP address was applied to the ens4 interface.

  `student@beachhead:~$` `ifconfig -a`

  - **Q1: Does ens4 still have an IP address?**
    - A1: No. It appears to have been removed.

0. Let's do the same thing we just did (apply an L3 address to ens4), but with the new iproute2 toolkit. For this we want the command *ip addr add*.

  `student@beachhead:~$` `sudo ip addr add 172.16.2.10/24 dev ens4`  

0. Display the L2 and L3 information with the updated IP address.

  `student@beachhead:~$` `ip addr show`
  
  - **Q1: What IPv4 address has been applied to *ens4*?**
    - A1: 172.16.2.10

0. It is possible to assign multiple IPv4 addresses to a network interface. While it is possible using legacy *net-tools* commands, we're not even going to bother, as it is ugly and complicated. Instead, let's just learn the proper way using the *ip addr add* command from the *iproute2* toolkit. First, apply a second IP address, 172.16.2.13 to the interface *ens4*.

  `student@beachhead:~$` `sudo ip addr add 172.16.2.13/24 dev ens4`  

0. Apply a third IP address, 172.16.2.14 to the interface *ens4*.

  `student@beachhead:~$` `sudo ip addr add 172.16.2.14/24 dev ens4`  

0. Apply a fourth IP address, 172.16.2.15 to the interface *ens4*.

  `student@beachhead:~$` `sudo ip addr add 172.16.2.15/24 dev ens4`
  
0. Display the L2 and L3 information with the updated IP address.

  `student@beachhead:~$` `ip addr show`
  
  - **Q1: What IP addresses have been applied to *ens4*?**
    - A1: 172.16.2.10, 172.16.2.13, 172.16.2.14, 172.16.2.15

0. Remove one of the IP addresses from *ens4*.

  `student@beachhead:~$` `sudo ip addr del 172.16.2.13/24 dev ens4`
  
0. Remove a second IP address *ens4*.

  `student@beachhead:~$` `sudo ip addr del 172.16.2.14/24 dev ens4`

0. Take away the last additional IP address from *ens4*.

  `student@beachhead:~$` `sudo ip addr del 172.16.2.15/24 dev ens4`

  - **Q1: Why would you want to run multiple IP addresses on the same interface?**
    - A1: Oftentime services will operate on the same default port. Examples include, Apache & WSGI (OpenStack Keystone), or perhaps two SIP servers on the same machine (5060).

0. Display the L2 and L3 information with the updated IP address.

  `student@beachhead:~$` `ip addr show`

  - **Q1: What IPv4 address has been applied to *ens4*?**
    - A1: Once again, back to just one IPv4 address, 172.16.2.10

0. Let's try applying an IPv6 address to the **ens4** network interface. First we can use a legacy *net-tools* command, *ifconfig*.

  `student@beachhead:~$` `sudo ifconfig ens4 inet6 add 2002:0db5:0:f102::1/64`  

0. Use the *net-tools* command to display the L3 IPv6 information on dev *ens4*.

  `student@beachhead:~$` `ifconfig ens4`
  
  - **Q1: What IPv6 is applied to the ens4 interface?**
    - A1: 2002:0db5:0:f102::1/64

0. Let's remove an IPv6 address from the **ens4** network interface. Again, use the legacy *net-tools* command, *ifconfig*.

  `student@beachhead:~$` `sudo ifconfig ens4 inet6 del 2002:0db5:0:f102::1/64`  

0. Use the *net-tools* command to display the L3 IPv6 information on dev *ens4*. There should no longer be an IPv6 address applied.

  `student@beachhead:~$` `ifconfig ens4`

0. The following command is the updated *iproute2* command, *ip -6 addr*, which eclipses the *ifconfig* command for manipulating IP address on interfaces.

  `student@beachhead:~$` `sudo ip -6 addr add 2002:0db5:0:f102::1/64 dev ens4`  

0. Retrieve the IPv6 with an *iproute2* command.

  `student@beachhead:~$` `ip -6 addr show dev ens4`  

0. Remove the IPv6 address with an *iproute2* command.

  `student@beachhead:~$` `sudo ip -6 addr del 2002:0db5:0:f102::1/64 dev ens4`  

0. Confirm that the IPv6 address has been removed.

  `student@beachhead:~$` `ip -6 addr show dev ens4`  

0. Try changing the MAC Address of a Network Interface. First a legacy *net-tools* command.

  `student@beachhead:~$` sudo ifconfig ens4 hw ether 08:00:27:75:2a:66`
   
  - **Q1: What is this process called?**
    - A1: MAC Address Override
  - **Q2: Why would you want to modify a MAC address?**
    - A2: Replacing a machine and would prefer to drop in a perfect copy of the one you are replacing (not to bust the ARP table).

0. Use the *net-tools* command to confirm that the MAC address on *ens4* has been changed to 08:00:27:75:2a:66.

  `student@beachhead:~$` `ifconfig ens4`

0. Change the MAC on this address again with the *iproute2* toolkit.

  `student@beachhead:~$` `sudo ip link set dev ens4 address 08:00:27:75:2a:67`

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

0. Use the legacy *net-tools*, *netstat* to display socket information.
  
  `student@beachhead:~$` `netstat -not`

0. The updated analogous *iproute2* command is *ss*. 

  `student@beachhead:~$` `ss -not`  

0. Use legacy *net-tools* to display the arp table.

  `student@beachhead:~$` arp -an`  

0. Use *iproute2* to display the arp table.

  `student@beachhead:~$` `ip neigh`  

  - **Q1: Why doesn't the ip neigh command include flags?**
    - A1: The updated iproute2 command gives us the correct information we want to view the first time.

0. Add a static (-s) ARP entry with legacy *net-tools*.

  `student@beachhead:~$` `sudo arp -s 192.168.1.100 00:0c:29:c0:5a:ef`  

0. Remove/delete (-d) a static ARP entry with legacy *net-tools*.

  `student@beachhead:~$` `sudo arp -d 192.168.1.100`  

0. Add a static ARP entry (add) with updated *iproute2*.

  `student@beachhead:~$` `sudo ip neigh add 192.168.1.100 lladdr 00:0c:29:c0:5a:ef dev eth0`  

0. Remove/delete (del) ARP entry with updated *iproute2*.

  `student@beachhead:~$` `sudo ip neigh del 192.168.1.100 dev eth0`

0. Great job! That's it for this lab. To be clear, all of the future labs will use **iproute2** commands.


#### Additional Learning / References

The following is a list of pages we thought might be helpful for our students to know about:

* [iproute2 Wiki](https://en.wikipedia.org/wiki/Iproute2)

* [net-tools Deprecated - Mailing List](https://lists.debian.org/debian-devel/2009/03/msg00780.html)

* [iproute2 Cheatsheet](http://baturin.org/docs/iproute2/)

* [ip route Management](http://linux-ip.net/html/tools-ip-route.html)

* [Predictable Interface Naming](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)
