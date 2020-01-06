## DHCP-VENDOR-OPTIONS applied to triple-play internet service provider for augmented DHCP automatic CPE software upgrade   
---   

Author: Andrew John Russell   
Data: 7.11.2008   

### Abreviations   
   

|Abbreviation|      |   
|:------|:------|   
|CPE | Customer Premesis Equipment |   
|CTS | Connection Technology Systems |
|DHCP| Dynamic Host Configuration Protocol |   
|FTP | File Transfer Protocol | 
|FW | Firmware |
|IPTV | Internet Protocol Television |
|SNMP | Simple Network Management Protocol |
|SW | Software |   
|TFTP | Trivial File Transfer Protocol |


### 1.	Scope and purpose   

To describe conceptually the automated software upgrade process for Customer Premises equipment (CPE) fiber optic modems manufactured by Connection Technology Systems (CTS) for triple play service providers. CTS CPEs  terminate fiber to the home internet connectivity, and divide traffic between internet browser data, digital Internet Protocol Television (IPTV), and telephony.

This document does not describe exact configuration values to be used within a particular network, rather provides a sample configuration from an Internet Services Consortium (ISC) Dynamic Host Configuration Protocol (DHCP) server.

This document does not specify recommended versions hardware, or versions of software and associated releases.

### 2. Summary

The automated software upgrade of CTS CPE modems is accomplished using an appropriately configured DHCP server and a Trivial File Transfer Protocol (TFTP), or a File Transfer Protocol (FTP) server, which provides the latest version of software to the DHCP client; in this case the CTS CPE modem.

Once downloaded to the CPE modem, the software requires the CPE to be rebooted in-order to take effect. The CPE modem may either be activated automatically after download (immediate reboot) or activated by a manual reboot via (SNMP) or user intervention.

Whilst not within the scope of this document, it should be noted that a similar process can be used for the automatic download of new configuration files to the CTS modem.

### 3.	Automated software upgrade via DHCP Vendor-Options   

The conceptual description of the automated software upgrade process is described from two perspectives, namely that of the DHCP server and the CTS CPE modem. The sample config provided in this document is from a laboratory network is from an ISC DHCP Server, version 3.1.1.   

#### 3.1	DHCP server process

The decision processes of the DHCP server is made upon the receipt of a DHCP discover packet from a CTS modem. Upon bootup, the CPE has no IP address and sends a discovery packet to the DHCP server in anticipation of receiving a reply with an IP address offered. The CTS CPE sends a discover packet which contains an option 60 field, vendor class identifier. The DHCP server upon examining the vendor class identifier, identifiers the client as a CTS CPE, and constructs an offer packer with information required to by the CTS CPE to validate if the current software version is the latest. Data encapsulated within the offer packet sent in reply, is constructed through a several steps described in Table 1. The information encapsulated within the offer is written to an optional packet field, namely option 43 parameters. The CTS CPE replies to the offer packer with a request packet to confirm to the DHCP server that it is content with the contents of the offer packet. The DHCP server replies to the CTS CPE with an ACK packet to confirm mutual happiness.

|Step|  Description    |   
|:------:|------| 
| 1 | CTS modem sends a DHCP Discover.  This typically occurs on reboot or on a WAN linkup event occurring|
| 2 | DHCP Server depending on configuration will make a vendor ID check to confirm what type of device is requesting an IP address. Vendor Class Identifier (Option 60) can be used by DHCP servers to identify the vendor and functionality of a DHCP client. The information is a variable length string of characters or octets which has a meaning specified by the vendor of the DHCP client |
| 3 | For non CTS modems take appropriate action depending on the DHCP server configuration |
| 4 | Send a relevant DHCP offer packet for the CTS modem including DHCP option 43 parameters. The DHCP server includes the following vender specification option 43 to respond to Fiber Switch. 
1. Option 43: Protocol (0: TFTP or 1: FTP) 
2. Option 43: IP (TFTP or FTP server) 
3. Option 43: User (Server login name) 
4. Option 43: Password (Server login password) 
5. Option 43: Filename (Software image) 
6. Option 43: MD5 Code (Software image MD5 code) 
7. Option 43: Filename (Configuration image) 
8. Option 43: MD5 Code (Configuration image MD5 code) 
9. Option 43: 16 Bits Option (Bit 0: Urgency Bit 1-15: Reserve)
|  





![DHCPD_VEND][DHCP_SERVER_VENDOR]   
**Figure 1.** DHCP-server vendor-options process and CPE packet exchange   



[DHCP_SERVER_VENDOR]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/dhcp_server_option_actions.png


