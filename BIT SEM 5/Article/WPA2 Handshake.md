![[Pasted image 20250912150338.png]]
Wi-Fi is based on the IEEE 802.11 protocols(802.11a/b/g/n/ac/ax/...)
This defines the architecture, MAC and Physical Layer specifications for Wireless LANs(WLANs).
https://ieeexplore.ieee.org/document/10979691

A brief history Wi-Fi security,
1. WEP(Wired Equivalent Privacy)
	The original standard introduced in 1999.
	It's now considered completely insecure due to significant vulnerabilities that allows hackers to crack the key easily. 
	Not used in modern networks.
2. WPA(WiFi Protected Access)
	Introduced in 2003 as a temporary fix for WEP's vulnerability. 
	Major feature was the usage of dynamic key. 
	But still had vulnerabilities. 
3. WPA2(WiFi Protected Access 2)
	Introduced in 2004 and became the industry standard over a decade.
	It uses the stronger Advanced Encryption Standard(AES) and more secure handshake protocol(We will deep dive into this from this article).
	While it has a known vulnerability against KRACK attacks, it remains robust with a strong passwords. 
	Mostly used standard.
4. WPA3(WiFi Protected Access 2)
	The latest and most secured standard introduced in 2018. 
	WPA3 address WPA2's weaknesses by offering stronger protection against offline dictionary attacks and providing individual data encryption on public network. 
	Despite it's benefits, the transition from WPA2 to WPA3 is still ongoing.

while learning about Wi-Fi, I narrowed down my search on WPA2 security since it's the most common encryption method used in our day-to-day Wi-Fi. 

Let's see what's happens underneath the hood when you click on the "Connect" button on you device. 

1. When scanning for a Wi-Fi network, there are two methods available,
	- 