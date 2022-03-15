# Wired_client_simulation_Catalyst9300
Wired endpoint simulation for testing on Cat-9300
As networks are deployed remotely, it is quite a challenge to simulate endpoints from  remote locations.
We would try to use the native app-hosting tools to deploy a wired end point on a catalyst 9300 switche and simulate the wired endpoints.
The first step is to create a docker image, we have used a linux ubuntu OS to create a wired endpoint on the switch and have the connectivity checked.
The first step is to download the an existing docker image and create a tar file out of it. App-hosting supports tar based images.
 
 
 
To enable app-hosting on C9k  platforms, refer the config guide belo:


Docker image location:
First, We would need to download the image to the usbflash of the switch.
ftp://10.104.55.25/netub.tar 

9300#dir usbflash1:
Directory of usbflash1:/

12      -rw-        278953472   Jul 7 2021 11:27:00 +00:00  netub.tar
11      drwx            16384   Jul 6 2021 06:24:30 +00:00  lost+found
1835009  drwx             4096   Aug 6 2020 16:33:03 +00:00  iox_host_data_share


Switch config for Docker networking:

9300#show run interface  appGigabitEthernet 1/0/1

interface AppGigabitEthernet1/0/1
switchport mode trunk

app-hosting appid ub       client1 -vlan 1021
app-vnic AppGigabitEthernet trunk
  vlan 1021 guest-interface 0
app-hosting appid ub2      client2 -vlan 1022
app-vnic AppGigabitEthernet trunk
  vlan 1022 guest-interface 0
app-hosting appid ub3     client2 -vlan 1024
app-vnic AppGigabitEthernet trunk
  vlan 1024 guest-interface 0

Switch config for enabling Docker container:
Ensure that the appid in the networking is matching with the one specified in the install  command below.

app-hosting install appid ub4 package usbflash1:netub.tar
app-hosting activate appid ub4
app-hosting start appid ub4

To connect to the Docker conatiner:

9300l#app-hosting connect appid ub4 session /bin/bash

root@3c41f7773925:/# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
139: sss@if140: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 52:54:99:99:00:00 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.10.66/27 brd 192.168.10.95 scope global sss
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:99ff:fe99:0/64 scope link
       valid_lft forever preferred_lft forever
141: eth0@if142: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:dd:a2:39:98 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 9.40.50.155/24 brd 9.40.50.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ddff:fea2:3998/64 scope link
       valid_lft forever preferred_lft forever

