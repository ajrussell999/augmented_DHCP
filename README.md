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

To describe conceptually the automated software upgrade process for Customer Premises equipment (CPE) fiber optic modems manufactured by Connection Technology Systems (CTS) for triple play service providers. CTS CPEs terminate fiber to the home internet connectivity, and divide traffic between internet browser data, digital Internet Protocol Television (IPTV), and telephony.

This document does not describe exact configuration values to be used within a particular network, rather provides a sample configuration from an Internet Services Consortium (ISC) Dynamic Host Configuration Protocol (DHCP) server.

This document does not specify recommended versions hardware, or versions of software and associated releases.

### 2. Summary

The automated software upgrade of CTS CPE modems is accomplished using an appropriately configured DHCP server and a Trivial File Transfer Protocol (TFTP), or a File Transfer Protocol (FTP) server, to download the latest version of software to the DHCP client; which in this case is the CTS CPE modem.

Once downloaded to the CPE modem, the software requires the CPE to be rebooted in-order to take effect. The CPE modem may either be activated automatically after download (immediate reboot) or activated by a manual reboot via (SNMP) or user intervention.

Whilst not within the scope of this document, it should be noted that a similar process can be used for the automatic download of new configuration files to CTS modems.

### 3.	Automated software upgrade via DHCP Vendor-Options   

The conceptual description of the automated software upgrade process is described from two perspectives, namely that of the DHCP server and the CTS CPE modem. The sample DHCP-server config provided in this document is from a laboratory network, an ISC DHCP Server, version 3.1.1.   

#### 3.1	DHCP server process

The decision processes of the DHCP-server is made upon the receipt of a DHCP discover packet from a CTS modem. Upon bootup, the CPE has no IP address and sends a discovery packet to the DHCP server in anticipation of receiving a reply with an IP address offered. The CTS CPE sends a discover packet which contains an option 60 field, vendor-class-identifier. The DHCP server upon examining the vendor class identifier, identifies the client as a CTS CPE, and constructs an offer packer with information required by the CTS CPE to validate if the current software version is the latest. Data encapsulated within the offer packet, sent in reply, is constructed through several steps described in Table 1. The information encapsulated within the offer is written to an optional packet field; option 43 sub-options. The CTS CPE replies to the offer packer with a request packet to confirm to the DHCP server that it is happy with the contents of the offer packet. The DHCP server replies to the CTS CPE with an ACK packet to confirm mutual happiness.

**Table 1.** DHCP Server offer process

|Step|  Description    |   
|:------:|------|   
| 1 | CTS modem sends a DHCP Discover.  This typically occurs on reboot or on a WAN linkup event occurring|
| 2 | DHCP Server depending on configuration will make a vendor ID check to confirm what type of device is requesting an IP address. Vendor Class Identifier (Option 60) can be used by DHCP servers to identify the vendor and functionality of a DHCP client. The information is a variable length string of characters or octets which has a meaning specified by the vendor of the DHCP client |
| 3 | For non CTS modems take appropriate action depending on the DHCP server configuration |
| 4 | In reply to the DHCP-client discover packet. Send a relevant DHCP offer packet for the CTS modem including DHCP option 43 sub-options. |   

The sub-options of option 43 offered by the DHCP-server to the client are listed in Table 2.   
Sub-options 7 and 8 are omitted since these options are used to update the config file. Since the management vlan 4090, must be configured on the WAN interface of the CPE by default in order to reach the management network and DHCP server at power up. These two option codes have been omitted for this automated firmware update solution since CPE configuration is achieved in the laboratory network by Simple Network Management Protocol (SNMP).   


**Table 2.** DHCP-server offer option 43 sub-options  

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


Internet service providers have different types of customers with different data bandwidth needs. Generally business customers require high data bandwidths than domestic customers. To cater for both segments of their business, ISPs install high bandwidth CPEs at business premises. The high bandwidth CPEs use different software and therefore must be identified as a business CPE model in order to have the correct software credentials examined by the DHCP server, and for an IP address to be offered from a separate IP address pool. An address pool being a range of IP addresses specified for a particular purpose or type of client. 

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
**Figure 3.** Automated CPE software FTP upgrade via augmented DHCP   

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


### 4. ISC DHCP-server sample configuration file dhcpd.conf   

