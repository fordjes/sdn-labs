---
date: "2016-11-23"
draft: false
weight: 200
title: "Lab xx - Mininet"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to introduce mininet.

### Procedure

0. From your remote desktop, open a terminal session, and move to the student home directory.

  `student@beachhead:/$` `cd`

0.

  `student@beachhead:~$` `ifconfig -a`

0. Default all the things + investigate

  * `student@beachhead:~$` `sudo mn`
  * `mininet>` `help`
  * `mininet>` `nodes`
  * `mininet>` `dump`
  * `mininet>` `net`

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
