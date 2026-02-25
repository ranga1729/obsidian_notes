- ASIC hardware - When the OS is designed specifically for a certain set o hardware, they are called, ASIC hardware. 
	 ASIC = Application-Specific Integrated Circuit4
- Console connection = inbound connection
	You have to physically near the switch
- SSH/telnet = outbound connection
	You are connecting to the switch remotely.

****
#### Practical
![[Pasted image 20251221132018.png]]
Once the switch is up, 
Execute, `erase startup-config` to erase all startup configurations. 
	This will delete the startup-config file.
Then execute, `delete vlan.data`
	This will delete the vlan.dat file. 
Execute `reload` to reload the switch.

How to configure a password between the user EXSEC mode and privilege mode, 
1. Go to the global config mode
	`configure terminal`
2. `enable password class` to set a password
	"class" is the password
3. or `enable secret cisco` to set a password
	"cisco" is the password
4. Exit up to the user mode

> [!NOTE] Password vs. Secret
> password supports the older version and the newer versions only support secret

The `password` are saved into a file called "running.config". 
This saves into the RAM. So when you reboot the switch, it get erased. 
(volatile memory)

The `secrete` will be saved into a file called "startup.config".
This is a non-volatile memory. 

To view the running.cofig file, 
	`show running-config`

even though the running-config is non-volatile, you have to manually save the running-config if you want to retain the configs. 
![[Pasted image 20251221130043.png]]
- The Password is in plain text.
- Secrete is a md5 hash
#### How to save the running-config ?
1. not-recommended method
```bash
copy running-config startup-config
```
This will copy the content of the running config to the startup config. 
2. recommended
```
write
```
This will save the running-config into the startup config. 

#### Change the hostname
1. Boot the switch
2. `enable` to enter privilege mode
3. `config terminal` to enter global config mode
4. `hostname <new-name>` to change the host name.
#### Configure an IP for the switch
No point of assigning an IP for a switch. but if you need to access the switch online(telnet/SSH) and that's the only use case of an IP of a switch.
1. got to the global config mode. 
```bash
enable
config terminal
```
2. Then select the interface vlan 1.
```cisco
interface vlan 1
```
This will change your mode to `config-if` sub-configuration mode. 
3. Select `ip address <ip> {<subnet mask>}`

To remove the IP,
`no ip address`
This will remove the IP.

If you want to change the IP directly, 
Execute the same command `ip address <new ip> {<subnet mask>}` with the new IP. This will override/replace the previous values. 

Now we have to up the vlan 1,
```bash
no shutdown
```
Cisco doesn't have a command to turn it on. 
So we say "no shutdown" it works as a negation of the shutdown. 

- running-config save all the commands we are executing. 
- in the config mode, you can't execute `show running-config`
	Solutions, 
	1. Go back to privilege mode and execute `show running-config`
	2. Execute `do show running-config` in the config mode. This allows you to run privilege mode commands in the config mode. 

#### How to enable telnet ?
1. Execute `line vty`\
	VTY = virtual terminal
2. Then it asks for  line number <0-15>
	`line vty 0`
3. You have to select the last line number as well. <0-15>
	`line vty 0 5`
Line number means, how to simultaneous connection can have. 
If we select `line vty 0 5` means, 6 connection can be run simultaneously. 
4. Set a password for the connection 
	`password <password>`
5. Execute `login`
	Now you can connect vis telnet.
 