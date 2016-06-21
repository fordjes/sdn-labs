# Linux Networking 

## iproute2 vs net-tools

iproute2 is the collection of command-line tools used for interfacing with various networking components in the Linux kernel.
These commands provide a full feature set compatibility with the legacy commands (net-tools) that many linux users may be familar with.

| Legacy | Obsoleted by | Use case |  
|---|---|---|  
|ifconfig | ip addr, ip link, ip -s | Address and link config|  
|route | ip route | Routing tables|  
|arp | ip neigh | Neighbors|  
|iptunnel | ip tunnel | Tunnels|  
|nameif | ifrename | Renamen network interfaces|  
|ipmaddr | ip maddr | Multicast|  
|netstat | ip -s, ss, ip route | Network statistics|  


### 1. Using iproute2 commands

Run the both commands and find the desired information.
The goal is to see that all the commmand we are familar with are represented in the new iproute2 commands.

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

#### EC2
EC2 actually still clout-init's to eth0 so this section might be useful, or not.

#### Exploring new interface names


#### Resources

* [iproute2 wiki](https://en.wikipedia.org/wiki/Iproute2)
* [net-tools deprecated - mailing list](https://lists.debian.org/debian-devel/2009/03/msg00780.html)
* [iproute2 cheatsheet](http://baturin.org/docs/iproute2/)
* [ip route management](http://linux-ip.net/html/tools-ip-route.html)
