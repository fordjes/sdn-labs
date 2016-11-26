---
date: "2016-11-25"
draft: false
weight: 200
title: "Lab xx - Using Wireshark to Capture OpenFlow v1.3 Traffic"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)

### TUESDAY - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to use Wireshark to capture OpenFlow protocol as used by Mininet. We'll pass custom configurations to Mininet using the Ryu controller, operating on port 6653. Ryu is a component-based software defined networking framework. Ryu provides software components with well defined API that make it easy for developers to create new network management and control applications. Ryu supports various protocols for managing network devices, such as OpenFlow, Netconf, OF-config, etc

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
    - A1: The port of the Ryu control interface.

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
  - **Q2: Who contacts who first?**
    - A2: Based on the OpenFlow, the Controller contacted the switch first! Well, that's wrong. Continue on to find out why.

0. Change your *Display filter...* to: *((openflow_v4) or (tcp.flags.syn == 1 && tcp.flags.ack == 0))*

  - **Q1: What does the above filter do?**
    - A1: TCP sessions are always preceeded by a *SYN* message, the above filter will let us see that.
  - **Q2: How many times does the switch try to initiate a connection to the switch?**
    - A2: Twice. The first connection tests connectivity, the second will begin the OpenFlow session.

0. Look at the OpenFlow *OFPT_HELLO* messages. Use them to answer the following question: 

  - **Q1: What is exchanged in these messages?**
    - A1: Version compatibility.
    
0. Look at the OpenFlow *OFPT_FEATURES_REQUEST* messages. Use them to answer the following question: 

  - **Q1: Which device initiates an *OFPT_FEATURES_REQUEST*?**
    - A1: The controller.
  - **Q2: What was learned in the *OFPT_FEATURES_REPLY*?**
    - A2: Amount of buffers, amount of tables, along with statistics that may be available (flow, table, port, group, IP reasm, queue, port blocked). 

0. Click on the *OFPT_MULTIPART_REQUEST, OFPMP_PORT_DESC*. Answer the following questions:

 - **Q1: Who is sending this message?**
   - A1: The controller.
 - **Q2: What is this message asking?**
   - A2: This is the controller asking the switch to describe all of its ports.
 - **Q3: What message type (number) is this?**
   - A3: 
   
   
0. Click on the *OFPT_MULTIPART_REPLY, OFPMP_PORT_DESC*. Answer the following questions:

 - **Q1: How many ports are there?**
   - A1: 3. Two are two physical ports, and one reserved port.
 - **Q2: What are the ports names?**
   - A2: s1, s1-eth1, s1-eth2 
 - **Q3: What are the ports numbers?**
   - A3: The reserved port is `port no: OFPP LOCAL 0xfffffffe`, whereas the physical ports are `port no: 1`, and `port no: 2`.
 - **Q4: What does `port no: OFPP LOCAL 0xfffffffe` mean?**
   - A4: The reserved port is numbered, `port no: OFPP LOCAL 0xfffffffe`, which is the port on which the switch will get OpenFlow commands from the controller. 
   
0. Click on the first *OFPT_FLOW_MOD*. Answer the following questions:

 - **Q1: What is this message doing?**
   - A1: This is putting a Flow Miss rule in place with ultra low priority. This teaches the switch that the switch will take commands from the controller.
 - **Q2: What is the *priroity* of this message?**
   - A2: 0 (This represents highest priority)
 - **Q3: What is the matching rule?**
   - A3: OFPMT_OXM (1)
 - **Q4: What is the type of action rule?**
   - A4: OFPAT_OUTPUT (0) - Output to switchport
 - **Q5: What port to be associated with the action?**
   - Q5: OFPP_CONTROLLER (0xfffffffd)

0. Click on the first *OFPT_PACKET_IN*. Answer the following questions:

 - **Q1: ?**
   - A1:
   
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
