---
date: "2016-11-21"
draft: false
weight: 20
title: "Lab 02 - Virtual Interfaces"
---
[Click here to find out more about Alta3 Research's Openstack Training](https://alta3.com/courses/openstack)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective

The objective of this lab is to explore virtual interfaces (veth) within Linux. Virtual interfaces are typically used when you are trying to connect two entities via an interface to forward/receive frames. These entities could be containers/bridges/ovs-switch etc. Say you want to connect a docker/lxc container to OVS. You can create a veth pair and push the first interface to the docker/lxc (say, as a phys interface) and push the other interface to OVS. Linux interfaces created with *ip tuntap* (tap) cannot be used to attach to network namespaces, so we are dependent on veth pairs.

### Procedure

0. Close any terminals you currently have open within the remote desktop session.

0. From your remote desktop, open a new terminal session, and move to the student home directory.

    `student@beachhead:/$` `cd`

0. Begin by creating a pair of virtual ethernet interfaces. From this lab forward, we will issue all of our commands with the updated iproute2 toolkit.

    `student@beachhead:~$` `ip link add veth0 type veth peer name veth1`

0. Nothing special is displayed to the screen for issuing the previous command. To see the results, use the *ip link* command. The pair of interfaces we just created should be named veth0 and veth1.

    `student@beachhead:~$` `ip link list`

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
    3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
    4: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
    5: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
      link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
    ```  

0. If you'll notice, both of these interfaces are down. Let's bring up both interfaces. First, turn up veth0.

    `student@beachhead:~$` `ip link set dev veth0 up`

0. Again. Nothing special is displayed. If we want to see the results of our command, we need to issue the *ip link* command once again. Do so, and take notice to the state of the veth0 interface. The state of the interface should now be listed as LOWERLAYERDOWN.

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
        link/ether fa:16:3e:59:a0:b8 brd ff:ff:ff:ff:ff:ff
    3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
        link/ether fa:16:3e:ac:cb:ff brd ff:ff:ff:ff:ff:ff
    4: veth1@veth0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether 52:71:53:d6:3b:c2 brd ff:ff:ff:ff:ff:ff
    5: veth0@veth1: <NO-CARRIER,BROADCAST,MULTICAST,UP,M-DOWN> mtu 1500 qdisc noqueue state LOWERLAYERDOWN mode DEFAULT group default qlen 1000
        link/ether a6:c7:fa:98:86:af brd ff:ff:ff:ff:ff:ff
    ```

0. The state LOWERLAYERDOWN can be resolved to the more desireable state of UP by turning up the interface peered with veth0, veth1. Let's do that now. 

    `student@beachhead:~$` `sudo ip link set dev veth1 up`

0. Check out your work. Both interfaces should be displaying a state of UP.

    `student@beachhead:~$` `ip link list`

    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
    3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
    4: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
      link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
    5: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
      link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
    ```
  
0. Let's apply an IP address to each interface. First to the veth0 interface.

    `student@beachhead:~$` `ip addr add 10.0.0.1/24 dev veth0`

0. There won't be any special print-out from the previous command. Before we review our work, apply an IP address to the veth1 interface.

    `student@beachhead:~$` `ip addr add 10.0.0.2/24 dev veth1`

0. Issuing *ip addr list* will display the same information as *ip link list*, but will also include the L3 IP information.

    `student@beachhead:~$` `ip addr list`
  
    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP group default qlen 1000
        link/ether fa:16:3e:59:a0:b8 brd ff:ff:ff:ff:ff:ff
        inet 172.16.1.4/24 brd 172.16.1.255 scope global ens3
           valid_lft forever preferred_lft forever
        inet6 fe80::f816:3eff:fe59:a0b8/64 scope link
           valid_lft forever preferred_lft forever
    3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether fa:16:3e:ac:cb:ff brd ff:ff:ff:ff:ff:ff
        inet6 fe80::f816:3eff:feac:cbff/64 scope link
           valid_lft forever preferred_lft forever
    4: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether 52:71:53:d6:3b:c2 brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.2/24 scope global veth1
           valid_lft forever preferred_lft forever
        inet6 fe80::5071:53ff:fed6:3bc2/64 scope link
           valid_lft forever preferred_lft forever
    5: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
        link/ether a6:c7:fa:98:86:af brd ff:ff:ff:ff:ff:ff
        inet 10.0.0.1/24 scope global veth0
           valid_lft forever preferred_lft forever
        inet6 fe80::a4c7:faff:fe98:86af/64 scope link
           valid_lft forever preferred_lft forever
    ```

0. Time to examine the results of our work. Issue the following commands, and answer the assocaited questions along the way.

    `student@beachhead:~$` `ip addr show veth0`
  
    ```
    4: veth0@veth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
      link/ether 76:83:83:20:f1:4b brd ff:ff:ff:ff:ff:ff
      inet 10.0.0.1/24 scope global veth0
         valid_lft forever preferred_lft forever
      inet6 fe80::7483:83ff:fe20:f14b/64 scope link 
         valid_lft forever preferred_lft forever
    ```
    
0. Answer the following questions:

    - **Q1: What is the MAC address assigned to the veth0 interface?**
      - A1: The one applied in the example is 76:83:83:20:f1:4b
      
    > What are the mac addresses of our new interfaces?  
    > Where did these mac addresses come from?
    > How can we set these addresses if we wanted to ?

    
  * `student@beachhead:~$` `ip addr show veth1`
  
    ```
    6: veth1@veth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
      link/ether 7a:a9:ff:dd:05:96 brd ff:ff:ff:ff:ff:ff
      inet 10.0.0.2/24 scope global veth1
         valid_lft forever preferred_lft forever
      inet6 fe80::78a9:ffff:fedd:596/64 scope link 
         valid_lft forever preferred_lft forever
    ``` 

0. Ping from one veth interface to the other

  > Why wont this work?

  * `student@beachhead:~$` `ping -I veth1 10.0.0.1`
  * `student@beachhead:~$` `ping -I veth0 10.0.0.2`

0. Examine the routing table and run tcpdump whit the previous commands

  > When we force the ICMP packet destine for 10.0.0.2 on veth0 

  * `student@beachhead:~$` `ping -I veth1 10.0.0.1 > /dev/null &` _ping in the background_
  * `student@beachhead:~$` `sudo tcpdump -i veth1` _Ctrl^C to exit_
  * `student@beachhead:~$` `fg` _bring the ping back into the foreground and press Ctrl^C to quit_
  * `student@beachhead:~$` `ip route`
  
    ```
    default via 172.16.1.1 dev ens3 
    10.0.0.0/24 dev veth0  proto kernel  scope link  src 10.0.0.1 
    10.0.0.0/24 dev veth1  proto kernel  scope link  src 10.0.0.2 
    169.254.169.254 via 172.16.1.1 dev ens3 
    172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.4 
    ```
  
  * `student@beachhead:~$` `sudo ip route del 10.0.0.0/24 dev veth0`
  * `student@beachhead:~$` `sudo ip route del 10.0.0.0/24 dev veth1`
  * `student@beachhead:~$` `sudo ip route add 10.0.0.0/24 via 10.0.0.1 dev veth0`
  * `student@beachhead:~$` `ip route`

0. Remove all the virtual interfaces (back to scratch)

  > Why do we only need to run the delete once?

  * `student@beachhead:~$` `sudo ip link del veth0`
  * `student@beachhead:~$` `ip link list`
  
    ```
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
        link/ether fa:16:3e:6f:47:52 brd ff:ff:ff:ff:ff:ff
    3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
        link/ether 08:00:27:75:2a:67 brd ff:ff:ff:ff:ff:ff
    ```

