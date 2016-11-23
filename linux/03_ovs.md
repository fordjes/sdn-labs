---
date: "2016-11-22"
draft: false
weight: 30
title: "Lab 03 - Open vSwitch (OVS)"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective

The objective of this lab is very similar to the Linux Bridge lab, except we will use an Open vSwitch (OVS bridge).

Some of the commands are presented in a different order than the Linux Bridge lab.

### Procedure

0. Close any terminals, and Firefox windows you currently have open within the remote desktop session.

0. From your remote desktop, open a new terminal session, and move to the student home directory.

  `student@beachhead:/$` `cd`

0. We explored this in the last lab, but just to cover all our bases. In this lab, we'll be using a little code snippet we invented called the *a3diff function*. If you tail the .bashrc file, you'll see it at the end. Just something we want you to be aware of, as we'll be using it in this lab to compare the output of some iproute2 commands.

  `student@beachhead:~$` `tail .bashrc`

  ```
  function a3diff {
      wdiff -n $1 $2 | colordiff
  }
  ```
  >
  If you're not a programmer, no big deal. All you have to know is that this little code snippit will save us some keystrokes, and allow us to do two (2) things:
  >
  1) Look for within-line differences (`wdiff`)   
  >
  2) Display the results in color (`colordiff`)

0. While we eventually want to create a OVS Bridge with virtual interfaces, let's first collect information about our current configuration. First display the L2 information.

  `student@beachhead:~$` `ip link list`
  
0. Great, now display L3 information.

  `student@beachhead:~$` `ip addr show`

0. Finally, show the current state of the routing table.

  `student@beachhead:~$` `ip route show`
  
  ```
  default via 172.16.1.1 dev ens3
  169.254.169.254 via 172.16.1.1 dev ens3
  172.16.1.0/24 dev ens3  proto kernel  scope link  src 172.16.1.4
  ```

0.  The ovs-vsctl is the utility used for quering and confiruing ovs-vswitchd, so let's start with reviewing usage.

  `student@beachhead:~$` `sudo ovs-vsctl -help`

0. Let's display the version of OVS software we are using, as well as show the state of OVS switches tracked by the OVS database.
  
  `student@beachhead:~$` `sudo ovs-vsctl show`
  
  ```
  a00e3ca2-3926-4d0d-9f71-a2105f6c6166
    ovs_version: "2.5.0"
  ```

0. Save the current L2 configuration in a file called *ovs-link-init*.

  `student@beachhead:~$` `ip link list > /tmp/ovs-link-init`

0. Save the current L3 configuration in a file called *ovs-addr-init*.

  `student@beachhead:~$` `ip addr show > /tmp/ovs-addr-init`

0. Save the current routing information in a file called *ovs-route-init*.

  `student@beachhead:~$` `ip route show > /tmp/ovs-route-init`

0. Finally, save the current state of OVS confirmation in a file called *ovs-show-init*.

  `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-init`

0. Now let's create some network namespaces. In keeping in the spirit of our previous labs, we'll call this namespace *luigi*

  `student@beachhead:~$` `sudo ip netns add luigi`

0. And we'll need a second name space called *toad*.

  `student@beachhead:~$` `sudo ip netns add toad`

0. Display the current namespaces configured within the system.

  `student@beachhead:~$` `ip netns show`
  
  ```
  toad
  luigi
  ```
  
0. Network namespaces, like the two we just created, are tracked in the following location.

  `student@beachhead:~$` `ls /var/run/netns -ll`
  
  ```
  total 0
  -r--r--r-- 1 root root 0 Nov 22 15:52 luigi
  -r--r--r-- 1 root root 0 Nov 22 15:53 toad
  ```
  
0. Examine network namespaces. The following command will reveal the network devices within the luigi namespace.

  * `student@beachhead:~$` `sudo ip netns exec luigi ip link`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  ```

0.  The following command will reveal the network devices within the toad namespace.

  `student@beachhead:~$` `sudo ip netns exec toad ip link`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  ```
  
0. Write the current configuration of the *luigi* namespace to the file *ovs-l-link-init*.

  `student@beachhead:~$` `sudo ip netns exec luigi ip link > /tmp/ovs-l-link-init`

0. Write the current configuration of the *toad* namespace to the file *ovs-t-link-init*.

  `student@beachhead:~$` `sudo ip netns exec toad ip link > /tmp/ovs-t-link-init`

0. Create an open vswitch bridge. The following command instructs Open vSwitch to create a bridge named *yoshis-island*

  `student@beachhead:~$` `sudo ovs-vsctl add-br yoshis-island`

