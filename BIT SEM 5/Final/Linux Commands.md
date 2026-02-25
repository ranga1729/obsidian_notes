### **General File and Directory Management**

- **`ls`**: Used to **list and inspect directory contents**.
    - **Options**: `-a` (show all files, including hidden), `-l` (long format to show author/permissions).
- **`mkdir`**: Used to **create a new directory**.
- **`cp`**: Creates a **copy** of a file (e.g., `cp file1 file2` results in two separate files).
- **`mv`**: Used to **move or rename** files.
    - **Options**: `-f` (force).
- **`ln`**: Makes a file name an **additional reference (link)** to an existing file.
- **`cat`**: Used to **display the contents** of a file in the terminal (e.g., `/etc/passwd`).
- **`chmod`**: Changes **file permissions**.
    - **Options**: `+x` (makes a script executable).
- **`chown`**: Changes **file ownership**.
    - **Options**: `-R` (recursive change for directories).
- **`chgrp`**: Changes the **group ownership** of a file.
- **`umask`**: Used to assign **default permissions** to newly created files.

### **System and Process Monitoring**

- **`ps`**: The main tool for **monitoring current processes**.
    - **Scenarios**: Use `ps aux` to see PIDs, CPU/memory usage, and process status.
- **`top`**: Provides **interactive monitoring** of processes; useful for identifying **runaway processes**.
- **`kill`**: Sends a signal to **terminate a process** using its PID.
    - **Options**: `-9` (SIGKILL) guarantees the process will die.
- **`uptime`**: Reports **system load averages** to check if the system is overloaded.
- **`strace`**: Displays every **system call** a process makes and every signal it receives.
- **`nice`**: Sets the **CPU scheduling priority** when starting a process.
- **`renice`**: Adjusts the **priority** of an already running process.
- **`systemctl`**: All-purpose command for investigating and changing **systemd** status.
    - **Subcommands**: `enable` (activate at boot), `stop` (deactivate immediately), `status` (show logs/status).
- **`journalctl`**: Used to query and filter **systemd journal logs**.
- **`dmesg`**: Used to view **kernel log entries**, particularly useful for boot-time activities.

### **User and Access Control**

- **`sudo`**: Allows users to run commands with **root privileges** while maintaining an audit log.
- **`su`**: Used to **substitute user identity** (switch to another account).
- **`passwd`**: Used by users to **change their passwords**.
- **`useradd` / `adduser`**: Commands used to **create new user accounts**.
- **`usermod`**: Used to modify existing user accounts, such as **locking (`-L`)** or **unlocking (`-U`)** them.
- **`userdel`**: Used to **remove a user account** and its associated files.

### **Networking and Troubleshooting**

- **`ping`**: Checks if a **host is alive** and measures the round-trip travel time.
- **`traceroute`**: Identifies the **gateways (path)** between your host and a destination.
- **`ifconfig` / `ip`**: Used to **configure or verify network interface** settings (IP, netmask).
- **`ethtool`**: Retrieves information about a **network adapter**, such as link speed and duplex mode.
- **`netstat`**: Checks if a service (like SSH) is **up and accepting connections** on specific ports.
- **`tcpdump`**: A **packet sniffing** tool used to capture traffic flowing through an interface.
- **`nmtui`**: A text-based user interface for managing **NetworkManager**.
- **`iptables`**: Manages **firewall rules** and Network Address Translation (NAT).
- **`nslookup` / `dig` / `host`**: Utilities used to **query the DNS database**.

### **Package Management**

- **`apt` / `apt-get`**: High-level package managers for **Debian/Ubuntu** systems.
- **`yum`**: High-level package manager for **Red Hat/CentOS** systems.
- **`rpm`**: Low-level tool to install, verify, and query **RPM packages**.
- **`dpkg`**: Low-level tool to manage **Debian (.deb) packages**.

### **Storage and Filesystems**

- **`lsblk`**: Prints a list of **all disks** the system is aware of.
- **`parted`**: Lists **partition tables** and sizes for every disk.
- **`mkfs`**: Used to **format a filesystem** on a partition.
- **`fsck`**: Used to **check and repair** a filesystem.
- **`mount`**: Used to **attach a filesystem** to the system's file tree.
- **`pvcreate` / `vgcreate` / `lvcreate`**: Commands for managing **Logical Volume Management (LVM)**.
- **`mdadm`**: Used to manage and monitor **software RAID** arrays.

### **Automation and Scripting**

- **`grep`**: A filter command used to **search for patterns** in text.
- **`cut`**: Used to extract specific **fields or columns** from text.
- **`sort`**: Used to **sort lines** of text alphabetically or numerically.
- **`find`**: Searches for files in the directory hierarchy.
    - **Scenario**: `find / -name core 2>/dev/null` hides "permission denied" errors.
- **`git`**: Used for **version control**.
    - **Commands**: `git init` (create repo), `git add` (stage changes), `git commit` (save changes).
- **`docker`**: Manages **containerized processes**.
    - **Subcommands**: `docker ps` (list containers), `docker run` (start new container), `docker exec` (run command in existing container).