---
date: "2016-11-23"
draft: false
weight: 160
title: "Lab 06 - ADVANCED - Augmenting bash for working with Network Namespaces"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### MONDAY - &#x1F680;OPTIONAL&#x1F680;

### Lab Objective

The objective of this lab is to give **advanced** students a documented mechanism for creating some bash helpers that allow working with network namespaces to be more intuitive. The following steps are applicable for use in a production environment, which is why we thought it important to include them. This lab is **not required**, and should only be attempted by students who are comfortable changing configuration parameters within the bash environment. ***WARNING:* It is possible to wreck your environment by incorrectly tampering with the *.bashrc* file, therefore this lab should *not* be carelessly copy & pasted!**

### Procedure

0. Start by backing up a copy of the current *.bashrc* file.

    `student@beachhead:~$` `cp /home/student/.bashrc /home/student/old.bashrc` 

0. Open `/home/student/.bashrc` in vim.

    `student@beachhead:~$` `cp .bashrc old.bashrc` 

0. Edit the seciton around `"$color_prompt" = yes` to look like below:

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
  
    `ubutnu@beachhead:~$` `sudo ip netns exec mario bash`

0. Great! Now exit.

    `ubutnu@beachhead:~ (mario) #` `exit # or press Ctl^D` 

0. Next let's add an alias for `sudo ip netns exec [namespace] [command]`

    `ubutnu@beachhead:~$` `alias nse='sudo ip netns exec'`

0. Look how easy our commands have become!

    `ubutnu@beachhead:~$` `nse mario ip link list`

0. You can add the alias to your .bashrc file if you want it to persist across sessions


#### Additional Learning / References

The following is a list of pages we thought might be helpful for our students to know about:

* [LWN.net - Namespaces in operaton](https://lwn.net/Articles/531114/#series_index)

* [Scott Lowe - Introducing Linux Network Namespaces](http://blog.scottlowe.org/2013/09/04/introducing-linux-network-namespaces/) 

* [Interconnecting Namespaces](http://www.opencloudblog.com/?p=66)

* [Which interfaces are connected by a veth pair?](http://blog.abhijeetr.com/2014/06/veth-pair-how-to-know-what-interfaces.html)


