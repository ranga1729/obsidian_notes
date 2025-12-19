Two types of networking devices,
1. End devices
	When an end device provide a service : server
	When an end device use a service : client
2. Intermediate devices
	Connect clients and servers

#### Host Roles
Every computer in a network is a host/end device. 
Servers are computers that provide information to end devices.
![[Pasted image 20251207104153.png]]
Ex: 
	Email servers
	Web servers
	File servers
#### Peer-to-Peer
Peer-to-Peer is a type of network design where clients connect to each other directly and provide services.
![[Pasted image 20251207105501.png]]

| Advantages    | Disadvantages        |
| ------------- | -------------------- |
| Easy to setup | No centralized admin |
| Less complex  | Less secure          |
| Low cost      | Less scalable        |
|               | Slow performance     |
#### End Devices
This is where message originates/received. 
![[Pasted image 20251207105949.png]]
LAN : Local Area Network
	At least one intermediate must there there to create a LAN.
Internetwork : Connects multiple LANs.
	In the "Internetwork" area, there are no end devices. 
#### Intermediary Devices
Interconnects end devices. 
Ex: 
	switches,
	wireless access points
	routers
	firewalls

Jobs of intermediary devices,
- Main : management of data
- Regenerate and retransmit data signals. 
- Maintain information about pathways.
- Notify errors and com failures. 
#### Network Media
Carry messages through the network.
Ex:
![[Pasted image 20251207112411.png]]
RJ45 : Registered Jack 45
RJ11 : Telephone Jack

The process of connecting the wire to the port is called "Crimpling" that done done a tools.

**Patch Panel**
![[Pasted image 20251207114804.png]]
![[Pasted image 20251207114902.png]]
There's a special tool called "punch tool" to punch a wire to the Patch Panel. 

**Face Panel**
![[Pasted image 20251207115032.png]]
![[Pasted image 20251207115042.png]]

**Standard Cabling mechanism with Patch Panel and Face Panels**
![[Pasted image 20251207115202.png]]
The cable between Face panel and Patch panel is permanent.

**TDR(Time Domain Reflectometer)**
![[Pasted image 20251207120238.png]]
![[Pasted image 20251207120227.png]]
TLD tool test the quality of the cable and it has lots of other tests. 

**Fiber**
![[Pasted image 20251207124351.png]]
![[Pasted image 20251207124548.png]]
Tow types of fiber cables,
1. Single mode(always yellow)
2. Multi mode(some other color)

Three type of connectors for fiber,
![[Pasted image 20251207124817.png]]
1. ST connector(Circle)
2. SC Connector(Square)
3. LC Connector(Small)

You can't directly connect fiber cables to most of the devices. 
You have to use a SFP(Small Form-Factor Pluggable) module to connect. SFP module decides the speed of the fiber connection. 
- Single mode SFP : has a blue color lock
- Multi mode SFP has a black color lock

**OTDR(Time Domain Reflectometer)**
![[Pasted image 20251207125103.png]]
Used to test fiber cables. 

**VFL(Visual Fault Locator)**
![[Pasted image 20251207125303.png]]
Another tool used to test fiber cable damages. 

