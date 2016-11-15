# Bridging 

## What is a bridge
A bridge transparently relays traffic between multiple network interfaces.
A bridge connects two or more physical links together to form one bigger (logical) link.
A bridge does not know anything about protocols, it only sees Ethernet frames.
A bridge is essentially a switch, but with all the flexibility of Linux networking.

## a3diff

Open .bashrc and read the funciton `a3diff`.  
We will use this function in this lab to compare the output of some iproute2 commands.
We are saving ourselves some keystrokes in order to accomplish two things:

* look for within-line differences (`wdiff`)
* display the results in color (`colordiff`)

## Create a traditional Linux bridge with virtual interfaces

0. Create a pair of virtual ethernet interfaces 

  * `ubuntu@beachhead:~$` `ip link list`
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-init` _save for comparison_
  * `ubuntu@beachhead:~$` `sudo ip link add veth1 type veth peer name veth2`
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-veth1` _save for comparison_
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-link-list-init /tmp/ip-link-veth1 `

0. Bring up the first virtual ethernet interfaces we created

  * `ubuntu@beachhead:~$` `sudo ip link set dev veth1 up`
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-veth1up` _save for comparison_
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-link-veth1 /tmp/ip-link-veth1up `

0. Create a bridge and add the virtual interfaces to it

  * `ubuntu@beachhead:~$` `sudo ip link add name br0 type bridge`
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-bridge` _save for comparison_
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-link-veth1up /tmp/ip-link-bridge `
  * `ubuntu@beachhead:~$` `sudo ip link set dev veth1 master br0 `
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-bridge-veth1` _save for comparison_
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-link-bridge /tmp/ip-link-bridge-veth1 `

0. Add an ip address to the bridge interface

  * `ubuntu@beachhead:~$` `ip address show` 
  * `ubuntu@beachhead:~$` `ip route show` 
  * `ubuntu@beachhead:~$` `ip address show > /tmp/ip-addr-bridge` _save for comparison_
  * `ubuntu@beachhead:~$` `ip route show > /tmp/ip-route-bridge` _save for comparison_
  * `ubuntu@beachhead:~$` `sudo ip address add 172.16.2.100/24 dev br0`
  * `ubuntu@beachhead:~$` `sudo ip link set dev br0 up`
  * `ubuntu@beachhead:~$` `ip address show > /tmp/ip-addr-bridge` _save for comparison_
  * `ubuntu@beachhead:~$` `ip route show > /tmp/ip-route-bridge` _save for comparison_
  * `ubuntu@beachhead:~$` `ip link show > /tmp/ip-link-bridge-addr` _save for comparison_
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-route /tmp/ip-route-bridge `
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-addr /tmp/ip-addr-bridge `
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-link-bridge /tmp/ip-link-list-bridge `
  * `ubuntu@beachhead:~$` `bridge link show br0`

## Use the Linux Brige to access a network namespace 

0. Create a network namespace and move the 2nd veth into it 

  * `ubuntu@beachhead:~$` `sudo ip netns add mario`
  * `ubuntu@beachhead:~$` `ip netns list`
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-netns` _save for comparison_
  * `ubuntu@beachhead:~$` `sudo ip link set veth2 netns mario`
  * `ubuntu@beachhead:~$` `ip link list > /tmp/ip-link-netns-veth2` _save for comparison_
  * `ubuntu@beachhead:~$` `a3diff /tmp/ip-link-netns /tmp/ip-link-netns-veth2 `

0. Configure the namespace ip address
  
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip link list`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip address show`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip link list > /tmp/mario-ip-link` _save for comparison_
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip address show > /tmp/mario-ip-addr` _save for comparison_
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip addr add 172.16.2.101/24 dev veth2`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip link set dev veth2 up`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip link list`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip address show`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip link list > /tmp/mario-ip-link-veth2` _save for comparison_
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ip address show > /tmp/mario-ip-addr-veth2` _save for comparison_

0. Demonstrate connectivity

  * `ubuntu@beachhead:~$` `ping -c4 172.16.2.101`
  * `ubuntu@beachhead:~$` `sudo ip netns exec mario ping -c4 172.16.2.100`

## Cleanup

  * `ubuntu@beachhead:~$` `sudo ip link del dev br0`
  * `ubuntu@beachhead:~$` `sudo ip link del dev veth1`
  * `ubuntu@beachhead:~$` `sudo ip netns del mario`
  * `ubuntu@beachhead:~$` `ip link show` _expected veth1, veth2, br0 absent_
  * `ubuntu@beachhead:~$` `ip netns list` _expected empty_





