
--
date: "2016-11-23"
draft: false
weight: 210
title: "Lab xx - Introduction to Mininet"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to introduce the Mininet CLI. Mininet creates a realistic virtual network, running real kernel, switch and application code, on a single machine (VM, cloud or native), in seconds, with a single command. It is easily manipulated with the Mininet CLI, which makes it a great tool for developing, share, and experiment with OpenFlow and Software-Defined Networking systems.

  >
  Mininet is a network emulator, or perhaps more precisely a network emulation orchestration system. It runs a collection of end-hosts, switches, routers, and links on a single Linux kernel. It uses lightweight virtualization to make a single system look like a complete network, running the same kernel, system, and user code. A Mininet host behaves just like a real machine; you can ssh into it (if you start up sshd and bridge the network to your host) and run arbitrary programs (including anything that is installed on the underlying Linux system.) The programs you run can send packets through what seems like a real Ethernet interface, with a given link speed and delay. Packets get processed by what looks like a real Ethernet switch, router, or middlebox, with a given amount of queueing. When two programs, like an iperf client and server, communicate through Mininet, the measured performance should match that of two (slower) native machines.

  >
  In short, Mininet's virtual hosts, switches, links, and controllers are the real thing – they are just created using software rather than hardware – and for the most part their behavior is similar to discrete hardware elements. It is usually possible to create a Mininet network that resembles a hardware network, or a hardware network that resembles a Mininet network, and to run the same binary code and applications on either platform.

### Procedure

0. From your remote desktop, open a terminal session, and move to the student home directory.

  `student@beachhead:/$` `cd`

0. Let's issue our first mininet command.

  `student@beachhead:~$` `sudo mn`

  ```
  *** No default OpenFlow controller found for default switch!
  *** Falling back to OVS Bridge
  *** Creating network
  *** Adding controller
  *** Adding hosts:
  h1 h2
  *** Adding switches:
  s1
  *** Adding links:
  (h1, s1) (h2, s1)
  *** Configuring hosts
  h1 h2
  *** Starting controller
  
  *** Starting 1 switches
  s1 ...
  *** Starting CLI:
  mininet>
  ```

0. So the CLI has changed. We're now in the mininet environment. Start by issuing *help*.

  `mininet>` `help`
  
  ```
  Documented commands (type help <topic>):
  ========================================
  EOF    gterm  iperfudp  nodes        pingpair      py      switch
  dpctl  help   link      noecho       pingpairfull  quit    time
  dump   intfs  links     pingall      ports         sh      x
  exit   iperf  net       pingallfull  px            source  xterm

  You may also send a command to a node using:
    <node> command {args}
  For example:
    mininet> h1 ifconfig

  The interpreter automatically substitutes IP addresses
  for node names when a node is the first arg, so commands
  like
    mininet> h2 ping h3
  should work.

  Some character-oriented interactive commands require
  noecho:
    mininet> noecho h2 vi foo.py
  However, starting up an xterm/gterm is generally better:
    mininet> xterm h2
  ```

0. We currently have two hosts (h1 and h2), connected by a switch (s1). By issuing the mininet *nodes* command, we can see what kind of architecture has been built for us.

  `mininet>` `nodes`
  
  ```
  available nodes are:
  h1 h2 s1
  ```

0. The mininet *dump* command can be employed to display all available information regardining the host(s).

  `mininet>` `dump`
  
  ```
  <Host h1: h1-eth0:10.0.0.1 pid=9307>
  <Host h2: h2-eth0:10.0.0.2 pid=9310>
  <OVSBridge s1: lo:127.0.0.1,s1-eth1:None,s1-eth2:None pid=9316>
  ```
  
  - **Q1: What IP address was applied to h1 (host1)?**
    - A1: 10.0.0.1
  - **Q2: What IP address was applied to h2 (host2)?**
    - A2: 10.0.0.2

0. The mininet *net* command can be used to list any mininet network connections.

  `mininet>` `net`
  
  ```
  h1 h1-eth0:s1-eth1
  h2 h2-eth0:s1-eth2
  s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
  ```
  
0. We can also issue commands from in these "hosts". Run the command *ip link* on *h1* to determine its L2 configuration.

  `mininet>` `h1 ip link`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: h1-eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
      link/ether 4a:19:3a:99:0a:1a brd ff:ff:ff:ff:ff:ff link-netnsid 0
  ```

0. Instruct h1 (host1) to send two ICMP packets (-c 2) to h2 (host2).

  `mininet>` `h1 ping -c 2 h2`
  
  ```
  PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
  64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.919 ms
  64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.093 ms
  ```
  
0.
    
  `mininet>` `h1 lxterminal &`

  `root@beachhead:~#` `ping -c 2 10.0.0.2` 

  `root@beachhead:~#` `ip link show` 

  `root@beachhead:~#` `exit 

  `mininet>`

  > Given these outputs from commands run inside `h1`, what is the likely way in which mininet has setup this environment for us?

0. Setup tcpdump on h1

  * `mininet>` `h1 lxterminal &`
  * `root@beachhead:~#` `tcpdump -n` 
  
  * `root@beachhead:~#` `ctrl` + `c`
  * `root@beachhead:~#` `exit` 



