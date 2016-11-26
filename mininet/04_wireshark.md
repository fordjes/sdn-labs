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

0. Start a ryu controller with the below file saved as ss13.py

  `student@beachhead:~$` `wget -O ss13.py https://raw.githubusercontent.com/osrg/ryu/master/ryu/app/simple_switch_13.py`
  `ryu-manager ss13.py`

0. Run the *pingall* command in Mininet.

  `mininet>` `pingall`

0. Stop the Wireshark capture by *left-clicking* the red square *stop* button.

0. In the *Display filter...* box, type: `openflow_v4` and then press *Enter* to apply the filter.

0. In your resulting capture, click around to find the following types of OpenFlow packets.

0. Find the first message in your OpenFlow capture, answer the following questions:

  - **Q1: What is the first message in your OpenFlow capture?**
    - A1: HELLO
  - **Q2: In which direction is the *HELLO* it sent?**
    - A2: This message can be sent in either direction.
  - **Q3: **

0. 
