---
date: "2016-11-22"
draft: false
weight: 30
title: "Lab 03b - ADVANCED - Open vSwitch (OVS)"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x1F680;OPTIONAL&#x1F680;**

### Lab Objective

The objective of this lab is very similar to the Open vSwitch lab, however it takes a much deeper dive into OVS concepts. This lab has been designed for advanced students, and therefore is *not* required. This lab assumes you understand the configuration steps in the "BASIC - Open vSwitch (OVS)" lab. Therefore, only attempt this lab after you have finished its "basic" counterpart.

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

  `student@beachhead:~$` `ip link list > /tmp/ovs2-link-init`

0. Save the current L3 configuration in a file called *ovs-addr-init*.

  `student@beachhead:~$` `ip addr show > /tmp/ovs2-addr-init`

0. Save the current routing information in a file called *ovs-route-init*.

  `student@beachhead:~$` `ip route show > /tmp/ovs2-route-init`

0. Finally, save the current state of OVS confirmation in a file called *ovs-show-init*.

  `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs2-show-init`

0. Now let's create some network namespaces. In keeping in the spirit of our previous labs, we'll call this namespace *peach*

  * `student@beachhead:~$` `sudo ip netns add peach`
  
0. And we'll need a second name space called *bowser*.  
  
  * `student@beachhead:~$` `sudo ip netns add bowser`
  
0. Create an Open vSwitch bridge. The following command instructs Open vSwitch to create a bridge named *donut-plains*.
  
  * `student@beachhead:~$` `sudo ovs-vsctl add-br donut-plains`
  
0. Okay, so now let's create two veth pairs for adding into the namespaces.  
  
  * `student@beachhead:~$` `sudo ip link add eth0-peach type veth peer name veth-peach`
  
0. Create another veth pair.  
  
  * `student@beachhead:~$` `sudo ip link add eth0-bowser type veth peer name veth-bowser`

0. Add the veths into the network namespaces. First set the *eth0-peach* veth into the network namespace *peach*

  * `student@beachhead:~$` `sudo ip link set eth0-peach netns peach`
  
0. Add the second veth, *eth0-bowser*, into the network namespace *bowser*.  
  
  * `student@beachhead:~$` `sudo ip link set eth0-bowser netns bowser`
  
0. Add the namespace interfaces to the Open vSwitch bridge. First we'll add *veth-peach* to our Open Vswitch bridge named *donut-plains*.
  
  * `student@beachhead:~$` `sudo ovs-vsctl add-port donut-plains veth-peach`

0. Now add *veth-bowser* to our Open vSwitch bridge named *donut-plains*.

  * `student@beachhead:~$` `sudo ovs-vsctl add-port donut-plains veth-bowser`
  
0. Apply *VLAN tag 50* to *veth-peach*.
  
  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-peach tag=50`

0. Apply *VLAN tag 150* to *veth-bowser*.

  * `student@beachhead:~$` `sudo ovs-vsctl set port veth-bowser tag=150`

0. Bring all the interaces up and examine the changes. The following command will place the *veth-peach* interface in an UP state.

  * `student@beachhead:~$` `sudo ip link set veth-peach up`
  
0. The following command will place the *veth-bowser* interface in an UP state.
  
  * `student@beachhead:~$` `sudo ip link set veth-bowser up`
  
0. The following command will place the *eth0-peach* interface within the network namespace *peach* in an UP state.  
  
  * `student@beachhead:~$` `sudo ip netns exec peach ip link set dev eth0-peach up`
  
0. The following command will place the *eth0-bowser* interface within the network namespace *bowser* in an UP state.  
  
  * `student@beachhead:~$` `sudo ip netns exec bowser ip link set dev eth0-bowser up`

0. Copy the current state of L2 to the file *ovs2-link-config*.
  
  * `student@beachhead:~$` `ip link list > /tmp/ovs2-link-config`
  
0. Copy the current state of L3 (IP) to the file *ovs2-addr-config*.  
  
  * `student@beachhead:~$` `ip addr show > /tmp/ovs2-addr-config`
  
0. Copy the current state of IP routing to the file *ovs2-route-config*.  
  
  * `student@beachhead:~$` `ip route show > /tmp/ovs2-route-config`
  
0. Copy the current state of the Open vSwitch configuration to the file *ovs2-show-config*.
  
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs2-show-config`
  