0. Show connectivity while capturing traffic

  * `mininet>` `h1 ping -c 2 h2`

0. A nice way to determine connectivty within your mininet network is to use the *pingall* command, which does an all-pairs ping.

  `mininet>` `pingall`
  
  ```
  *** Ping: testing ping reachability
  h1 -> h2
  h2 -> h1
  *** Results: 0% dropped (2/2 received)
  ```
0. Inter node performance can be demonstrated with the following two commands. To determine TCP performance, use the *iperf* command:
 
  `mininet>` `iperf`
  
  ```
  *** Iperf: testing TCP bandwidth between h1 and h2
  could not parse iperf output: *** Results: ['', '38.2 Gbits/sec']
  ```

0. UDP performance can be determined using the iperfudp command.

  `mininet>` `iperfudp`
  
  ```
  *** Iperf: testing UDP bandwidth between h1 and h2
  could not parse iperf output: Waiting for server threads to complete. Interrupt again to force quit.
  *** Results: ['10M', '', '10.0 Mbits/sec']
  ```

0. Exit the mininet environment., cleanup and restart mininet with induced latency 

  `mininet>` `exit`
  
0. Always practice good habits when working with mininet. The --clean command ensures that all processes associated with mininet are shutdown and exited properly.

  `student@beachhead:~$` `sudo mn --clean`

0. There is a Linux tool we're going to play with called *traffic control* (tc), however, 'traffic control' is also the name given to the sets of queuing systems and mechanisms by which packets are received and transmitted on a router.

  >
  Traffic control includes deciding which (and whether) packets to accept at what rate on the input of an interface and determining which packets to transmit in what order at what rate on the output of an interface. In the overwhelming majority of situations, traffic control consists of a single queue which collects entering packets and dequeues them as quickly as the hardware (or underlying device) can accept them. This sort of queue is a FIFO.

  >
  Traffic control is the set of tools which allows the user to have granular control over these queues and the queuing mechanisms of a networked device. The power to rearrange traffic flows and packets with these tools is tremendous and can be complicated, but is no substitute for adequate bandwidth.
  
  >
  The term Quality of Service (QoS) is often used as a synonym for traffic control.

0. We can check out the manual page on the traffic control (tc) utility with the following command.

  `student@beachhead:~$` `man tc`

0. The following command will ask mininet to deploy two hosts and a switch, connected with 20 Mbit/sec links, with 20 ms delays.

  `student@beachhead:~$` `sudo mn --link tc,bw=20,delay=20ms`

0. Take a moment to think about our current, very basic, topology. Then, answer the questions below:

  ```
  (h1)-----20ms-----(s1)-----20ms-----(h2)
  ```
  
  - **Q1: When you send a *ping*, you measure the roundtrip delay for an ICMP packet to travel from one host to another. Assuming our current deployment, what will be the reported roundtrip delay?**
    - A1: ~80 ms

0. Let's test that assertion! The following command will ask mininet to send an ICMP packet (ping) from host1 (h1) to host2 (h2), a total of 8 times (-c 8).

  `mininet>` `h1 ping -c 8 h2`

  ```
  PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
  64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=161 ms
  64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=80.3 ms
  64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=80.2 ms
  64 bytes from 10.0.0.2: icmp_seq=4 ttl=64 time=80.3 ms
  64 bytes from 10.0.0.2: icmp_seq=5 ttl=64 time=80.3 ms
  64 bytes from 10.0.0.2: icmp_seq=6 ttl=64 time=80.2 ms
  64 bytes from 10.0.0.2: icmp_seq=7 ttl=64 time=80.3 ms
  64 bytes from 10.0.0.2: icmp_seq=8 ttl=64 time=80.2 ms
  ```

0. Wow cool! There you go, ~80 ms. Okay, exit mininet. Let's deploy some other simple topologies.

  `mininet>` `exit`

0. Make sure mininet has closed all current processes.

  `student@beachhead:~$` `sudo mn --clean`

0. Deploy a 5-host *single* topology.

  `student@beachhead:~$` `sudo mn --topo=single,5`
  
  ```
  *** No default OpenFlow controller found for default switch!
  *** Falling back to OVS Bridge
  *** Creating network
  *** Adding controller
  *** Adding hosts:
  h1 h2 h3 h4 h5
  *** Adding switches:
  s1
  *** Adding links:
  (h1, s1) (h2, s1) (h3, s1) (h4, s1) (h5, s1)
  *** Configuring hosts
  h1 h2 h3 h4 h5
  *** Starting controller

  *** Starting 1 switches
  s1 ...
  *** Starting CLI:
  ```

