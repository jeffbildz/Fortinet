Fortinet CLI Cheat Sheet
========================

This cheat sheet provides commonly used CLI commands to manage a FortiGate firewall and integrated or directly managed FortiSwitch devices. Command syntax may vary slightly by FortiOS version. Always consult official Fortinet documentation for detailed instructions.

* * *

1\. Basic System & Device Information
-------------------------------------

Command

Description

`get system status`

Displays firmware version, serial number, and system status.

`get system performance status`

Shows real-time CPU, memory, session counts, etc.

`get system interface`

Shows status and configuration of all interfaces.

`show system interface`

Displays interface settings in config format.

`execute reboot`

Reboots the FortiGate.

`execute shutdown`

Powers off the device (if supported).

* * *

2\. Interface & Network Configuration
-------------------------------------

### 2.1 Viewing/Editing Interfaces

    # Enter interface configuration:
    config system interface
        edit <interface_name>
            set ip <x.x.x.x/24>
            set allowaccess ping https ssh
            set alias "LAN Interface"
            ...
        next
    end
    

### 2.2 DHCP Server (if configured)

    config system dhcp server
        edit <id>
            set interface <interface_name>
            set dns-service default
            set default-gateway <x.x.x.x>
            set netmask 255.255.255.0
            set range <start_ip> <end_ip>
            ...
        next
    end
    

### 2.3 Routing

    # View current routing table:
    get router info routing-table all
    # Add a static route:
    config router static
        edit 1
            set dst <destination_ip/mask>
            set gateway <gateway_ip>
            set device <interface_name>
        next
    end
    

* * *

3\. Firewall Policies & Objects
-------------------------------

### 3.1 Firewall Policy

    # View all firewall policies:
    show firewall policy
    # Create or edit a policy:
    config firewall policy
        edit <policy_id>
            set name "Example Policy"
            set srcintf "port2"
            set dstintf "port3"
            set srcaddr "all"
            set dstaddr "all"
            set action accept
            set schedule "always"
            set service "ALL"
            set nat enable
        next
    end
    

### 3.2 Address Objects

    config firewall address
        edit "Server-1"
            set subnet 192.168.10.50 255.255.255.255
        next
    end
    

* * *

4\. Diagnostics & Troubleshooting
---------------------------------

### 4.1 Real-Time Debugging of Traffic

    # Common flow debug commands:
    diagnose debug reset
    diagnose debug flow filter clear
    diagnose debug flow filter proto <tcp|udp|icmp|...>   # Optional
    diagnose debug flow filter addr <x.x.x.x>            # Optional
    diagnose debug console timestamp enable
    diagnose debug flow show function-name enable
    diagnose debug flow show iprope enable
    diagnose debug enable
    diagnose debug flow trace start 100   # Capture 100 packets
    # -- Watch the output, then stop --
    diagnose debug disable
    

### 4.2 Session & System Diagnostics

    # View all active sessions:
    diagnose sys session list
    # Filter sessions by source or destination:
    diagnose sys session filter src <x.x.x.x>
    diagnose sys session filter dst <x.x.x.x>
    # Clear all sessions (use with caution):
    diagnose sys session clear
    # System process usage:
    diagnose sys top
    

### 4.3 Packet Capture (Sniffer)

    # Basic sniffer example:
    diagnose sniffer packet any "host x.x.x.x" 4 100
    # Syntax:
    # diagnose sniffer packet <interface> "<filter>" <verbosity> <count>
    # - <verbosity> 4 = detailed; 3 = less verbose
    # - <count> = number of packets
    

### 4.4 Connectivity Checks

    execute ping <destination_ip>
    execute traceroute <destination_ip>
    

* * *

5\. Managing Integrated or FortiLinked FortiSwitches
----------------------------------------------------

In many deployments, FortiSwitch is managed via FortiLink (Layer 2 or Layer 3). Below are some key commands for viewing and configuring FortiSwitch devices from the FortiGate CLI. (Exact syntax may differ based on FortiOS/Switch OS versions.)

### 5.1 Viewing FortiSwitch Status & Information

    # List managed FortiSwitches and status:
    get switch-controller managed-switch
    # More detailed info on each FortiSwitch:
    diagnose switch-controller switch-info <fortiswitch_serial>
    

