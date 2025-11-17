brief explain
history
how it works
usage info of airdrop
conclusion

Airdrop is a wireless file-sharing feature built into all of apple's major operating systems like iOS, iPadOS, macOS and visionOS.

Airdrop was introduced to apple ecosystem in two major phases and enhanced over the years. 
- Phase 1
Airdrop was first introduced on July 20, 2011, as a feature in Mac OS X Lion(10.7)
The original version only support transferring files between mac computers and wasn't available for iPhones or iPads.
- Phase 2
Then the airdrop start to support iPhones with the launch of iOS 7 in September 2013. This feature bridged the gap between computers and mobile devices. 

How Airdrop works ?
Airdrop is a combination of Bluetooth and Wi-Fi.

1. Discovery
When Airdrop is enabled, your device uses Bluetooth  Low Energy(BLE) to scan for nearby Apple devices that also have Airdrop turned on. This Bluetooth broadcast contains a hashed, temporary device identity, not your real device name for privacy concerns. (Only devices within 10 meter can discover each other)
2. Secure connection setup
Once a device is found and you choose to send a file, the two device establish a peer-to-peer Wi-Fi connection. This is not through your router or the internet, instead it creates a direct wireless link(WiFi direct/AWDL(Apple Wireless Direct Link)).
If both devices are on the same Wi-Fi network, Airdrop still prefers a direct connection instead of using the router.

3. Encrypted Transfer
The data us encrypted end-to-end using TLS \(same level of security as HTTPS websites). Files doesn't go to any server or cloud, its a device-to-device sharing feature. 

4. Receiver Approval 
Depending on AirDrop settings, the receive may:
- Accept or decline the file
- Automatically receive if the sender is in contacts and contact only mode is active. 