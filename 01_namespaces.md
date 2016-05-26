
0. Create a new network namespace

  * `ubuntu@ubuntu:~$` `sudo ip netns add mario`
  * `ubuntu@ubuntu:~$` `ip netns list`
  * `ubutnu@ubuntu:~$` `ls /var/run/netns`

0. Create virtual Ethernet pair

  * `ubuntu@ubuntu:~$` `sudo ip link add veth0 type veth peer name veth1`
  * `ubuntu@ubuntu:~$` `ip link list`

0. Add a virutal Ethernet to the namespace

  * `ubuntu@ubuntu:~$` `sudo ip link set veth1 netns mario`
  * `ubutnu@ubuntu:~$` `ip link list`

  Where did the veth1 go?
  What namespace are we displaying with the `ip link list` command?

0. Display the links inside the namespace

  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario

0. "Enter" into a namespace

  * `ubutnu@ubuntu:~$` `sudo ip netns exec mario bash`
  * `ubutnu@ubuntu:~$` `ip addr`
  * `ubutnu@ubuntu:~$` `ip link list`
  * `ubutnu@ubuntu:~$` `exit # or press Ctl^D` 

0. From inside the namespace, use `ip` to add `10.0.0.1` ip address `veth1`

Use the help command in order to determine the correct command syntax.
If you have issues the solution is in a comment of the HTML of this page.

  * `ubutnu@ubuntu:~$` `sudo ip netns exec ip addr help`
  * `ubutnu@ubuntu:~$` `sudo ip netns exec ip addr [finish the command here]`

  <!-- ubuntu@ubuntu:~$ ip addr add 10.0.0.1 dev veth1 -->

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
  * `ubutnu@ubuntu:~$` `exit # or press Ctl^D` 

0. Next let's add an alias for `sudo ip netns exec [namespace] [command]`

  * `ubutnu@ubuntu:~$` `alias nse='sudo ip netns exec'`
  * `ubutnu@ubuntu:~$` `nse mario ip link list`

You can add the alias to your .bashrc file if you want it to persist across sessions

