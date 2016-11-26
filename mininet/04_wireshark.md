---
date: "2016-11-25"
draft: false
weight: 200
title: "Lab xx - Using Wireshark to Capture OpenFlow v1.3 Traffic"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is

### Procedure

0. Take a moment to clean up your remote Desktop. For now, close all other terminal spaces or windows you might have open.

0. Open a new terminal, then *cd* to the home student directory

  `student@beachhead:/$` `cd`

0. Launch Wireshark.

  `student@beachhead:~$` `sudo wireshark &`

0. Select **Capture > Options**

0. In the *Wirshark - Capture Interfaces* screen that pops up, *left-click* on the **any** interface.

0. In the *Enter a capture filter ...* box, type `port 6653`.

  - **Q1: What is running on port 6653?**
    - A1: The port of the control interface.

0. Click the Start button.

0. Start a Mininet basic topology with connection to a 'remote' controller

  `student@beachhead:~$` `sudo mn --controller=remote,ip=127.0.0.1,port=6653`

0. Download the python-code for our Ryu controller, name it *ss13.py*.

  `student@beachhead:~$` `wget -O ss13.py https://raw.githubusercontent.com/osrg/ryu/master/ryu/app/simple_switch_13.py`

0. Start a Ryu controller with the file *ss13.py*.

  `student@beachhead:~$` `ryu-manager ss13.py`

0. Run the *pingall* command in Mininet.

  `mininet>` `pingall`

0. Stop the Wireshark capture by *left-clicking* the red square *stop* button.

0. In the *Display filter...* box, type: `openflow_v4` and then press *Enter* to apply the filter.

0. In your resulting capture, click around to find the following types of OpenFlow packets.

0. Find the first packet in your OpenFlow capture, and answer the following questions:

  - **Q1: What is the first message in your OpenFlow capture?**
    - A1: HELLO
  - **Q2: In which direction is the *HELLO* it sent?**
    - A2: Who contacts who first?
  - **Q3: Based on the OpenFlow, the Controller contacted the switch first! Well, that's wrong. Continue on to find out why.**

0. Change your *Display filter...* to: *((openflow_v4) or (tcp.flags.syn == 1 && tcp.flags.ack == 0))*

  - **Q1: What does the above filter do?**
    - A1: TCP sessions are always preceeded by a *SYN* message, the above filter will let us see that.
  

0. Find the second packet in your OpenFlow capture, and answer the following question.

  - **Q1: 

0. Find a packet labeled *OFPT_ECHO_REQUEST*. *Left-click* the packet.

0. Open the OpenFlow 1.3 header.

  >
  If you're lost, see the screenshot below.
  
  
  
0. Answer the following quesiton based on what you're seeing displayed:

  - **Q1: What is the type of message is this?**
    - A1: Type 2
  - **Q2: What is this message measuring?**
    - A2: It doesn't appear to be carrying any payload. So this is simply measuring latency. 

0. *Right-click* on the packet, and choose *Set/Unset Time Reference...*.

0. The difference between the *REF* and next number, is the time to Send / Recieve the ECHO REQUEST & ECHO REPLY.

0.