0. Great. Now check out the current state of the Open vSwitch.

  `student@beachhead:~$` `sudo ovs-vsctl show`
  
  ```
  a00e3ca2-3926-4d0d-9f71-a2105f6c6166
    Bridge yoshis-island
        Port yoshis-island
            Interface yoshis-island
                type: internal
    ovs_version: "2.5.0"
   ```
  
0. Write the current state of Open vSwtich to the file *ovs-show-br*, so that we can compare it to our original configuration.

  `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-br`
  
0. Run the *a3diff* function to clearly reveal how the Open vSwitch configuration has changed.

  `student@beachhead:~$` `a3diff /tmp/ovs-show-init /tmp/ovs-show-br`
  
  ```
  a00e3ca2-3926-4d0d-9f71-a2105f6c6166
      {+Bridge yoshis-island+}
  {+        Port yoshis-island+}
  {+            Interface yoshis-island+}
  {+                type: internal+}
      ovs_version: "2.5.0"
  ```

0. Now that we've created this Open vSwitch, let's write the current state of L2 to a new file, so that we might compare it to our original configuration.

  `student@beachhead:~$` `ip link show > /tmp/ovs-link-br`

0. Run the *a3diff* function to clearly reveal how the creation of the Open vSwitch bridge changed L2 configuration. 

  `student@beachhead:~$` `a3diff /tmp/ovs-link-init /tmp/ovs-link-br`
  
  ```
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether fa:16:3e:59:a0:b8 brd ff:ff:ff:ff:ff:ff
  3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether fa:16:3e:ac:cb:ff brd ff:ff:ff:ff:ff:ff
  11: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1
      link/ether ea:fb:5d:d4:53:b6 brd ff:ff:ff:ff:ff:ff
  {+17: yoshis-island: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1+}
  {+    link/ether b2:b4:6d:13:43:48 brd ff:ff:ff:ff:ff:ff+}
   ```

0. Write the current state of the L3 system configuration to a file called *ovs-addr-br*

  `student@beachhead:~$` `ip addr show > /tmp/ovs-addr-br`

0. Run the *a3diff* function to clearly reveal how the creation of the Open vSwitch bridge changed L3 configuration. 

  `student@beachhead:~$` `a3diff /tmp/ovs-addr-init /tmp/ovs-addr-br`

0. Okay, so now let's create two veth pairs, will get added into the namespaces.

  `student@beachhead:~$` `sudo ip link add eth0-luigi type veth peer name veth-luigi`
  
0. Create another veth pair.

  `student@beachhead:~$` `sudo ip link add eth0-toad type veth peer name veth-toad`

0. Write out the current L2 state to a file called *ovs-link-veths*

  `student@beachhead:~$` `ip link show > /tmp/ovs-link-veths`

0. Examine how L2 configuration changed with the creation of these two veth pairs.

  `student@beachhead:~$` `a3diff /tmp/ovs-link-br /tmp/ovs-link-veths`

0. Add the veths into the network namespaces. First set the *eth0-luigi* veth into the network namespace *luigi*

  `student@beachhead:~$` `sudo ip link set eth0-luigi netns luigi`

0. Add the second veth, *eth0-toad*, into the network namespace *toad*.

  `student@beachhead:~$` `sudo ip link set eth0-toad netns toad`

0. Write out the current L2 state to a file called *ovs-link-vethmove*

  `student@beachhead:~$` `ip link show > /tmp/ovs-link-vethmove`

0. Compare the current state of L2 (having placed two of the veth interfaces into namespaces), with how L2 looked just after creating those veth pairs.

  `student@beachhead:~$` `a3diff /tmp/ovs-link-veths /tmp/ovs-link-vethmove`
  
0. So we just examined how the changes look outside of the network namespaces. Now lets look at how things have changed inside of the network namespaces. This next command will write out the L2 information from within the *luigi* network namespace to a file called *ovs-l-link-veth*.
 
  `student@beachhead:~$` `sudo ip netns exec luigi ip link > /tmp/ovs-l-link-veth`
  
0. We can issue the same command within the *toad* namespace, creating a new file called *ovs-t-link-veth*.

  `student@beachhead:~$` `sudo ip netns exec toad ip link > /tmp/ovs-t-link-veth`

