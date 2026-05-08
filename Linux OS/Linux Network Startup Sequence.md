
1. Firmware / bootloader stage
   *  BIOS/UEFI initializes hardware enough to boot.
   *  Bootloader loads the Linux kernel and initramfs.
    
1. Kernel initializes buses and discovers devices
   * During boot, the kernel enumerates buses like PCI/PCIe.
   * The kernel detects network controllers (Ethernet, Wi-Fi adapters, etc.) as PCI devices.
  
3. Driver loading  
    There are two common cases:
	    * Driver built into the kernel:
	      The driver is already present and initializes immediately.
	    * Driver is a module:
	      The kernel emits a uevent.
		    * `udevd` receives it.
		    * `udevd` may invoke `modprobe`.
		    * `modprobe` loads the matching kernel module.
		    The Important Nuance
		    * `udevd` itself does not “drive” the hardware.
		    * The actual hardware support always comes from the kernel driver module.
    

4. Network interface creation
   * Once the driver successfully initializes the hardware, the kernel registers a network interface:
     * `eth0`
     -  `wlan0`
     - `enp3s0`
     -  etc.
   * The interface appears in the kernel networking subsystem (`netdev` layer).
    

4. Interface naming
   * udevd` rules (or systemd predictable naming rules) may rename interfaces.
   * Example:    
    - old style: `eth0`
    - predictable naming: `enp2s0`
        

6. Network management layer  
    Now userspace networking tools take over. This may be:
    *  `NetworkManager`
    * `systemd-networkd`
    *  `ifupdown`
    *  `connman`
    *  others
    These tools configure the interface.

7. Link establishment / authentication  
    This depends on interface type.
    For Ethernet:
    *  Usually no “handshake” in the Wi-Fi sense.
    *  The physical link simply comes up when connected.
    * There may be:
      *  802.1X authentication in enterprise networks
      *  PPPoE in some ISP setups

	For Wi-Fi:
		* A supplicant like `wpa_supplicant` is typically involved.
		*  It:
		    *  scans networks
		    *  associates with the access point
		    *  performs WPA/WPA2/WPA3 authentication/key exchange
	    NetworkManager` often controls `wpa_supplicant` underneath.



8. IP configuration  
    After layer-2 connectivity is established:
    * DHCP client may start:
      * `dhclient`
      * `dhcpcd`
      * `systemd-networkd`’s internal DHCP client
      *  NetworkManager’s internal DHCP support

	This obtains:
	* IP address
	* subnet mask
	* gateway
	* DNS servers
	 
	 from the DHCP server (often the router).

So the refined chain is:

```text
PCIe enumeration
    ↓
kernel detects NIC
    ↓
kernel/udev loads driver
    ↓
driver registers network interface
    ↓
udev/systemd may rename interface
    ↓
NetworkManager (or similar) configures it
    ↓
(wifi only) wpa_supplicant authenticates/associates
    ↓
DHCP client acquires IP configuration
```

One especially important correction:

- DHCP is not responsible for authentication.
    
- DHCP only assigns network configuration after link/authentication is already successful.