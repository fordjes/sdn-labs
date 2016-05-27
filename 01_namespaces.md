
0. Create a new network namespace

  * `ubuntu@ubuntu:~$` `sudo ip netns add mario`
  * `ubuntu@ubuntu:~$` `ip netns list`
  * `ubutnu@ubuntu:~$` `ls /var/run/netns`

0. Create a virtual Ethernet pair

  * `ubuntu@ubuntu:~$` `sudo ip link add veth0 type veth peer name veth1`
  * `ubuntu@ubuntu:~$` `ip link list`

0. Add a virutal Ethernet to the namespace

  * `ubuntu@ubuntu:~$` `sudo ip link set veth1 netns mario`
  * `ubutnu@ubuntu:~$` `ip link list`  
  
  > Where did the veth1 go?  
  > What namespace are we displaying with the `ip link list` command?

0. Display the links inside the namespace

  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip link list`

0. "Enter" into a namespace with bash

  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario bash`
  * `ubutnu@ubuntu:~#` `ip addr`
  * `ubutnu@ubuntu:~#` `ip link list`
  * `ubutnu@ubuntu:~#` `exit # or press Ctl^D` 

0. Fix the link state of our veth inside the namespacep
  
  _You may also "enter" into the namespace instead of running `sudo ip netns exec mario` every time._  
  
  > What is the state of the link?

  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip link list`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip link set dev veth1 up`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip link list`
  

  > What is the state of the link now?

0. From inside the namespace, use `ip` to add `10.0.0.1` ip address `veth1`

  _You may also "enter" into the namespace instead of running `sudo ip netns exec mario` every time._  
  Use the help command in order to determine the correct command syntax.  
  If you have issues the solution is in a comment of the HTML of this page.  

  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip addr help`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip addr add [finish the command here]`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario ip addr 10.0.0.1 dev veth1` TODO: Hide this?

  <!-- ubuntu@ubuntu:~$ ip addr add 10.0.0.1 dev veth1 -->

0. From inside the namespace, use `ip` to add a default route to `veth1`

  _You may also "enter" into the namespace instead of running `sudo ip netns exec mario` every time._  

  Use the help command in order to determine the correct command syntax.  
  If you have issues the solution is in a comment of the HTML of this page.  

  * `ubutnu@ubuntu:~$` `sudo ip netns exec ip route help`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec ip route add [finish the command here]`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec ip addr 10.0.0.1 dev veth1` TODO: Hide this?

  <!-- ubuntu@ubuntu:~$ ip route add default via 10.0.0.1 -->
#### Rocket Scientist lab:

Lets add some helpers to bash so we can work with network name spaces even more intuitively.

0. Open `/home/ubuntu/.bashrc` in your favorite editor and edit the seciton around `"$color_prompt" = yes` to look like below:

  ``` bash
  namespace=`ip netns identify $$`
  if [ "$color_prompt" = yes ]; then
      if [[ -z "${namespace}" ]]; then
          PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
      else
          PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\] ($namespace) \$ '
      fi
  else
      PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
  fi
  unset color_prompt force_color_prompt
  ```

0. Now re-source the bashrc and re-enter the namespace to see the difference in your prompt.
  
	* `ubutnu@ubuntu:~$` `sudo ip netns exec mario bash`
  * `ubutnu@ubuntu:~ (mario) #` `exit # or press Ctl^D` 

0. Next let's add an alias for `sudo ip netns exec [namespace] [command]`

  * `ubutnu@ubuntu:~$` `alias nse='sudo ip netns exec'`
  * `ubutnu@ubuntu:~$` `nse mario ip link list`

  You can add the alias to your .bashrc file if you want it to persist across sessions


#### Resources

  * [LWN.net - Namespaces in operaton](https://lwn.net/Articles/531114/#series_index)
  * [Scott Lowe - Introducing Linux Network Namespaces](http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/) 


