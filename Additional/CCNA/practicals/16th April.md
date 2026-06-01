Connect the router and the switch with a RJ45.
Connect 2 PCs to the switch.

Have a serial connection with the router using one computer. 
![[Pasted image 20260419042534.png]]
If the router asked to load the initial configurations: select no.

Then enable the router. (user mode --> privilege mode)
Then erase the initial configurations and confirm it.
Then remove any vlan setups
```cicso
enable

//to erase the startup configuration
erase startup-config

// to remove any pre set vlan data
delete vlan.dat

show version
```

Then connect to console cable to the switch.
enable it. 
```cisco
enable
reload
```

Change back to the router
change your hostname
change the banner message(its like an startup message)
then end the configure mode
and exit
```cisco
//
configure terminal
hostname R1

banner motd #UOM#
end
exit
```

In your router.
check your interfaces
```
enable

show ip interface brief
//or "show ip interface"
```
![[Pasted image 20260419050156.png]]

Then assign an IP address
```
configure terminal
interface GigabitEthernet0/0/0
ip address 192.168.2.100 255.255.255.0
no shutdown
exit
```
![[Pasted image 20260419051324.png]]
Then exit from the configure mode and check the IP interfaces.
```
exit
show ip interface brief
```
![[Pasted image 20260419051513.png]]

![[Pasted image 20260419052842.png]]
Now change your console to the switch
Assign an IP to the switch
```
enable
configure terminal
interfave vlan1
ip address 192.168.2.200 255.255.255.0
no shutdown
exit
```
![[Pasted image 20260419052404.png]]

Then assign 192.168.2.10 for PC0 and 192.168.2.20 for PC1 and set the default gateway to the switch(192.168.2.200)

![[Pasted image 20260419053432.png]]
Try pinging each PCs.
![[Pasted image 20260419054142.png]]

Now enable SSH in your Router(R1)
Connect the console to the switch
```
enabel
configure temrinal
hostname SW1
ip domain-name uom.lk
username ccna secret class
username ccna privilege 15
```
Now we have created a user in the switch
Then you can enable SSH
```
crypto key generate rsa
```
![[Pasted image 20260419055843.png]]
Then wait for the message,
![[Pasted image 20260419060000.png]]

Then enable SSH version 2.
then open vty lines
```
ip ssh version 2
line vty 0 15
transport input ssh
login local
```
![[Pasted image 20260419060733.png]]

Now you can login to the router via PC0/1 using an SSH client(putty`).
Host: 192.169.2.100 (Router's IP)
username: ccna
password: class

Then create another network with 192.168.1.0 255.255.255.0
![[Pasted image 20260419063000.png]]
Now configure routing.