The ISC DHCP-server configuration text is split into four sections, global, class ESH-3105, class FS-0900E, and default class. The four sections are listed below.  

#### 4.1 DHCP.conf global
First the global section of the dhcp.conf file.

```sh
# dhcpd.conf
# Sample configuration file for ISC dhcpd
# default-lease-time 600; 600 seconds if not specified by DHCP-client
# max-lease-time 7200 seconds; Maximum IPv4 address lease time (2 hours)
# ddns-update-style interim; enable dynamic dns updates globally.
 ddns-update-style interim;

## allow bootp; #commented out since bootp protocol is not recommended.

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
 authoritative;

# Use <log-facility local 7;> to send dhcp log messages to a different log # file (n.b. must also hack syslog.conf to complete the redirection).
log-facility local7;

#IPv4 24bit subnet and netmask 
subnet 10.5.5.0 netmask 255.255.255.0 {
}   
```  
#### 4.2 class ESH-3105   
The second section of the dhcp.conf file contains class ESH-3105, the parameters for the ESH-3105 business CPE.   


```sh
# class ESH-3105 for Business CPE
# Business CPE model ESH-3105, options space variables types
# options 7 & 8 relevant only for DHCP-client config downloads 
# options 7 & 8 not required Software downloads
option space ESH-3105;

  option ESH-3105.protocol code 1 = unsigned integer 8; 
  option ESH-3105.server-ip code 2 = ip-address; 
  option ESH-3105.server-login-name code 3  = text; 
  option ESH-3105.server-login-password code 4 = text; 
  option ESH-3105.firmware-file-name code 5 = text; 
  option ESH-3105.firmware-md5 code 6 = string;
  option ESH-3105.option code 9 = unsigned integer 16;

# Conditional: Inspect dhcp-client discover packet 
# IF vendor-class id = ESH-3105 
# THEN dhcp-client = CTS Business CPE model 3105
class "ESH-3105"{
	match if (option vendor-class-identifier = "ESH-3105");

    vendor-option-space ESH-3105;

# class ESH-3105 FTP options
    option ESH-3105.protocol 0;

# class ESH-3105 Software upgrade FTP server IP address
    option ESH-3105.server-ip 10.5.5.250;

# class ESH-3105 Software upgrade FTP server CPE login username
    option ESH-3105.server-login-name "3105";

# class ESH-3105 Software upgrade FTP server CPE login password 
    option ESH-3105.server-login-password "abc123ftp";

# class ESH-3105 Software upgrade FTP server CPE latest software filename
    option ESH-3105.firmware-file-name ESH-3105_FW_1_02_00_20081006;

# class ESH-3105 latest software MD5 hash
    option ESH-3105.firmware-md5 c1:70:59:24:be:c6:44:a3:aa:62:ba:d8:e1:c2:06:1b;

# class ESH-3105 IPv4 address options
    option ESH-3105.option 1;
    }

# class ESH-3105 Business CPE IPv4 address pool 
# lease address range permitted for subnet, from 5 to 50 
  subnet 10.5.31.0 netmask 255.255.255.0 {
  pool {
  range 10.5.31.5 10.5.31.50;
  allow members of "ESH-3105"; 
  }

# Default ESH-3105 IP address lease time 10 minutes 
# 600 seconds if the DHCP-client does not specify
  default-lease-time 600;

# Maximum IP address lease time 2hours (7200s) 
  max-lease-time 7200;

# ESH-3105 class, IPv4 24bit subnet mask and broadcast address
  option subnet-mask 255.255.255.0;
  option broadcast-address 10.5.31.255; 

# IP address of router on DHCP-client (ESH-3105)network, DHCP Relay agent
  option routers 10.5.31.250; 

# enable dns updates
  ddns-update-style interim;

# domain name (unregistered domain used in lab test)
  option domain-name "option-vendor-class-identifier-test.org";
  authoritative; 
  }
  ``` 
  #### 4.3 class FS-0900E   
  
  The third section of the dhcp.conf file contains class FS-0900E, which contains the parameters for the domestic CPE.   
  
  ```sh
# Class FS-0900E for domestic CPE
# Domestic CPE model ESH-2109, options space variables types
  option space FS-0900E;

  option FS-0900E.protocol code 1 = unsigned integer 8;
  option FS-0900E.server-ip code 2 = ip-address; 
  option FS-0900E.server-login-name code 3  = text; 
  option FS-0900E.server-login-password code 4 = text; 
  option FS-0900E.firmware-file-name code 5 = text; 
  option FS-0900E.firmware-md5 code 6 = string;
  option FS-0900E.option code 9 = unsigned integer 16;

# Conditional IF: DHCP-client discover packet, vendor-class id = FS-0900E 
# THEN DHCP-client = CTS Business CPE model 2109
class "FS-0900E"{
	match if (option vendor-class-identifier = "FS-0900E");

    vendor-option-space FS-0900E;

# class FS-0900E (ESH-2109) FTP options
    option FS-0900E.protocol 0;

# class FS-0900E Software upgrade FTP server IP address
    option FS-0900E.server-ip 10.5.5.250; 

# class FS-0900E Software upgrade FTP server CPE login username
    option FS-0900E.server-login-name "2109";

# CPE ESH-2109 Software upgrade FTP server CPE login password 
    option FS-0900E.server-login-password "abc1234ftp";

# CPE ESH-2109 Software upgrade FTP server CPE latest software filename
    option FS-0900E.firmware-file-name 2108-2109SERIES_FW_1_06_10_ALU_20080808;

# CPE ESH-2109 latest software MD5 hash
    option FS-0900E.firmware-md5 c1:70:59:24:be:c6:44:a3:aa:62:ba:d8:e1:c2:06:1b;

# class FS-0900 IPv4 address options
    option FS-0900E.option 1;
    }

# class FS-0900E Domestic CPE IPv4 address pool 
# lease address range permitted for subnet, from 9 to 90 

  subnet 10.5.21.0 netmask 255.255.255.0 {
  pool {
  range 10.5.21.9 10.5.21.90;
  allow members of "FS-0900E";
  } 

# class FS-0900E Default ESH-2109 IP address lease time 10 minutes 
# 600 seconds if the DHCP-client does not specify
 default-lease-time 600;

# Maximum IP address lease time 2hours (7200s) 
  max-lease-time 7200;

# class ESH-0900E, IPv4 24bit subnet mask and broadcast address
  option subnet-mask 255.255.255.0;
  option broadcast-address 10.5.21.255; 

#IP address of router on DHCP-client (ESH-2109)network, DHCP Relay agent
  option routers 10.5.21.250; 

# class ESH-0900E enable dns updates
  ddns-update-style interim;

# class ESH-0900E domain name (unregistered domain used in test)
  option domain-name "option-vendor-class-identifier-test.org";

# class ESH-0900E official DHCP server for the local network
  authoritative; 
  }
  ```   
 
