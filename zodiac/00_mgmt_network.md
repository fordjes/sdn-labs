---
date: "2016-11-27"
draft: true
weight: 00
title: "Lab XX - Zodiac IP Configuration"
---
[Click here to find out more about Alta3 Research's SDN Training](https://alta3.com/courses/sdn)
### ??? - &#x2B50;REQUIRED&#x2B50;

### Lab Objective
The objective of this lab is to...

### Procedure


# Zodiac FX - IP Configuration

### Connecting to a Zodiac FX via Raspberry Pi

* `student@beachhead:~$` `ssh pi@[ip]` - ssh into raspberry pi
* `pi@raspberrypi:~ $` `dmesg | grep -i 'usb'` - find Zodiac FX tty locations 
* `pi@raspberrypi:~ $` `minicom --device /dev/ttyACM#` - connect to Zodiac FX via serial

> Note: To exit a minicom session: `Ctrl+a` + `x`

### First commands on a Zodiac FX

* `Zodiac_FX#` `help`
* `Zodiac_FX#` `config`
* `Zodiac_FX(config)#` `factory reset`
* `Zodiac_FX(config)#` `save`
* `Zodiac_FX(config)#` `exit`
* `Zodiac_FX#`

### Show Baisc Config of Zodiac FX

* `Zodiac_FX#` `show version`
* `Zodiac_FX#` `show status`
* `Zodiac_FX#` `show ports`
* `Zodiac_FX#` `config`
* `Zodiac_FX(config)#` `show config`
* `Zodiac_FX(config)#` `show vlans`
* `Zodiac_FX(config)#` `exit`
* `Zodiac_FX#`

### Show OpenFlow Config of Zodiac FX

* `Zodiac_FX#` `openflow`
* `Zodiac_FX(openflow)#` `show flows`
* `Zodiac_FX(openflow)#` `show status`
* `Zodiac_FX(openflow)#` `exit`
* `Zodiac_FX#`

### IP address configuration of Zodiac FX

Your IP address info:

> Raspberry Pi: `10.<Class #>.<Student #>.8`
> FX #0: `10.<Class#>.<Student#>.9`
> FX #1: `10.<Class#>.<Student#>.10`

* `pi@raspberrypi:~ $` `minicom --device /dev/ttyACM0`
* `Zodiac_FX#` `config`
* `Zodiac_FX(config)#` `set name S<Student#>-FX0`
* `Sn-FX0(config)#` `set ip-address 10.<Class#>.<Student#>.9`
* `Sn-FX0(config)#` `set netmask 255.255.255.0`
* `Sn-FX0(config)#` `set gateway 10.0.0.1`
* `Sn-FX0(config)#` `set netmask 255.240.0.0`
* `Sn-FX0(config)#` `set of-controller 10.<Class#>.<Student#>.4`
* `Sn-FX0(config)#` `set of-port 6633`
* `Sn-FX0(config)#` `save`
* `Sn-FX0(config)#` `exit`
* `Sn-FX0#` `openflow` 
* `Sn-FX0(openflow)#` `enable`
* `Sn-FX0(openflow)#` `exit`
* `Sn-FX0#` `restart`
* `Sn-FX0#` `config`
* `Sn-FX0#` `show config`
* `Sn-FX0(config)#` `exit`
* `Sn-FX0#` **press**: `ctrl+a` + `x`

* `pi@raspberrypi:~ $` `minicom --device /dev/ttyACM1`
* `Zodiac_FX#` `config`
* `Zodiac_FX(config)#` `set name FX1`
* `Sn-FX1(config)#` `set ip-address 10.<Class#>.<Student#>.10`
* `Sn-FX1(config)#` `set netmask 255.255.255.0`
* `Sn-FX1(config)#` `set gateway 10.0.0.1`
* `Sn-FX1(config)#` `set of-controller 10.<Class#>.<Student#>.4`
* `Sn-FX1(config)#` `set of-port 6633`
* `Sn-FX1(config)#` `save`
* `Sn-FX1(config)#` `exit`
* `Sn-FX1#` `openflow` 
* `Sn-FX1(openflow)#` `enable`
* `Sn-FX1(openflow)#` `exit`
* `Sn-FX1#` `restart`
* `Sn-FX1#` `config`
* `Sn-FX1(config)#` `show config`
* `Sn-FX1(config)#` `exit`

* `FX1#` **press**: `ctrl+a` + `x`, **choose:** `yes` to exit

