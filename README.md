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

**Table 1.** DHCP Server offer process

|Step|  Description    |   
|:------:|------|   
| 1 | CTS modem sends a DHCP Discover.  This typically occurs on reboot or on a WAN linkup event occurring|
| 2 | DHCP Server depending on configuration will make a vendor ID check to confirm what type of device is requesting an IP address. Vendor Class Identifier (Option 60) can be used by DHCP servers to identify the vendor and functionality of a DHCP client. The information is a variable length string of characters or octets which has a meaning specified by the vendor of the DHCP client |
| 3 | For non CTS modems take appropriate action depending on the DHCP server configuration |
| 4 | Send a relevant DHCP offer packet for the CTS modem including DHCP option 43 parameters. The DHCP server includes the following vender specification option 43 to respond to Fiber Switch |   

The sub-options of option 43 offered by the DHCP-server to the client are listed in Table 2.   


**Table 2.** DHCP Server offer option 43 parameters  

|Sub-option | Description |   
|:------:|:------|   
| 1 | Protocol (0: TFTP or 1: FTP) |   
| 2 | IP (TFTP or FTP server) |   
| 3 | User (Server login name) |   
| 4 | Password (Server login password) |   
| 5 | Filename (Software image) |   
| 6 | MD5 Code (Software image MD5 code) |
| 7 | Filename (Configuration image)  |
| 8 | MD5 Code (Configuration image MD5 code) |
| 9 | 16 Bit Option (Bit 0: Urgency Bit 1-15: Reserve) |   


Internet service providers have different types of customers with different data bandwidth needs. Generally business customers require high data bandwidths than domestic customers. To cater for both sides of the business, ISPs install high bandwidth CPEs at business premises. The high bandwidth CPE use different software and therefore must be identified as a business CPE model in order to have the correct software credentials examined by the DHCP server, and for an IP address to be offered from a different IP address pool. An address pool being a range of IP addresses specified for a particular purpose, which in this case is business clients. 

The process by which the DHCP server differentiates between residential and business customers is illustrated with the flow diagram in Figure 1.

![CPE_TEYPE][DHCP_SERVER_VENDOR_FLOW]   
**Figure 1.** DHCP-server business & domestic CPE type identification and options
  
The DHCP server-client packet exchange is illustrated in Figure 2. The vendor option 43 sub-options being encapsulated in the DHCP-server's offer packet. The the CPE inspects the sub-options in the offer packet, before replying with a request packet. The ACK returned by the DHCP-server confirms client and server mutual happiness with the options assigned.


![DHCPD_VEND][DHCP_SERVER_VENDOR]   
**Figure 2.** DHCP-server vendor-options process and CPE packet exchange   

#### 3.2	CTS CPE DHCP-client process   

The CTS CPE operates as a DHCP-client, and as such receives an IP addresses offer from the DHCP server. The address offered to the CPE depends on the precise model of CPE, since high bandwidth business CPEs receive addresses from a separate IP address range than domestic CPEs. The DHCP server identifies different CPE models from their unique vendor class identifier. Domestic CPEs have a vendor class identifier of FS-0900E, while high bandwidth business CPEs have a vendor class identifier of ESH-3105. The packet exchange between client and server, and the server process steps taken in returning an IP address and option 43 vendor specific options to the CPE DHCP-client are illustrated in Figure 2.  

The DHCP option 43 vendor specific options sent to the CTS CPE inform the CPE of the latest available software for the CPE operating system. The terms software and firmware are used interchangeably in the tables and figures describing the automated software upgrade process. The DHCP server includes the MD5 hash of the latest CPE software in the option 43 options of the offer packet. On receiving the DHCP offer packet, the CPE compares MD5 hash with that of its own. If the MD5 hashes are the same, then the CPE takes no further action, but if they differ, then the software upgrade process is initiated.   

![CPE_FTP][DHCP_SW_FTP]   
**Figure 3.** Automated CPE software upgrade via augmented DHCP   

The software upgrade process is achieved by using option 43 credentials to authenticate an FTP session and download the requested filename. The IP address of the FTP server, the session username and password, and the software file name are contained with the option 43 fields. The software upgrade process and the packet exchange to authenticate the CPE and FTP download the latest software are illustrated in Figure 3. The sequential steps of the upgrade process are listed in Table 3.   

**Table 3.** CTS CPE Software upgrade process     

|Step | Description |   
|:------:|:------|   
| 1 | Based on the DHCP discover the DHCP-server sends back an appropriate offer |   
| 2 | Check the local MD5 File Code Hash value against the one returned in the DHCP offer |   
| 3 | If the checksums are the same, take no further action relating to SW upgrade |   
| 4 | If local checksum differs, use the credentials returned by the DHCP server to attempt a download of the new SW from a TFTP or FTP server |   
| 5 | If the SW download fails, after 3 attempts take no further action until the next DHCP ACK |
| 6 | If the SW download is successful, check the urgency bit   |
| 7 | If urgency bit set - reboot immediately |   
| 8 | If not set, then await manual reboot (user intervention or via SNMP) before activating new SW |   





[DHCP_SERVER_VENDOR]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/dhcp_server_option_actions.png
[DHCP_SERVER_VENDOR_FLOW]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/flow-chart_dhcp_vendor_options_id_ip_pool.png
[DHCP_SW_FTP]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/dhcp-clinet_vendor_options.png   