0. Run the *a3diff* file to see how L2 configuration has changed since we started this lab.  
  
  * `student@beachhead:~$` `a3diff /tmp/ovs2-link-init /tmp/ovs2-link-config`

0. Run the *a3diff* file to see how L3 (IP) configuration has changed since we started this lab.

  * `student@beachhead:~$` `a3diff /tmp/ovs2-addr-init /tmp/ovs2-addr-config`

0. Run the *a3diff* file to see how the IP routing configuration has changed since we started this lab.

  * `student@beachhead:~$` `a3diff /tmp/ovs2-route-init /tmp/ovs2-route-config`

0. Run the *a3diff* file to see how the Open vSwitch configuration has changed since we started this lab.

  * `student@beachhead:~$` `a3diff /tmp/ovs2-show-init /tmp/ovs2-show-config`

### Setup dhcp for these two interfaces

0. Create new dhcp namespaces (for the dhcp servers)

  * `student@beachhead:~$` `ip netns add dhcp-peach`
  * `student@beachhead:~$` `ip netns add dhcp-bowser`

0. Create internal ovs interfaces for the dhcp servers to run on

  * `student@beachhead:~$` `sudo ocs-vsctl add-port yoshi-island dhcp-peach -- set interface dhcp-peach type=internal`
  * `student@beachhead:~$` `sudo ocs-vsctl add-port yoshi-island dhcp-bowser -- set interface dhcp-bowser type=internal`
  * `student@beachhead:~$` `sudo ovs-vsctl set port dhcp-peach tag=50`
  * `student@beachhead:~$` `sudo ovs-vsctl set port dhcp-bowser tag=150`
  * `student@beachhead:~$` `sudo ip link set dhcp-peach netns dhcp-peach`
  * `student@beachhead:~$` `sudo ip link set dhcp-bowser netns dhcp-bowser`
  * `student@beachhead:~$` `sudo ovs-vsctl show > /tmp/ovs-show-dhcp`
  * `student@beachhead:~$` `a3diff /tmp/ovs2-show-config /tmp/ovs-show-dhcp`
  * `student@beachhead:~$` `ip link show > /tmp/ovs-link-dhcp`
  * `student@beachhead:~$` `a3diff /tmp/ovs-link-config /tmp/ovs-link-dhcp`

0. Bring up dhcp interfaces and assign ip addresses  

  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach ip link set dev dhcp-peach up`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach ip addr add 172.16.2.50/24 dev dhcp-peach`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser ip link set dev dhcp-bowser up`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser ip addr add 172.16.2.150/24 dev dhcp-bowser`

0. Start the dnsmasq service in both network name spaces

  
  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach dnsmasq --interface=dhcp-peach --dhcp-range=172.16.2.51,172.16.2.149,255.255.255.0`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser dnsmasq --interface=dhcp-bowser --dhcp-range=172.16.2.151,172.16.2.249,255.255.255.0`

0. DHCPDISCOVER 

  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach dhclient eth0-peach`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-peach ip addr`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser dhclient eth0-bowser`
  * `student@beachhead:~$` `sudo ip netns exec dhcp-bowser ip addr`
 
0. OVS additional commands

  * `student@beachhead:~$` `sudo ovs-appctl show-commands`
  * `student@beachhead:~$` `sudo ovs-appctl fdb/show donut-plains`
  * `student@beachhead:~$` `sudo ovs-vsctl dump flows donut-plains`
  
0. Well that was fun! Time to cleanup our system. Running all of the steps from here down will remove all of our configuration.

0. Trash the network namespace *peach*.

  `student@beachhead:~$` `sudo ip netns del peach`

0. Trash the network namespace *bowser*.

  `student@beachhead:~$` `sudo ip netns del bowser`

0. Trash the L2 interface *veth-peach*.

  `student@beachhead:~$` `sudo ip link del veth-peach`

0. Trash the L2 interface *veth-bowser*.

  `student@beachhead:~$` `sudo ip link del veth-bowser`

0. Finally, trash the bridge *donut-plains*.

  `student@beachhead:~$` `ovs-vsctl del-br donut-plains`

0. You should be right back to the way things were before you started this lab. If anything was unclear, run it again!
