# Rocket Scientist lab:

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
  
	* `ubutnu@beachhead:~$` `sudo ip netns exec mario bash`
  * `ubutnu@beachhead:~ (mario) #` `exit # or press Ctl^D` 

0. Next let's add an alias for `sudo ip netns exec [namespace] [command]`

  * `ubutnu@beachhead:~$` `alias nse='sudo ip netns exec'`
  * `ubutnu@beachhead:~$` `nse mario ip link list`

  You can add the alias to your .bashrc file if you want it to persist across sessions


#### Resources

  * [LWN.net - Namespaces in operaton](https://lwn.net/Articles/531114/#series_index)
  * [Scott Lowe - Introducing Linux Network Namespaces](http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/) 
  * [Interconnecting Namespaces](http://www.opencloudblog.com/?p=66)
  * [Which interfaces are connected by a veth pair?](http://blog.abhijeetr.com/2014/06/veth-pair-how-to-know-what-interfaces.html)


