## Virtual Interfaces

0. Create a pair of virtual ethernet interfaces 

  * `ubuntu@ubuntu:~$` `sudo ip link add veth0 type veth peer name veth1`
  * `ubuntu@ubuntu:~$` `ip link list`

0. Bring up and add ip addresses to both virtual ethernet interfaces we created

  * `ubuntu@ubuntu:~$` `sudo ip link set dev veth0 up`
  * `ubuntu@ubuntu:~$` `sudo ip link set dev veth1 up`
  * `ubuntu@ubuntu:~$` `ip link list`
  * `ubuntu@ubuntu:~$` `sudo ip addr add 10.0.0.1/24 dev veth0`
  * `ubuntu@ubuntu:~$` `sudo ip addr add 10.0.0.2/24 dev veth1`
  * `ubuntu@ubuntu:~$` `ip link list`

0. Examine the results
  
  > What are the mac addresses of our new interfaces?  
  > Where did these mac addresses come from?
  > How can we set these addresses if we wanted to ?

  * `ubuntu@ubuntu:~$` `ip addr show veth0`
  * `ubuntu@ubuntu:~$` `ip addr show veth1`

0. Ping from one veth interface to the other

  > Why wont this work?

  * `ubuntu@ubuntu:~$` `ping -I veth1 10.0.0.1`
  * `ubuntu@ubuntu:~$` `ping -I veth0 10.0.0.2`

0. Examine the routing table and run tcpdump whit the previous commands

  > When we force the ICMP packet destine for 10.0.0.2 on veth0 

  * `ubuntu@ubuntu:~$` `ping -I veth1 10.0.0.1 > /dev/null &` _ping in the background_
  * `ubuntu@ubuntu:~$` `sudo tcpdump -i veth1` _Ctrl^C to exit_
  * `ubuntu@ubuntu:~$` `fg` _bring the ping back into the foreground and press Ctrl^C to quit_
  * `ubuntu@ubuntu:~$` `ip route`


  * `ubuntu@ubuntu:~$` `ip route del 10.0.0.0/24 dev veth0`
  * `ubuntu@ubuntu:~$` `ip route del 10.0.0.0/24 dev veth1`
  * `ubuntu@ubuntu:~$` `ip route add 10.0.0.0/24 via 10.0.0.1 dev veth0`
  * `ubuntu@ubuntu:~$` `ip route`
  * `ubuntu@ubuntu:~$` `ip route`


0. Remove all the virtual interfaces (back to scratch)

  > Why do we only need to run the delete once?

  * `ubuntu@ubuntu:~$` `sudo ip link del veth0`
  * `ubuntu@ubuntu:~$` `ip link list`