0. Run the *a3diff* function on how the *luigi* namespace looks now that a veth pair has been added to it.
  
  `student@beachhead:~$` `a3diff /tmp/ovs-l-link-init /tmp/ovs-l-link-veth`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  {+19: eth0-luigi@if18: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
  {+    link/ether c2:4f:8c:83:8f:54 brd ff:ff:ff:ff:ff:ff link-netnsid 0+}
  ```

0. Run the *a3diff* function on how the *luigi* namespace looks now that a veth pair has been added to it.
  
  `student@beachhead:~$` `a3diff /tmp/ovs-t-link-init /tmp/ovs-t-link-veth`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  {+21: eth0-toad@if20: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000+}
  {+    link/ether ea:07:01:40:30:b6 brd ff:ff:ff:ff:ff:ff link-netnsid 0+}
  ```
  
0. Alright, so we pretty much exhausted the L2 basics, so let's move up to L3. Take a look a the current L3 (IP) configuration in the *luigi* namespace. The following command executes the command **ip addr** within the luigi namespace.

  `student@beachhead:~$` `sudo ip netns exec luigi ip addr`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  19: eth0-luigi@if18: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
      link/ether c2:4f:8c:83:8f:54 brd ff:ff:ff:ff:ff:ff link-netnsid 0
  ```

0. Take a look a the current L3 (IP) configuration in the *toad* namespace. The following command executes the command **ip addr** within the toad namespace.

  `student@beachhead:~$` `sudo ip netns exec toad ip addr`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  21: eth0-toad@if20: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
      link/ether ea:07:01:40:30:b6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
  ```

0. Save the current L3 information from the *luigi* namespace in a file called, *ovs-l-addr-init*

  `student@beachhead:~$` `sudo ip netns exec luigi ip addr > /tmp/ovs-l-addr-init`

0. Save the current L3 information from the *toad* namespace in a file called, *ovs-t-addr-init*

  `student@beachhead:~$` `sudo ip netns exec toad ip addr > /tmp/ovs-t-addr-init`

0. Add the namespace interfaces to the Open vSwitch bridge. First we'll add *veth-luigi* to our Open Vswitch bridge named *yoshis-island*.

  `student@beachhead:~$` `sudo ovs-vsctl add-port yoshis-island veth-luigi`

0. Now add *veth-toad* to our Open vSwitch bridge named *yoshis-island*.

  `student@beachhead:~$` `sudo ovs-vsctl add-port yoshis-island veth-toad`

0. Make a copy of the current Open vSwitch configuration in a file called, *ovs-show-veth*.

  `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-veth`

0. Compare how adding the veths to the Open vSwitch bridge changed its configuration.

  `student@beachhead:~$` `a3diff /tmp/ovs-show-br /tmp/ovs-show-veth`
  
  ```
  a00e3ca2-3926-4d0d-9f71-a2105f6c6166
      Bridge yoshis-island
          Port {+veth-toad+}
  {+            Interface veth-toad+}
  {+        Port+} yoshis-island
              Interface yoshis-island
                  type: internal
          {+Port veth-luigi+}
  {+            Interface veth-luigi+}
      ovs_version: "2.5.0"
  ``` 

0. Bring all the interaces up and examine the changes. The following command will place the veth-luigi interface in an UP state.

  `student@beachhead:~$` `sudo ip link set veth-luigi up`

0. The following command will place the veth-toad interface in an UP state.

  `student@beachhead:~$` `sudo ip link set veth-toad up`
  
0. The following command will place the *eth0-luigi* interface within the network namespace *luigi* in an UP state.

  `student@beachhead:~$` `sudo ip netns exec luigi ip link set dev eth0-luigi up`

0. The following command will place the *eth0-toad* interface within the network namespace *toad* in an UP state.

  `student@beachhead:~$` `sudo ip netns exec toad ip link set dev eth0-toad up`

0. Write out the current L2 configuration within the *luigi* namespace to a file called, *ovs-l-link-up*.

  `student@beachhead:~$` `sudo ip netns exec luigi ip link > /tmp/ovs-link-l-link-up`

0. Write out the current L2 configuration within the *toad* namespace to a file called, *ovs-t-link-up*.

  `student@beachhead:~$` `sudo ip netns exec toad ip link > /tmp/ovs-link-t-link-up`

0. Use the *a3diff* function to compare how placing the interfaces in an UP state within the *luigi* namespaces changed the L2 configuration. 

  `student@beachhead:~$` `a3diff /tmp/ovs-l-link-veth /tmp/ovs-link-l-link-up`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  14: eth0-luigi@if13: [-<BROADCAST,MULTICAST>-] {+<BROADCAST,MULTICAST,UP,LOWER_UP>+} mtu 1500 qdisc [-noop-] {+noqueue+} state [-DOWN-] {+UP+} mode DEFAULT group default qlen 1000
      link/ether aa:d9:a6:f9:f5:e3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
  ```

0. Use the *a3diff* function to compare how placing the interfaces in an UP state within the *toad* namespaces changed the L2 configuration. 

  `student@beachhead:~$` `a3diff /tmp/ovs-t-link-veth /tmp/ovs-link-t-link-up`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  16: eth0-toad@if15: [-<BROADCAST,MULTICAST>-] {+<BROADCAST,MULTICAST,UP,LOWER_UP>+} mtu 1500 qdisc [-noop-] {+noqueue+} state [-DOWN-]   {+UP+} mode DEFAULT group default qlen 1000
      link/ether 5e:9b:e9:d9:66:f8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
  ```

0. Add IP addresses to the internal interaces. First apply the IP address to 172.16.2.100 to the dev *eth0-luigi*.

  `student@beachhead:~$` `sudo ip netns exec luigi ip addr add 172.16.2.100/24 dev eth0-luigi`

0. Add the IP address 172.16.2.101 to the dev *eth0-toad*.

  `student@beachhead:~$` `sudo ip netns exec toad ip addr add 172.16.2.101/24 dev eth0-toad`

0. Run the *ip addr* in the *luigi* network namespace to reveal the current L3 (IP) configuration.

  `student@beachhead:~$` `sudo ip netns exec luigi ip addr`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  14: eth0-luigi@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
      link/ether aa:d9:a6:f9:f5:e3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      inet 172.16.2.100/24 scope global eth0-luigi
         valid_lft forever preferred_lft forever
      inet6 fe80::a8d9:a6ff:fef9:f5e3/64 scope link
         valid_lft forever preferred_lft forever
  ```

0. Run the *ip addr* in the *toad* network namespace to reveal the current L3 (IP) configuration.

  `student@beachhead:~$` `sudo ip netns exec toad ip addr`

  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  16: eth0-toad@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
      link/ether 5e:9b:e9:d9:66:f8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      inet 172.16.2.101/24 scope global eth0-toad
         valid_lft forever preferred_lft forever
      inet6 fe80::5c9b:e9ff:fed9:66f8/64 scope link
         valid_lft forever preferred_lft forever
  ```

0. Write the current L3 (IP) configuration of the *luigi* namespace into a file called *ovs-l-addr-ip*.

  `student@beachhead:~$` `sudo ip netns exec luigi ip addr > /tmp/ovs-link-addr-ip`

0. Write the current L3 (IP) configuration of the *toad* namespace into a file called *ovs-t-addr-ip*.

  `student@beachhead:~$` `sudo ip netns exec toad ip addr > /tmp/ovs-toad-addr-ip`

0. Run the *a3diff* function to display how the L3 (IP) configuration wtihin the *luigi* namespace changed when an IP address was added to the *eth0-luigi* interface.

  `student@beachhead:~$` `a3diff /tmp/ovs-l-addr-init /tmp/ovs-link-addr-ip`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  19: eth0-luigi@if18: [-<BROADCAST,MULTICAST>-] {+<BROADCAST,MULTICAST,UP,LOWER_UP>+} mtu 1500 qdisc [-noop-] {+noqueue+} state [-DOWN-] {+UP+} group default qlen 1000
      link/ether c2:4f:8c:83:8f:54 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      {+inet 172.16.2.100/24 scope global eth0-luigi+}
  {+       valid_lft forever preferred_lft forever+}
  {+    inet6 fe80::c04f:8cff:fe83:8f54/64 scope link +}
  {+       valid_lft forever preferred_lft forever+}
  ```

0. Run the *a3diff* function to display how the L3 (IP) configuration wtihin the *toad* namespace changed when an IP address was added to the *eth0-toad* interface.

  `student@beachhead:~$` `a3diff /tmp/ovs-t-addr-init /tmp/ovs-toad-addr-ip`
  
  ```
  1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  21: eth0-toad@if20: [-<BROADCAST,MULTICAST>-] {+<BROADCAST,MULTICAST,UP,LOWER_UP>+} mtu 1500 qdisc [-noop-] {+noqueue+} state [-DOWN-]   {+UP+} group default qlen 1000
      link/ether ea:07:01:40:30:b6 brd ff:ff:ff:ff:ff:ff link-netnsid 0
      {+inet 172.16.2.101/24 scope global eth0-toad+}
  {+       valid_lft forever preferred_lft forever+}
  {+    inet6 fe80::e807:1ff:fe40:30b6/64 scope link +}
  {+       valid_lft forever preferred_lft forever+}
   ```

0. At this time we cannot ping the addresses we applied to the interfaces. Confirm this is the case by trying to ping, *172.16.2.100*, which was applied to *eth0-luigi*.

  `student@beachhead:~$` `ping -c 2 172.16.2.100`

0. It won't work. You'll need to press **CTRL + C** to quit trying to ping.

0. Try to ping the other IP address, *172.16.2.101*, which was applied to *eth0-toad*.

  `student@beachhead:~$` `ping -c 2 172.16.2.101`

0. So that one didn't work either. You'll need to press **CTRL + C** to quit trying to ping.

  `student@beachhead:~$` `ip route`

0. So from outside of the network namespaces, we are unable to ping these IP addresses. However, these two IP do have connectivity to each other from *within* the namespace.

  `student@beachhead:~$` `sudo ip netns exec luigi ping 172.16.2.101`
  
  ```
  PING 172.16.2.101 (172.16.2.101) 56(84) bytes of data.
  64 bytes from 172.16.2.101: icmp_seq=1 ttl=64 time=0.739 ms
  64 bytes from 172.16.2.101: icmp_seq=2 ttl=64 time=0.089 ms
  ^C
  ```

0. So the IP address is there, and assigned to an interface within the Linux system. However, we can only access it if we gain access to the network namespace. You'll need to press **CTRL + C** to quit pinging.

  `student@beachhead:~$` `sudo ip netns exec toad ping 172.16.2.100`

  ```
  PING 172.16.2.100 (172.16.2.100) 56(84) bytes of data.
  64 bytes from 172.16.2.100: icmp_seq=1 ttl=64 time=0.052 ms
  64 bytes from 172.16.2.100: icmp_seq=2 ttl=64 time=0.063 ms
  ^C
  ```

0. Again, that should work. You'll need to press **CTRL + C** to quit trying to ping.

0. VLAN tags (L2) can also exist within the network namespaces. If two devices have different VLAN tags applied to them, they're effectively on seperate L2 networks. So to drive home a point, let's change the VLAN tags for the two interfaces, and then demonstrate that they can no loger connect with each other.

0. Apply *VLAN tag 100* to *veth-luigi*

  `student@beachhead:~$` `sudo ovs-vsctl set port veth-luigi tag=100`
  
0. Apply *VLAN tag 101* to *veth-toad*
  
  `student@beachhead:~$` `sudo ovs-vsctl set port veth-toad tag=101`

0. Try pinging the IP address *172.16.2.101*, the IP applied to *eth0-toad*, from within the *luigi* namespace.

  `student@beachhead:~$` `sudo ip netns exec luigi ping 172.16.2.101`
  
0. It won't work. You'll need to press **CTRL + C** to quit trying to ping.

0. Try pinging the IP address *172.16.2.100*, the IP applied to *eth0-luigi*, from within the *toad* namespace.

  `student@beachhead:~$` `sudo ip netns exec toad ping 172.16.2.100`

0. It won't work. You'll need to press **CTRL + C** to quit trying to ping.

0. Let's check out our "final" OVS configuration by creating a file called *ovs-show-tags*.

  `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-tags`

0. Run the *a3diff* function one last time to reveal how the addition of VLAN tags changed the OVS configuration.

  `student@beachhead:~$` `a3diff /tmp/ovs-show-veth /tmp/ovs-show-tags`
  
  ```
  a00e3ca2-3926-4d0d-9f71-a2105f6c6166
    Bridge yoshis-island
        Port veth-toad
            {+tag: 101+}
            Interface veth-toad
        Port yoshis-island
            Interface yoshis-island
                type: internal
        Port veth-luigi
            {+tag: 100+}
            Interface veth-luigi
    ovs_version: "2.5.0"
  ```

0. So imagine what we just demonstrated on a massive scale, with management augmented by a human. That's effectively what the OVS project has set out to do. Configure lots of network namespaces, connect them together when applicable (by veths), and further isolate L2 networks with VLAN tags.

0. Well that was fun! Time to cleanup our system. Running all of the steps from here down will remove all of our configuration.

0. Trash the network namespace *luigi*.

  `student@beachhead:~$` `sudo ip netns del luigi`

0. Trash the network namespace *toad*.

  `student@beachhead:~$` `sudo ip netns del toad`

0. Trash the L2 interface *veth-luigi*.

  `student@beachhead:~$` `sudo ip link del veth-luigi`

0. Trash the L2 interface *veth-toad*.

  `student@beachhead:~$` `sudo ip link del veth-toad`

0. Finally, trash the bridge *yoshis-island*.

  `student@beachhead:~$` `ovs-vsctl del-br yoshis-island`

0. You should be right back to the way things were before you started this lab. If anything was unclear, run it again!
