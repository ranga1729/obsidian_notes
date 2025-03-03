1. The tool must identify each removable drive connects to the computer. 
2. Dashboard must have a master button to enable or disable auto backup.
 If auto backup is enabled, tool must actively looking for the newly connected removable devices and check if they are initialized as backup drives or not. if it detects a initialized drive, backup will be automatically performed for necessary devices.
 If disabled, it will not perform auto backups. But it will scan the device.
3. Dashboard must shows a button to see a list of removable devices initialized by this computer.
	- If they are connected, they will be shown as green color box. 
	- If they are not connected, they will be shown in red color box. 
	- Both connected and disconnected boxes must show a status indicator if they are updated or not.
		green - file system and the devices back up is synced(no changes happened to the tracking files after the last back up), 
		blue/rotating - backup in process(device is connected and backup process is ongoing), 
		red - backup required(tracking files has been changed after the last backup)
	- Each drivers must show a list of files/destinations tracked by the device.
1. Each removable device, initialized as a backup drive must has a option to enable/disable auto backup. (if disabled, an button must be appear to sync.)
2. There must be a button to initialize a new drive as a easy backup drive. 
	When a driver is initialized as a backup drive, a hidden folder will be created in the device. 
	This folder include all details including, (ex: .git folder)
		computer Id,
		device Id,
		Tracking files(directories)
	- Computer Id : Computer has a unique ID. 
	- Device Id: Each removable device has n unique ID.
	- Both computer ID and all computer IDs will be stored in the installation location of the tool. It helps the tool to identify if a devices is initialized by this computer or not.
	- Computer Id and the it's device ID will be stored in the hidden folder in the device. 
	- These ID will be compared with each other to identify devices and computers.
6. 