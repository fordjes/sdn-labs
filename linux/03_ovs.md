# Bridging 

# What is a bridge
A bridge transparently relays traffic between multiple network interfaces.
A bridge connects two or more physical links together to form one bigger (logical) link.
A bridge does not know anything about protocols, it only sees Ethernet frames.
A bridge is essentially a switch, but with all the flexibility of Linux networking.

## Create a OVS Bridge with virtual interfaces

0. Create a pair of virtual ethernet interfaces and bring them up

  * `ubuntu@ubuntu:~$` `ip link list`
  * `ubuntu@ubuntu:~$` `ip link list > /tmp/link-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `sudo ip link add veth1 type veth peer name veth2`
  * `ubuntu@ubuntu:~$` `sudo ip link set dev veth1 up`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/link-before <(ip link list) | colordiff`

0. Create a bridge, give it an IP,  and add the virtual interfaces to it

  * `ubuntu@ubuntu:~$` `ip link list > /tmp/link-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `ip address show > /tmp/addr-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `ip route show > /tmp/route-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl add-br br0`
  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl list-br`
  * `ubuntu@ubuntu:~$` `sudo ip address add 10.0.0.1/24 dev br0`
  * `ubuntu@ubuntu:~$` `sudo ip link set dev br0 up`
  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl list-ports br0`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/link-before <(ip link list) | colordiff`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/addr-before <(ip address show) | colordiff`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/route-before <(ip route show) | colordiff`

## Use the OVS bridge to access a network namespace 

0. Create a network namespace and move the 2nd veth into it 

  * `ubuntu@ubuntu:~$` `sudo ip netns add mario`
  * `ubuntu@ubuntu:~$` `sudo ip netns list`
  * `ubuntu@ubuntu:~$` `ip link list > /tmp/link-before` _save for comparison_
  * `ubuntu@ubuntu:~$` `sudo ip link set veth2 netns mario`
  * `ubuntu@ubuntu:~$` `wdiff -n /tmp/link-before <(ip link list) | colordiff`

0. Configure the namespace 
  
  * `ubuntu@ubuntu:~$` `sudo ip netns exec mario ip addr add 10.0.0.2/24 dev veth2`
  * `ubuntu@ubuntu:~$` `sudo ip netns exec mario ip link set dev veth2 up`

0. Demonstrate connectivity

  * `ubuntu@ubuntu:~$` `ping -c4 10.0.0.2`
  * `ubuntu@ubuntu:~$` `sudo ip netns exec mario ping -c4 10.0.0.1`

## Cleanup

  * `ubuntu@ubuntu:~$` `sudo ovs-vsctl del-br br0`
  * `ubuntu@ubuntu:~$` `sudo ip link del dev veth1`
  * `ubuntu@ubuntu:~$` `sudo ip netns del mario`
  * `ubuntu@ubuntu:~$` `ip link show` _expected veth1, veth2, br0 absent_
  * `ubuntu@ubuntu:~$` `ip netns list` _expected empty_





