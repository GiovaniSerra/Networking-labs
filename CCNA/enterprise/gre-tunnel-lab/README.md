
# Objective
This lab demonstrates the implementation of a GRE (Generic Routing Encapsulation) tunnel between two non-directly connected routers (R1 and R3) over an OSPF underlay network.

The goal is to enable communication between loopback interfaces using an overlay tunnel while traffic physically traverses an intermediate router (R2).

## Topology
![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Intermediate/GRE-tunnel-lab/gre-lab.PNG?raw=true)

Underlay: R1 - R2 - R3 (OSPF Area 0)  
Overlay: GRE tunnel between R1 and R3  

Loopbacks simulate remote networks:
- R1: 1.1.1.1  
- R3: 3.3.3.3  

## IP Addressing
| Device | Interface  | IP Address     |
|--------|------------|----------------|
| R1     | Fa0/0      | 12.12.12.1/24  |
| R2     | Fa0/0      | 12.12.12.2/24  |
| R2     | Fa1/0      | 23.23.23.2/24  |
| R3     | Fa1/0      | 23.23.23.3/24  |
| R1     | Loopback1  | 1.1.1.1/32     |
| R3     | Loopback3  | 3.3.3.3/32     |
| R1     | Tunnel1    | 13.13.13.1/24  |
| R3     | Tunnel1    | 13.13.13.3/24  |


## Key Configuration

### R1 (GRE Tunnel)
interface Tunnel1

 ip address 13.13.13.1 255.255.255.0
 
 tunnel source FastEthernet0/0
 
 tunnel destination 23.23.23.3

ip route 3.3.3.3 255.255.255.255 Tunnel1

### R3 (GRE Tunnel)
interface Tunnel1

 ip address 13.13.13.3 255.255.255.0
 
 tunnel source FastEthernet1/0
 
 tunnel destination 12.12.12.1

ip route 1.1.1.1 255.255.255.255 Tunnel1


## Underlay Routing (OSPF)
R1 advertises: 12.12.12.0/24
R2 advertises: 12.12.12.0/24 and 23.23.23.0/24
R3 advertises: 23.23.23.0/24

### Routing Design
OSPF is used only in the underlay network
GRE tunnel creates a logical adjacency between R1 and R3
Static routes are used to force traffic into the tunnel
Loopback networks are not advertised in OSPF


## Verification

### Tunnel Reachability
R3#ping 13.13.13.1

!!!!!

Success rate is 100 percent (5/5)



### End-to-End Connectivity
R1#ping ip 3.3.3.3 source 1.1.1.1

Type escape sequence to abort.

Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:

Packet sent with a source address of 1.1.1.1

!!!!!

Success rate is 100 percent (5/5), round-trip min/avg/max = 28/32/36 ms




## Traceroute (Overlay Path)
R1#traceroute 3.3.3.3 source 1.1.1.1
Type escape sequence to abort.
Tracing the route to 3.3.3.3
VRF info: (vrf in name/id, vrf out name/id)
  1 13.13.13.3 24 msec 32 msec *

## Packet-Level Validation (Wireshark)
![image alt](https://github.com/GiovaniSerra/Networking-labs/blob/main/CCNA/Intermediate/GRE-tunnel-lab/Wireshark-GRE-lab.PNG?raw=true)

### Interpretation
This capture confirms:
- GRE encapsulation using IP protocol 47  
- Forwarding decisions in the underlay are based on the outer IP header  
- The original payload is preserved across the tunnel
### Analysis

- **Outer IP Header (Underlay)**
  - Source: 12.12.12.1  
  - Destination: 23.23.23.3  
  - Protocol: 47 (GRE)

- Encapsulated Packet (Overlay)
  - Source: 1.1.1.1  
  - Destination: 3.3.3.3  

## CEF Validation
R1#show ip cef 3.3.3.3

3.3.3.3/32

  attached to Tunnel1

This shows that forwarding decisions use the tunnel interface.

## Key Concepts Demonstrated
- GRE encapsulation over IP (Protocol 47)  
- Underlay vs Overlay network separation  
- Static routing over tunnel interfaces  
- Logical connectivity across non-directly connected routers  
- CEF-based forwarding behavior  

## Design Consideration

The tunnel destination must be reachable via the underlay network.  
If routing to the tunnel destination depends on the tunnel itself, it creates a recursive routing issue and the tunnel will fail.

## Notes/Limitations
- GRE does not provide encryption  
- Tunnel endpoints must be reachable via the underlay  
- Loopbacks are used to simulate remote networks  


## Conclusion
The lab successfully demonstrates how GRE tunnels enable communication between remote networks over an intermediate routed infrastructure, with clear separation between control plane (OSPF) and data plane (GRE encapsulation).
