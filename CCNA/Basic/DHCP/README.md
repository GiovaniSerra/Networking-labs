# DHCP Basic Lab
## Overview

This lab demonstrates how to configure a Cisco router as a DHCP server and how a client obtains an IP address dynamically.

The goal is to validate DHCP functionality and understand the dynamic IP assignment process.

## Topology

Single broadcast domain environment.

- Router (R1) acting as DHCP Server
- Layer 2 Switch (SW1)
- Client (PC1)

![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Basic/DHCP/topology.png)

## Lab Environment
- Router: 7206VXR (Dynamips)
- Switch: IOL L2 l2-adipservicesk9-m-15.2-20170202.bin
- PC: IOL L2 (configured as DHCP client via switchport)

## IP Addressing
- Network: 192.168.1.0/24
- Default Gateway: 192.168.1.254
- DHCP Range: 192.168.1.101 – 192.168.1.253
- DNS Server: 8.8.8.8

## Configuration

### R1 (DHCP Server)

enable
configure terminal
hostname R1

interface fa0/0
 ip address 192.168.1.254 255.255.255.0
 no shutdown

ip dhcp excluded-address 192.168.1.1 192.168.1.100

ip dhcp pool DHCPLAB
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.254
 dns-server 8.8.8.8
 lease 7
end

### PC1 (DHCP Client)

enable
configure terminal
hostname PC1

interface e0/1
 no switchport
 ip address dhcp
end

## Verification

### DHCP Binding Table (R1)

show ip dhcp binding

### Expected result:

IP address assigned to the client
MAC address associated with the lease

![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Basic/DHCP/ip%20dhcp%20binding.png)

### Client IP Address (PC1)

Verify that the client received a valid IP address from the 192.168.1.0/24 network.

![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Basic/DHCP/dhcp.png)

## Connectivity Test

ping 192.168.1.254

## Expected result:

Successful replies

![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Basic/DHCP/ping.png)

## DHCP Process (Packet Capture)

The DHCP process follows the DORA sequence:

Discover
The client broadcasts a request looking for a DHCP server
Offer
The server responds with an available IP address
Request
The client requests the offered IP
Acknowledge
The server confirms the assignment

Packets 154–159 show the full DORA process using Transaction ID 0x16a8.

![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Basic/DHCP/dhcp-dora-wireshark.png)

The DHCP Discover is sent from 0.0.0.0 to 255.255.255.255 using UDP port 68 → 67, indicating the client does not yet have an IP address.

## DHCP DORA Sequence

This capture highlights the complete DHCP DORA process between the client and the DHCP server.

Discover
The client broadcasts a request (0.0.0.0 → 255.255.255.255) to locate available DHCP servers
Offer
R1 responds with an available IP address from the configured pool
Request
The client requests the offered IP address, confirming its selection
Acknowledge (ACK)
The DHCP server finalizes the lease and assigns the IP configuration to the client

All packets share the same Transaction ID, ensuring they belong to the same DHCP exchange.

![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Basic/DHCP/dora-sequence.png)
