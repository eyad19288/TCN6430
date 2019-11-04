# Lab 1: Configuring a Network
- [Devices Description](#devices)
- [logical (layer 3) diagram](#diagram)
- [Logging In](#login)
- [Tasks](#tasks)
  -  [1. Configure IP Addresses](#ips)
    - [1.1 Linux Router R1](#R1ips)
    - [1.2 Cisco R2 & R3](#R2R3ips)
    - [1.3 Juniper BR](#BRips)
  -  [2. Configure Static Routing](#static)
    - [2.1 R1 Routes](#R1routes)

## <a name="devices"></a> Devices Description

R1 - Linux router running the Quagga routing suite <br>
R2 - Cisco 7206 router <br>
R3 - Cisco 7206 router <br>
BR - Juniper router and border router to the network <br>
WS0 – end workstation that connects to network 0 and functions as DNS, DHCP, NTP, syslog server <br>
WS1 – end workstation that connects to network 1 <br>
WS2 – end workstation that connects to network 2 <br>
WS3 – end workstation that connects to network 3 <br>

## <a name="diagram"></a> Logical (layer 3) diagram
![logical diagram](https://users.cs.fiu.edu/~esj/cnt4504/figs/logical-a.png)

## <a name="login"></a> Logging In
vm server is fluorine.cs.fiu.edu

XX is group number 61-63

4XX00   - BR serial console via telnet protocol un=juniper pw=juniper1 <br>
4XX01   - R1 console via VNC protocol pw=cis2008PW un=root pw=r1password <br>
4XX02   - R2 serial console via telnet protocol enable=cisco <br>
4XX03   - R3 serial console via telnet protocol enable=cisco <br>
4XX04   - arista switch serial console via telnet protocol un=admin pw=arista <br>
4XX10   - ws0 console via VNC protocol vnc pw=cis2008PW un=root pw=aitgXXpw <br>
4XX11   - ws1 console via VNC protocol vnc pw=cis2008PW un=root pw=cis2008PW <br>
4XX12   - ws2 console via VNC protocol vnc pw=cis2008PW un=root pw=cis2008PW <br>
4XX13   - ws3 console via VNC protocol vnc pw=cis2008PW un=root pw=cis2008PW

ws0 for each can be accessed from hosts in the department (like 
margay.cs.fiu.edu) via

ssh cisnlab-ws0-gXX.cs.fiu.edu -l root with password aitgXXpw


For the telnet access to out of band consoles you would 
telnet fluorine.cs.fiu.edu 4XXYY  (XX is group, YY is 
from table above)

For VNC use a vnc client to talk to fluorine.cs.fiu.edu
on the port listed.  VNC is also only available from 
department hosts.  (though if student provides a public 
IP for a home or office machine, it can be allowed off campus
VNC access. )

## <a name="tasks"></a> Tasks
### <a name="ips"></a> 1. Configure IP Addresses
Configure the correct IP address (according to the logical layer 3 diagram) on each interface of each router. View and save the changes made into the running configuration and then view the running configuration. 

#### <a name="R1ips"></a> 1.1 Linux Router R1
use vtysh command to control zebra. vtysh is a shell for FRR daemons. It provides users with a Cisco IOS-like shell to operate quagga-based routers so all router configuration commands must be done from here.

```
[root@r1 ~]# vtysh
r1#
```

Enter configuration mode. Configuration mode allows users to modify the running system configuration.

```
r1# conf terminal
r1(config)#
```

configure each interface with the correct IP address according to the diagram.

```
r1(config)# interface eth0                  - enter interface mode 
r1(config-if)# ip address 10.61.1.129/26    - set ip address 
r1(config-if)# description link to net2     - add a description 
r1(config-if)# link-detect                  - allow zebra to detect changes in link state 
r1(config-if)# quit                         - return to regular config mode 
```

Check that each interface is up and has been configured with the correct IP address.

```
r1# show interface eth0
```

Check the changes made by viewing the running configuration.

```
r1#show running-config
Building configuration...
```

Finally, save the changes made.

```
r1# copy running-config startup-config
Building Configuration…
Configuration saved to /mnt/hda1/quagga/zebrad.conf
Configuration saved to /mnt/had1/quagga/ospfd.conf
[ok]
```

#### <a name="R2R3ips"></a> 1.2 Cisco R2 & R3
Enter enable prompt and from there enter global configuration mode.

```
changeme>enable
password:cisco
changeme#                 - Privileged EXEC (enable) prompt
changeme#config terminal
changeme(config)#         - global configuration mode
```

Change the name of the router to R2.

```
changeme(config)#hostname R2
R2(config)#
```

configure each interface with the correct IP address according to the diagram. <br>
Note: Cisco IOS does not understand slash notation; therefore, the netmask must be entered in dotted quad notation.

```
R2(config)#interface e1/0                            - enter interface configuration
R2(config-if)#ip address 10.61.0.2 255.255.255.0     - set ip addresses
R2(config-if)#no shutdown                            - bring interface up
R2(config-if)#description link to net0               - add a description
R2(config-if)#exit                                   - leave interface configuration mode
```

Check that the interfaces are up and have been configured with the correct IP address.

```
R2#show ip interface brief
```

Check the changes made by viewing the running configuration.

```
R2#show running-config
Building configuration...

Current configuration : 1298 bytes
…
```

Finally, save the changes made.

```
R3#copy run start
Destination filename [startup-config]?
Building configuration...
[OK]
```

#### <a name="BRips"></a> 1.3 Juniper BR
Enter configuration mode.

```
juniper@br> edit
Entering configuration mode

[edit]
juniper@br#
```

Configure the interfaces with the correct IP addresses according to the diagram.

```
[edit]
juniper@br# edit interfaces                                 -move down the hierarchy to edit the interface tree

[edit interfaces]
juniper@br# edit fxp0                                       

[edit interfaces fxp0]
juniper@br# edit unit 0

[edit interfaces fxp0 unit 0]
juniper@br# set family inet address 10.90.0.61/24           -set IP address

[edit interfaces fxp0 unit 0]
juniper@br# set description "connection to ISP peering net" -add a description

[edit interfaces fxp0 unit 0]
juniper@br# show                                            -show changes
description "connection to ISP peering net";
family inet {
    address 10.90.0.61/24;
}

[edit interfaces fxp0 unit 0]
juniper@br# up                                              -move up a level in the hierarchy

[edit interfaces fxp0]
juniper@br# up

[edit interfaces]
juniper@br#
```

Permanently save changes and go back to operational mode.

```
[edit interfaces fxp1 unit 0]
juniper@br# commit and-quit
commit complete
Exiting configuration mode

juniper@br>
```

Check that the interfaces are up and have been configured with the correct IP address.

```
juniper@br> show interfaces terse
```

Check the changes made by viewing the running configuration.

```
juniper@br> show configuration
version 8.0R1.9;
…
```

### <a name="static"></a> 2. Configure Static Routing
At this point we only have directly connected routes. For instance, R2 only knows how to reach 10.61.0.0/24 and 10.61.1.192/30. This means that to obtain full connectivity between all the subnetworks, we must configure static routes for the non-connected networks.  Configure routing so that traffic between BR and networks 1 & 2 will go through R2; traffic between BR and network 3 will go through R3; traffic between R1 and network 0 will go through R2; traffic between R1 and network 3 will go through R3; and traffic between R1 and the default route will take the path of R1 > R2 > BR. Also include default route on BR that points to the ISP and on R1, R2, and R3 include a default route that points traffic to BR. Afterwards check the routing tables and check connectivity.

#### <a name="R1routes"></a> R1 Routes
Set static routes on R1 following the guidelines stated above.
R1 is directly connected to 10.61.1.0/25 (net1), 10.61.1.128/26(net2), 10.61.1.192/30(link between R1 and R2), and 10.61.1.196/30(link between R1 and R3). R2 needs routes to get to network 0, network 3, and default. 

Example:

```
r1(config)# ip route 10.61.0.0/24 10.61.1.193
```

Check the routing table in orer to view the added routes:

```
r1(config)# show ip route
```