### 5.2 Configuring FortiSwitch Ports via FortiGate

    config switch-controller managed-switch
        edit <fortiswitch_serial>
            config ports
                edit <port_name>  # Example: "port1" or "port2"
                    set description "Server Port"
                    set vlan <vlan_id_or_name>
                    set allowed-vlans <vlan_list>
                    # set trunk enable (if needed)
                next
            end
        next
    end
    

### 5.3 VLANs & VLAN Interfaces (on FortiSwitch)

    config switch-controller vlan
        edit <vlan_id>
            set name "<vlan_name>"
            set vlanid <vlan_id_number>
            set interface "<fortilink_interface_name>"
            # Optional: set ipv4 address if routing
        next
    end
    

### 5.4 FortiLink Interface (on FortiGate)

    # If you need to configure or view the FortiLink interface:
    config system interface
        edit "fortilink"
            set mode dhcp  # or static
            set allowaccess ping https ssh
            set type fortilink
            # Additional settings as needed
        next
    end
    

### 5.5 Switch Interfaces (Soft Switch or Hardware Switch)

    # Example: Combining ports into a hardware or software switch
    config system switch-interface
        edit "lan-switch"
            set member "port2" "port3"
            set vdom "root"
            set span enable/disable
        next
    end
    

* * *

6\. High Availability (HA) & MC-LAG (Bonus)
-------------------------------------------

### 6.1 High Availability on FortiGate

    config system ha
        set mode a-p       # or a-a for Active-Active
        set group-name "HA-Group"
        set password "HA-Password"
        set priority 200
        ...
    end
    

### 6.2 MC-LAG (Multi-Chassis LAG) on FortiSwitch

MC-LAG configuration varies by FortiSwitch model and firmware. High-level steps include configuring two FortiSwitches in an MC-LAG cluster, setting up the inter-switch link, and configuring MC-LAG on the relevant ports.

Refer to FortiSwitch documentation for exact CLI commands and best practices.

* * *

7\. Other Handy Commands
------------------------

Command

Description

`execute backup config flash`

Backup configuration to FortiGate’s flash memory (varies by model).

`execute restore config flash`

Restore configuration from flash.

`execute backup config tftp <filename> <ip>`

Backup configuration to a TFTP server.

`execute restore config tftp <filename> <ip>`

Restore config from TFTP.

`show full-configuration`

Displays the entire config (including default values). This can be large.

`diag test application <process_name> <id>`

Runs specific debug/test commands for daemons (ips, wad, dnsproxy, etc.).

* * *

General Tips
------------

*   **Type-Ahead & ? or Tab:** Type `?` or press Tab to see available commands or complete partial commands.
*   **Show vs. Get:**
    *   **get** = runtime or real-time info (e.g., `get system status`)
    *   **show** = saved configuration in the config file (e.g., `show system interface`)
