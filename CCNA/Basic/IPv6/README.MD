# 🌐 IPv6 Fundamentals

### 1. Why IPv6 Exists (The Problem)
The Internet was built on IPv4, which uses a 32-bit addressing scheme providing roughly **4.3 billion addresses**. With the explosion of mobile devices, IoT (Internet of Things), and global internet access, these addresses have completely run out.

To delay this exhaustion, technologies like **NAT (Network Address Translation)** were created. While NAT solved the immediate shortage, it introduced complexity, overhead, and broke the fundamental **end-to-end communication** model of the internet. IPv6 was designed to permanently solve the address shortage and restore direct peer-to-peer connectivity.

---

### 2. How IPv6 Works (The Architecture)
IPv6 drops the old dotted-decimal format in favor of a much larger, cleaner structure:
*   **128-bit Address Space:** Provides $3.4 \times 10^{38}$ unique addresses (enough to assign trillions of IPs to every single grain of sand on Earth).
*   **Hexadecimal Notation:** Formatted as 8 groups of 4 hexadecimal digits (hexatet or quartets) separated by colons (e.g., `2001:0db8:85a3:0000:0000:8a2e:0370:7334`).

#### Address Compression Rules:
To make these long addresses manageable, IPv6 uses two strict compression rules:
1.  **Omit Leading Zeros:** Any leading zeros in a quartet can be dropped. (`0db8` becomes `db8`).
2.  **Double Colon (`::`):** A single contiguous string of all-zero quartets can be replaced with a double colon. *Note: This can only be done once per address to avoid ambiguity.*

*   **Original:** `2001:0db8:0000:0000:0000:0000:1428:57ab`
*   **Compressed:** `2001:db8::1428:57ab`

---

### 3. Types of IPv6 Addresses
Unlike IPv4, which relies heavily on broadcasts, IPv6 does not have broadcast addresses. Instead, it categorizes addresses into three main types:

*   **Unicast (One-to-One):** Uniquely identifies a single interface on an IPv6-enabled device.
    *   **Global Unicast Address (GUA):** Publicly routable on the internet (similar to public IPv4). Typically starts with the prefix `2000::/3`.
    *   **Link-Local Address (LLA):** Required on every IPv6 interface. Used only for communication within the same local network segment (not routable). Automatically generated or manually configured; always starts with the prefix `fe80::/10`.
    *   **Unique Local Address (ULA):** Similar to private IPv4 addresses (RFC 1918). Used for internal routing within a site or company. Starts with the prefix `fc00::/7` (commonly `fd00::/8`).
    *   **Loopback Address:** Used by a host to send a packet to itself. Written as `::1/128`.
*   **Multicast (One-to-Many):** Used to send a single IPv6 packet to multiple destinations simultaneously. Always starts with the prefix `ff00::/8`.
    *   `ff02::1` -> All IPv6-enabled nodes on the link.
    *   `ff02::2` -> All IPv6 routers on the link.
*   **Anycast (One-to-Nearest):** Assigned to multiple devices providing the same service. The network routes the packet to the nearest device configured with that address.

---

### 4. IPv4 vs. IPv6 Comparison

![img ipv4]()

![img ipv6]()

#### Header Optimization (Why IPv6 is faster)
While the IPv6 address is 4 times larger than IPv4, its packet header is significantly streamlined. 

![IPv4 Header]()
![IPv6 Header]()

*   **Fixed Size:** The IPv6 header has a fixed size of 40 bytes. This allows hardware application-specific integrated circuits (ASICs) in modern routers to process packets much faster.
*   **Removed Fields:** IPv6 removed fields like *Header Length*, *Identification*, *Flags*, *Fragment Offset*, and *Header Checksum*. Routers no longer need to recalculate the checksum at every hop, drastically reducing processing overhead and latency.

---

### 5. IPv4 and IPv6 Coexistence Techniques

Since migrating the entire global internet overnight is impossible, IPv4 and IPv6 must coexist seamlessly. The IETF developed three primary transition strategies to allow these two protocols to communicate and operate on the same infrastructure:

#### 1. Dual-Stack
This is the most common and recommended deployment strategy. In a Dual-Stack network, devices and routers are configured to run **both IPv4 and IPv6 protocols simultaneously**. 
*   **How it works:** If a destination website supports IPv6, the client uses its IPv6 stack. If the destination only supports IPv4, the client falls back to its IPv4 stack. They run completely independent of each other on the same physical wire.

#### 2. Tunneling
Tunneling is used when you have two IPv6 networks that need to communicate with each other, but they are separated by an older, **IPv4-only infrastructure** (like a legacy ISP core).
*   **How it works:** The edge router takes the entire IPv6 packet and **encapsulates** it inside a standard IPv4 header. The packet travels across the IPv4 network disguised as regular IPv4 traffic. When it reaches the remote destination edge router, the IPv4 header is stripped away (decapsulated), leaving the original IPv6 packet intact.

