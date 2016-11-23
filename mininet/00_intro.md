---
date: "2016-11-23"
draft: false
weight: 200
title: "Lab xx - Introduction to Mininet"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to introduce the Mininet CLI. Mininet creates a realistic virtual network, running real kernel, switch and application code, on a single machine (VM, cloud or native), in seconds, with a single command. It is easily manipulated with the Mininet CLI, which makes it a great tool for developing, share, and experiment with OpenFlow and Software-Defined Networking systems.

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

0. The mininet *net* command can be used to list any mininet network connections.

  `mininet>` `net`
  
  ```
  h1 h1-eth0:s1-eth1
  h2 h2-eth0:s1-eth2
  s1 lo:  s1-eth1:h1-eth0 s1-eth2:h2-eth0
  ```

0. We can also issue commands from in these "hosts"

  * `mininet>` `h1 ping -c 2 h2`
  * `mininet>` `h1 lxterminal &`
  * `root@beachhead:~#` `ping -c 2 10.0.0.2` 
  * `root@beachhead:~#` `ip link show` 
  * `root@beachhead:~#` `exit 
  * `mininet>`

  > Given these outputs from commands run inside `h1`, what is the likely way in which mininet has setup this environment for us?

0. Setup tcpdump on h1

  * `mininet>` `h1 lxterminal &`
  * `root@beachhead:~#` `tcpdump -n` 

0. Show connectivity while capturing traffic

  * `mininet>` `h1 ping -c 2 h2`
  * `mininet>` `pingall`
  * `root@beachhead:~#` `ctrl` + `c`
  * `root@beachhead:~#` `exit` 

0. Show performance between nodes
 
  * `mininet>` `iperf`
  * `mininet>` `iperfudp`
   
0. Exit, cleanup and restart mininet with induced latency 

  * `mininet>` `exit`
  * `student@beachhead:~$` `sudo mn --clean`
  * `student@beachhead:~$` `man tc`
  * `student@beachhead:~$` `sudo mn --link tc,bw=20,delay=20ms`

0. Show performance between nodes
 
  * `mininet>` `iperf`
  * `mininet>` `iperfudp`
  * `mininet>` `exit`
  * `student@beachhead:~$` 

0. Show mn topology options

  * `student@beachhead:~$` `sudo mn --topo=single,5` 
  * `mininet>` `pingall`
  * `mininet>` `exit`
  * `student@beachhead:~$` `sudo mn --topo=linear,5` 
  * `mininet>` `pingall`
  * `mininet>` `exit`
  * `student@beachhead:~$` `sudo mn --topo=tree,5` 
  * `mininet>` `pingall`
  * `mininet>` `exit`