*   **Use next/end:** In config mode, `next` saves changes to that entry and moves on, `end` saves and exits config mode.
*   **Save Your Config:** FortiGate automatically saves most changes, but verify via `show` commands or the GUI.
*   **Debug with Caution:** Debug outputs can overload the device. Always `diagnose debug disable` when finished.

  Fortinet CLI Management Cheat Sheet body { font-family: Arial, sans-serif; line-height: 1.6; margin: 20px; color: #333; } h1, h2, h3 { color: #0056b3; } pre { background-color: #f8f9fa; padding: 10px; border: 1px solid #ccc; border-radius: 4px; overflow-x: auto; } code { font-family: Consolas, monospace; background-color: #f8f9fa; padding: 2px 4px; border-radius: 4px; } ul { margin-left: 20px; } li { margin-bottom: 5px; }

Fortinet CLI Management Cheat Sheet
===================================

This cheat sheet provides common CLI commands for managing **Fortinet Firewalls**, **Switches**, and **Access Points (APs)**.

* * *

General CLI Basics
------------------

### Enter Configuration Mode:

    config <section>

### End Configuration Mode:

    end

### Save Configuration:

    execute save

### Show Current Configuration:

    show

### Show Full Configuration (including defaults):

    show full-configuration

### Test Connectivity:

*   Ping:
    
        execute ping <ip_address>
    
*   Traceroute:
    
        execute traceroute <ip_address>
    

* * *

FortiGate Firewall
------------------

### System Management

*   Reboot Device:
    
        execute reboot
    
*   Check Firmware Version:
    
        get system status
    
*   Upgrade Firmware:
    
        execute restore <image_file> <tftp_ip>
    
*   Check System Resource Usage:
    
        get system performance status
    

### Interface Management

*   View Interface Configuration:
    
        show system interface
    
*   Configure an Interface:
    
        config system interface
          edit <interface_name>
            set mode static
            set ip <ip_address> <subnet_mask>
            set allowaccess ping https ssh
          next
        end
    
*   Check Interface Status:
    
        get system interface physical
    

### Routing

*   View Routing Table:
    
        get router info routing-table all
    
*   Add a Static Route:
    
        config router static
          edit 0
            set dst <destination_ip> <subnet_mask>
            set gateway <gateway_ip>
            set device <interface_name>
          next
        end
    

### Firewall Policies

*   View All Policies:
    
        show firewall policy
    
*   Create a Policy:
    
        config firewall policy
          edit 0
            set name "Allow_HTTP"
            set srcintf "port1"
            set dstintf "port2"
            set srcaddr "all"
            set dstaddr "all"
            set action accept
            set schedule "always"
            set service "HTTP"
          next
        end
    

* * *

FortiSwitch
-----------

### Switch Management

*   View Managed Switches:
    
        get switch-controller managed-switch
    
*   Authorize a Switch:
    
        config switch-controller managed-switch
          edit <switch_serial_number>
            set switch-id <switch_id>
            set description "Core Switch"
          next
        end
    

### Port Management

*   View Port Status:
    
        execute switch-controller get-ports <switch_serial_number>
    
*   Configure a Port:
    
        config switch-controller managed-switch
          edit <switch_serial_number>
            config ports
              edit <port_number>
                set description "Uplink to Router"
                set native-vlan <vlan_id>
              next
            end
          next
        end
    

### VLAN Management

*   Create a VLAN:
    
        config switch-controller vlan
          edit <vlan_id>
            set vlanid <vlan_id>
            set name "Office_VLAN"
          next
        end
    
*   Assign VLAN to Ports:
    
        config switch-controller managed-switch
          edit <switch_serial_number>
            config ports
              edit <port_number>
                set native-vlan <vlan_id>
              next
            end
          next
        end
    

* * *

FortiAP
-------

### AP Management

*   View Managed APs:
    
        get wireless-controller managed-ap
    
*   Authorize an AP:
    
        config wireless-controller managed-ap
          edit <ap_serial_number>
            set description "Office_AP"
          next
        end
    

### Wireless SSID Management

*   View SSIDs:
    
        get wireless-controller vap
    
*   Create a New SSID:
    
        config wireless-controller vap
          edit "Guest_SSID"
            set ssid "GuestWiFi"
            set vlanid <vlan_id>
            set security "WPA2"
            set passphrase "<password>"
          next
        end
    

* * *

Troubleshooting
---------------

### System Logs

*   View System Logs:
    
        execute log display
    

### Debug Commands

*   Enable Debugging:
    
        diagnose debug enable
    
*   Set Debug Filters (Example for IP):
    
        diagnose debug flow filter addr <ip_address>
    
*   View Debug Output:
    
        diagnose debug console timestamp enable
        diagnose debug flow show console enable
        diagnose debug flow trace start 10
    

* * *

Common Shortcuts
----------------

*   Show IP Routing Table:
    
        get router info routing-table all
    
*   Show Active Sessions:
    
        diagnose sys session list
    
*   Clear ARP Table:
    
        execute clear system arp-table
    

* * *

Best Practices
--------------

1.  Always backup configurations:
    
        execute backup config <tftp|usb> <destination>
    
2.  Use consistent naming conventions for interfaces, VLANs, and SSIDs.
3.  Apply policies in order of priority to optimize performance.
4.  Document all changes for future reference.

* * *

This cheat sheet is a starting point for managing Fortinet devices via CLI. For detailed information, refer to the [Fortinet Documentation](https://docs.fortinet.com/).