#### 3. Translation (NAT64)
Translation is a last-resort compatibility method used when an **IPv6-only device needs to communicate directly with an IPv4-only device** (or vice-versa) without any dual-stack or tunneling support.
*   **How it works:** A specialized network device (usually a firewall or border router) performs **NAT64**. It translates the actual packet headers, converting the IPv6 header into an IPv4 header on the fly. Because it alters the packet layers, it adds processing overhead and is only used when absolutely necessary.

---

### 6. 🛠️ Lab Experience: Cisco Router Configuration (EVE-NG)

To demonstrate the theoretical concepts in practice, I built a basic topology in **EVE-NG** to configure IPv6 unicast routing, static GUA allocation, custom LLA assignment, and Stateless Address Autoconfiguration (SLAAC) on a Cisco router.

#### Topology & EVE-NG Setup
Below is the laboratory environment used for this validation:

![img topology]()

#### Step-by-Step CLI Configuration
1. Enabled IPv6 routing globally.
2. Initialized IPv6 processing on the interface (`ipv6 enable`) — this automatically generates an EUI-64 Link-Local address before we even assign a global one.
3. Configured the Global Unicast Address (GUA) and a custom Link-Local Address (LLA) for easier gateway identification.

![img config ipv6]()

---

### 7. Address Allocation: SLAAC vs. DHCPv6

In IPv6, devices don't always need a traditional DHCP server to get an IP address. There are three main methods for dynamic assignment, which are managed by flags sent in the **Router Advertisement (RA)** messages:

1. **SLAAC Only (Stateless):** The router provides the prefix, and the client creates its own Interface ID. No server tracks who has what IP.
2. **SLAAC with Stateless DHCPv6:** The router provides the prefix (SLAAC) for the IP address, but directs the client to a DHCPv6 server *only* to get additional options (like DNS servers or domain names).
3. **Stateful DHCPv6:** The router tells the client to get all information from a DHCPv6 server. The server centrally manages and leases the IPv6 addresses, exactly like DHCP in IPv4.

#### Key Differences:

| Feature | SLAAC (Stateless) | Stateful DHCPv6 |
| :--- | :--- | :--- |
| **Server Requirement** | None (Requires an IPv6 Router sending RAs) | Dedicated DHCPv6 Server/Router role |
| **Address Generation** | Prefix from Router + Interface ID (EUI-64 or Random) | Assigned directly by the pool on the server |
| **State Tracking** | No central registry (Stateless) | Server maintains a binding table of leases (Stateful) |
| **Additional Info (DNS)** | Requires RFC 8106 support on older devices | Naturally provides DNS, NTP, and domain details |
| **CLI Implementation (Client)** | `ipv6 address autoconfig` | `ipv6 address dhcp` |

---

### 8. Verification & Connectivity Testing

##### Testing Connectivity (Ping):
When pinging a Global Unicast Address (GUA), a standard ping works normally because the router can look up the destination prefix in its global routing table. 

However, because Link-Local Addresses (LLA) are not routable and the same `fe80::/10` prefix exists on *every* single IPv6 interface, the Cisco IOS does not know which physical link to use. If you attempt a standard ping to an LLA, it will fail with an IOS syntax error:

##### Testing Connectivity (Scenario 1 - Ping Fails):

![img ping erro]()

Even after correcting the syntax by adding the output interface (output GigabitEthernet0/0), the ping command executes but results in a 0% success rate (.....). This happens because the neighbor router (Router2) has not yet been configured with IPv6, making its interface completely blind to this traffic.

By capturing the link inside EVE-NG using Wireshark, we can visually confirm this Layer 3 failure: Router1 floods the wire with ICMPv6 Echo Request packets, but no ICMPv6 Echo Reply ever returns from the unconfigured neighbor.

![img wireshark ping erro]()

##### Testing Connectivity (Scenario 1 - Ping Fixed):

To fix the environment, we access Router2 and initialize IPv6 on its link-facing interface using the ipv6 enable command (which automatically generates its Link-Local address fe80::2 via EUI-64) or by assigning it manually.

Once Router2 becomes IPv6-aware, re-running the ping from Router1 now results in a flawless 100% success rate (!!!!!):

![img ping]()

Checking our updated Wireshark capture confirms a healthy, active network state. The packet analyzer now beautifully displays the perfect pairing of every transmitted ICMPv6 Echo Request immediately followed by its corresponding ICMPv6 Echo Reply:

![img wireshark ping certo]()

---

### 📚 Standards & Official Documentation (RFCs)
For deeper technical reference, the core behavior and architectures of IPv6 are governed by the following Internet Engineering Task Force (IETF) standards:

*   **RFC 8200:** Internet Protocol, Version 6 (IPv6) Specification *(Updates the original RFC 2460, defining the standard core protocol)*.
*   **RFC 4291:** IP Version 6 Addressing Architecture *(Defines how IPv6 addresses are structured and classified)*.
*   **RFC 4862:** IPv6 Stateless Address Autoconfiguration *(Governs the behavior of SLAAC used in the autoconfig process)*.

---