0. Take a moment to think about our current, very basic, topology. Then, answer the questions below:

  ```
  ------------------(s1)-------------------
  |         |         |         |         |
  h1        h2        h3        h4        h5
  ```
  
  - **Q1: **
    - A1:

0. Use the *pingall* command to demonstrate that there is connectivity betwen all of the hosts.
  
  `mininet>` `pingall`

0. Exit mininet.

  `mininet>` `exit`

0. Shutdown all associated processes with mininet.

  `student@beachhead:~$` `sudo mn --clean` 

0. Deploy a 5-host *linear* topology.

  `student@beachhead:~$` `sudo mn --topo=linear,5`
  
  ```
  *** No default OpenFlow controller found for default switch!
  *** Falling back to OVS Bridge
  *** Creating network
  *** Adding controller
  *** Adding hosts:
  h1 h2 h3 h4 h5
  *** Adding switches:
  s1 s2 s3 s4 s5
  *** Adding links:
  (h1, s1) (h2, s2) (h3, s3) (h4, s4) (h5, s5) (s2, s1) (s3, s2) (s4, s3) (s5, s4)
  *** Configuring hosts
  h1 h2 h3 h4 h5
  *** Starting controller

  *** Starting 5 switches
  s1 s2 s3 s4 s5 ...
  *** Starting CLI:
  ```

0. Take a moment to think about our current, very basic, topology. Then, answer the questions below:

  ```
  (s1)------(s2)------(s3)------(s4)------(s5)
   |         |         |         |         |
   h1        h2        h3        h4        h5
  ```
  
  - **Q1: **
    - A1:

0. Demonstrate connectivity between all of the deployed hosts. 

  `mininet>` `pingall`

0. Exit mininet.

  `mininet>` `exit`

0. Shutdown all associated processes with mininet.

  `student@beachhead:~$` `sudo mn --clean` 

0. Deploy a 5-host *tree* topology.

  `student@beachhead:~$` `sudo mn --topo=tree,5`
  
  ```
  *** No default OpenFlow controller found for default switch!
  *** Falling back to OVS Bridge
  *** Creating network
  *** Adding controller
  *** Adding hosts:
  h1 h2 h3 h4 h5 h6 h7 h8 h9 h10 h11 h12 h13 h14 h15 h16 h17 h18 h19 h20 h21 h22 h23 h24 h25 h26 h27 h28 h29 h30 h31 h32
  *** Adding switches:
  s1 s2 s3 s4 s5 s6 s7 s8 s9 s10 s11 s12 s13 s14 s15 s16 s17 s18 s19 s20 s21 s22 s23 s24 s25 s26 s27 s28 s29 s30 s31
  *** Adding links:
  (s1, s2) (s1, s17) (s2, s3) (s2, s10) (s3, s4) (s3, s7) (s4, s5) (s4, s6) (s5, h1) (s5, h2) (s6, h3) (s6, h4) (s7, s8) (s7, s9) (s8, h5) (s8, h6) (s9, h7) (s9, h8) (s10, s11) (s10, s14) (s11, s12) (s11, s13) (s12, h9) (s12, h10) (s13, h11) (s13, h12) (s14, s15) (s14, s16) (s15, h13) (s15, h14) (s16, h15) (s16, h16) (s17, s18) (s17, s25) (s18, s19) (s18, s22) (s19, s20) (s19, s21) (s20, h17) (s20, h18) (s21, h19) (s21, h20) (s22, s23) (s22, s24) (s23, h21) (s23, h22) (s24, h23) (s24, h24) (s25, s26) (s25, s29) (s26, s27) (s26, s28) (s27, h25) (s27, h26) (s28, h27) (s28, h28) (s29, s30) (s29, s31) (s30, h29) (s30, h30) (s31, h31) (s31, h32)
  *** Configuring hosts
  h1 h2 h3 h4 h5 h6 h7 h8 h9 h10 h11 h12 h13 h14 h15 h16 h17 h18 h19 h20 h21 h22 h23 h24 h25 h26 h27 h28 h29 h30 h31 h32
  *** Starting controller

  *** Starting 31 switches
  s1 s2 s3 s4 s5 s6 s7 s8 s9 s10 s11 s12 s13 s14 s15 s16 s17 s18 s19 s20 s21 s22 s23 s24 s25 s26 s27 s28 s29 s30 s31 ...
  *** Starting CLI:
  ```

0. Demonstrate connectivity between all of the deployed hosts. 

  `mininet>` `pingall`

0. Exit mininet.

  `mininet>` `exit`

0. Shutdown all associated processes with mininet.

  `student@beachhead:~$` `sudo mn --clean` 


#### Additional Learning / References

The following is a list of pages we thought might be helpful for our students to know about:

* [Introduction to Mininet](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet)

* [The Linux Documentation Project - Traffic Control](http://tldp.org/HOWTO/Traffic-Control-HOWTO/overview.html)
