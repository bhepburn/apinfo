# AP Info - Access Point (AP) Information View for OpenWrt/LuCI

A lightweight monitoring tool for OpenWrt-based access points, developed as an extension for OpenWrt/LuCI. It provides visibility into access points, the devices connected to them, and a framework for executing user-defined scripts on access points without requiring additional software packages to be installed.

This package is expected to be installed on a primary OpenWrt-based routing device that connects to all defined access points to gather information about connected clients and Wi-Fi access points.

**Important:** This package does NOT make any changes to access point configurations. Any configuration changes must be explicitly implemented by the user through custom scripts.
 
The application provides the following features:
- Enabling/disabling device monitoring and presenting device operation parameters
- List of wireless clients connected to monitored devices with Wi-Fi
- Downloading logs from the device, with the ability to reboot and ping the device
- Executing user-created scripts on access points
 
Device monitoring is performed periodically by a cron-executed script. For a device to be monitored, it must be enabled (the "Enabled" option) and have valid access credentials provided. The system executes commands over SSH to retrieve operational data from the access point. A device will be displayed with Offline status if it fails to provide data within two and a half times the defined polling interval (e.g., 12.5 minutes if the polling cycle is set to 5 minutes).

The application provides simple configuration of the device polling interval and selection of displayed columns. For better usability, only necessary columns should be enabled for display.
 
This lightweight tool requires minimal dependencies and has minimal interference with the access points themselves. With periodic monitoring, network issues can be quickly identified.

## Access point authentication

A predefined SSH configuration in `/root/.ssh/config` is created and maintained by the user. This allows the user has complete control to set up their SSH keys and configuration. This allows users some flexibility to install OpenSSH and take advantage of advanced SSH features like ControlPersist to prevent a login on every execution. The script will assume that SSH is configured correctly and will simply connect to the configured host.

Here is an example of how to generate an SSH key:
```
mkdir /root/.ssh
ssh-keygen -t ed25519 -f /root/.ssh/id_ed25519
```

Then users should send the public key content to each access point that is using SSH through options 1 or 2:
```
ssh root@192.168.1.2 "tee -a /etc/dropbear/authorized_keys" < /root/.ssh/id_ed25519.pub
ssh root@192.168.1.3 "tee -a /etc/dropbear/authorized_keys" < /root/.ssh/id_ed25519.pub
```

Here is an example SSH configuration file that will cache the SSH session for 24 hours:
```
Host 192.168.1.2
  User root
  IdentityFile /root/.ssh/id_ed25519
  ControlMaster auto
  ControlPersist 1440m
  ControlPath ~/.ssh/cm-%r@%h:%p
Host 192.168.1.3
  User root
  IdentityFile /root/.ssh/id_ed25519
  ControlMaster auto
  ControlPersist 1440m
  ControlPath ~/.ssh/cm-%r@%h:%p
```

Users should also make sure to update the flash and upgrade options to save the SSH configuration between upgrades. The configuration for this is under **System > Backup / Flash Firmware > Configuration**. When utilizing option 1, here are sample files to configure in the backup configuration:
```
/root/.ssh/config
/root/.ssh/id_ed25519
/root/.ssh/id_ed25519.pub
/root/.ssh/known_hosts
```
 
## User-Defined Scripts

A user-created script can be executed on access points through the `Additional Script` tab. The user script is placed at `/root/apinfo.sh` on the monitoring device (not the access points). When executed, this script runs on each enabled access point via SSH.

**Important:** Any configuration changes to access points (Wi-Fi settings, network configurations, etc.) must be explicitly implemented in your user-defined scripts. This package does not make configuration changes on its own.

To preserve your user-defined script across firmware upgrades or system resets, add `/root/apinfo.sh` to the backup configuration under **System > Backup / Flash Firmware > Configuration** in the OpenWrt web interface.

### Built-in Scripts

The application includes built-in scripts located in `/usr/share/apinfo/scripts/` that can be executed on access points:

- **001-log**: Downloads the last 100 lines of system logs from the access point
- **999-reboot**: Reboots the access point (marked with a warning flag)

Built-in scripts use a special naming convention where the filename prefix (e.g., `001-`, `999-`) determines execution order. Scripts can include metadata:
- `#desc:` - Description shown in the UI
- `#warn` - Marks the script as potentially disruptive (e.g., reboot)

For best practices related to OpenWrt scripting, see the following documentation: [https://openwrt.org/docs/guide-developer/write-shell-script](https://openwrt.org/docs/guide-developer/write-shell-script)

## Installation

This package consists of two components that should be installed on your primary OpenWrt router:

1. **apinfo** - Core monitoring daemon and scripts
2. **luci-app-apinfo** - Web interface for LuCI

Both packages can be built using the standard OpenWrt package build system.

## Configuration

The main configuration file is located at `/etc/config/apinfo` and includes:

- **Global settings**: Polling interval (in minutes), data storage path, and column visibility preferences
- **Host definitions**: Individual access point configurations including IP address, port, authentication method, and credentials

The default polling interval is 5 minutes. Data is stored in `/tmp/apinfo` by default but can be configured to use persistent storage.

## Data Collection

The monitoring system collects the following information from each access point:

- Hostname, model, and MAC address
- Software version (OpenWrt release)
- System load and uptime
- Active wireless channels (2.4GHz, 5GHz, 6GHz)
- Connected wireless clients with details:
  - SSID, MAC address, and IP address
  - Client hostname (from DHCP leases or ARP table)
  - Signal strength (dBm)
  - Connection duration
  - Wi-Fi standard (802.11n/ac/ax/be)
  - Data transfer statistics (RX/TX bytes)

Historical activity data is logged hourly and retained for up to 28 days (672 hours), allowing you to track access point availability over time.