#### 4.4 default class - no vendor option identifier   

The fourth section of the dhcp.conf file contains the default configuration parameters for dhcp-clients with no vendor option identifier.   

```sh
#
#DEFAULT class: subnet for DHCP-clients with unmatched or NO vendor id
#DEFAULT IPv4 address pool, range permitted for subnet, from 1 to 127
  subnet 10.5.5.0 netmask 255.255.255.0 {
  pool {
  range 10.5.5.1 10.5.5.127;
  } 
 
#Clients with unmatched or no vendor id, default lease time 10 minutes
#600seconds if the DHCP-client does not specify
  default-lease-time 600;

#Maximum IP address lease time 2 hours, for clients with no vendor-id
  max-lease-time 7200;

#default DHCP class IPv4 24bit subnet mask and broadcast address
  option subnet-mask 255.255.255.0;
  option broadcast-address 10.5.5.255; 

#IP address of router on DHCP-client DEFAULT network, DHCP Relay agent
  option routers 10.5.5.250; 

#enable dns updates
  ddns-update-style interim;

#domain name (unregistered domain used in lab test)
  option domain-name "option-vendor-class-identifier-test.org";

# DHCP server is the official DHCP server for the local network
  authoritative; 
  }

```   



[DHCP_SERVER_VENDOR]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/dhcp_server_option_actions.png
[DHCP_SERVER_VENDOR_FLOW]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/flow-chart_dhcp_vendor_options_id_ip_pool.png
[DHCP_SW_FTP]:https://github.com/ajrussell999/augmented_DHCP/blob/master/images/dhcp-clinet_vendor_options.png   

