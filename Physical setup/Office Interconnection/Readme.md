**Office Interconnection: Connecting Tokyo Office to Osaka Office for Centralized Access (Detailed Guide)**

This comprehensive section outlines the complete step-by-step approach to securely connect the Osaka office to the Tokyo office network. This setup ensures that the Osaka site becomes an extension of the Tokyo network, giving Osaka users full access to Tokyo's Active Directory (AD) services, shared resources, printers, and VoIP infrastructure.

---

## Objective:
- Interconnect Tokyo and Osaka offices using a secure site-to-site VPN.
- Allow Osaka users full, seamless access to all services hosted in Tokyo.
- Maintain network segmentation using VLANs and strict firewall policies.

---

## VLAN Layout (Common for Tokyo and Osaka Offices)
To maintain a consistent structure and enable secure traffic control, **both offices use the following VLAN segmentation**:

- **VLAN 10**: IT Department (computers and printer) – `192.168.10.0/24`
- **VLAN 20**: HR Department (computers and printer) – `192.168.20.0/24`
- **VLAN 30**: Sales Department (computers and printer) – `192.168.30.0/24`
- **VLAN 40**: Production Department (computers and printer) – `192.168.40.0/24`
- **VLAN 50**: VoIP Phones – `192.168.50.0/24`
- **VLAN 60**: Servers (e.g., AD Domain Controller, File Servers, RODC) – `192.168.60.0/24`

> This VLAN design is implemented in **both Tokyo and Osaka offices** to allow seamless integration, simplified routing, centralized access control, and standardized GPO deployment.

---

## Step-by-Step Procedure to Connect Tokyo and Osaka Offices

### **Step 1: Site-to-Site VPN Design and Implementation**

#### 1.1. **Select VPN-Capable Devices**
Each office requires a firewall/router that supports site-to-site VPN:
- Recommended hardware:
  - Cisco RV340, FortiGate 60F, pfSense, or Ubiquiti EdgeRouter

#### 1.2. **Network Addressing**
- **Tokyo Office**: Uses `192.168.0.0/16` segmented into VLANs as defined above
- **Osaka Office**: Uses the same VLAN segmentation and addressing scheme

#### 1.3. **VPN Configuration**
- **VPN Type**: IKEv2/IPSec
- **Authentication**: Pre-Shared Key (PSK)
- **Encryption**: AES-256
- **Tunnel Parameters**:
  - **Tokyo Side**:
    - WAN IP: `203.0.113.1`
    - Local Subnet: `192.168.0.0/16`
    - Remote Subnet: `192.168.0.0/16`
  - **Osaka Side**:
    - WAN IP: `203.0.114.1`
    - Local Subnet: `192.168.0.0/16`
    - Remote Subnet: `192.168.0.0/16`

> 🔐 **Note**: Because both sites share VLAN IP ranges, care must be taken to avoid conflicts with DHCP, DNS, or server roles. Coordination and centralized role assignment are crucial.

---

### **Step 2: Routing and Firewall Policy Configuration**

#### 2.1. **Static Routing**
- Define bidirectional static routes on both VPN endpoints to allow inter-VLAN traffic via the VPN tunnel.

#### 2.2. **Firewall Rules – Tokyo Office**
- Permit Osaka VLANs (`192.168.10.0/24` to `192.168.60.0/24`) access to:
  - Domain Controller (`192.168.60.10`)
  - Shared file servers
  - VoIP and print services
  - Interdepartmental access based on organizational policy

#### 2.3. **Firewall Rules – Osaka Office**
- Allow outbound access to all Tokyo resources
- Enforce inter-VLAN restrictions per department policy
- Permit full access for Osaka IT staff to manage and monitor Tokyo resources

---

### **Step 3: Active Directory Integration**

#### 3.1. **DNS Configuration**
- All clients (Tokyo and Osaka) should point to the AD DNS server (`192.168.60.10`)
- Optionally deploy a Read-Only Domain Controller (RODC) in Osaka VLAN 60

#### 3.2. **Join Osaka Clients to Tokyo Domain**
- Use the `company.local` domain to join all Osaka systems to the centralized AD

#### 3.3. **Optional: RODC in Osaka VLAN 60**
- Deploy RODC for:
  - Faster user login
  - Local policy application
  - Secure local credential caching

---

### **Step 4: Access to Resources**

- Map shared drives hosted on Tokyo file servers
- Enable printer access across both sites
- Allow VoIP phones to register via VPN if using centralized SIP servers
- Ensure ACLs and GPOs reflect appropriate access per department

---

### **Step 5: Performance Enhancements**

- Apply QoS rules to prioritize voice, DNS, and domain traffic
- Use tools like PRTG or Zabbix to monitor VPN health and inter-office latency
- Label all VLAN ports, trunk links, and inter-switch connections clearly

---

### **Step 6: VLAN-Specific Switch Port Mappings**

| Port Range     | VLAN ID | Department       | Description                         |
|----------------|---------|------------------|-------------------------------------|
| 1–4            | 10      | IT               | 4 Computers + Printer               |
| 5–8            | 20      | HR               | 4 Computers + Printer               |
| 9–13           | 30      | Sales            | 5 Computers + Printer               |
| 14–21          | 40      | Production       | 8 Computers + Printer               |
| 22–42          | 50      | VoIP             | 21 VoIP Phones                      |
| 43             | 60      | Server           | AD Domain Controller                |
| 44–48 (trunk)  | Trunk   | Inter-switch/VPN | Trunk to router/VPN/other switches |

---

### **Step 7: DHCP Relay Agent Setup**

To centralize DHCP service and avoid conflicts between Tokyo and Osaka:

1. **Central DHCP Server**:
   - Located in VLAN 60 (Tokyo) – IP: `192.168.60.10`

2. **Configure DHCP Relay on Each VLAN Interface (Layer 3 Switch/Router)**:
   - Example (Cisco IOS):
     - `interface vlan 10`
     - `ip helper-address 192.168.60.10`
     - Repeat for VLANs 20–50

3. **DHCP Scope Setup**:
   - On DHCP Server (Windows Server):
     - VLAN 10: `192.168.10.100`–`192.168.10.200`
     - VLAN 20: `192.168.20.100`–`192.168.20.200`
     - VLAN 30: `192.168.30.100`–`192.168.30.200`
     - VLAN 40: `192.168.40.100`–`192.168.40.200`
     - VLAN 50: `192.168.50.100`–`192.168.50.200`
   - Set DNS to `192.168.60.10`, Gateway to VLAN interface IP

---

### **Step 8: Visual Diagram Overview**

```
              [Osaka Firewall]                                   [Tokyo Firewall]
            +------------------+                             +------------------+
            | WAN: 203.0.114.1 |=============================| WAN: 203.0.113.1 |
            | VPN: IPSec Tunnel|                             | VPN: IPSec Tunnel|
            +--------+---------+                             +--------+---------+
                     |                                                  |
        +------------+-------------+                    +---------------+--------------+
        | Managed Switch (Osaka)   |                    |  Managed Switch (Tokyo)     |
        +--------------------------+                    +-----------------------------+
        | VLAN 10: IT              |                    | VLAN 10: IT                 |
        | VLAN 20: HR              |                    | VLAN 20: HR                 |
        | VLAN 30: Sales           |                    | VLAN 30: Sales              |
        | VLAN 40: Production      |                    | VLAN 40: Production         |
        | VLAN 50: VoIP            |                    | VLAN 50: VoIP               |
        | VLAN 60: Servers (RODC)  |                    | VLAN 60: Servers (DC, DHCP) |
        +--------------------------+                    +-----------------------------+
```

---

**Office Interconnection: Connecting Tokyo Office to Osaka Office for Centralized Access (Detailed Guide)**

This comprehensive section outlines the complete step-by-step approach to securely connect the Osaka office to the Tokyo office network. This setup ensures that the Osaka site becomes an extension of the Tokyo network, giving Osaka users full access to Tokyo's Active Directory (AD) services, shared resources, printers, and VoIP infrastructure.

---

## Objective:
- Interconnect Tokyo and Osaka offices using a secure site-to-site VPN.
- Allow Osaka users full, seamless access to all services hosted in Tokyo.
- Maintain network segmentation using VLANs and strict firewall policies.

---

## VLAN Layout (Common for Tokyo and Osaka Offices)
To maintain a consistent structure and enable secure traffic control, **both offices use the following VLAN segmentation**:

- **VLAN 10**: IT Department (computers and printer) – `192.168.10.0/24`
- **VLAN 20**: HR Department (computers and printer) – `192.168.20.0/24`
- **VLAN 30**: Sales Department (computers and printer) – `192.168.30.0/24`
- **VLAN 40**: Production Department (computers and printer) – `192.168.40.0/24`
- **VLAN 50**: VoIP Phones – `192.168.50.0/24`
- **VLAN 60**: Servers (e.g., AD Domain Controller, File Servers, RODC) – `192.168.60.0/24`

> This VLAN design is implemented in **both Tokyo and Osaka offices** to allow seamless integration, simplified routing, centralized access control, and standardized GPO deployment.

---

**Office Interconnection: Connecting Tokyo Office to Osaka Office for Centralized Access (Detailed Guide)**

This comprehensive section outlines the complete step-by-step approach to securely connect the Osaka office to the Tokyo office network. This setup ensures that the Osaka site becomes an extension of the Tokyo network, giving Osaka users full access to Tokyo's Active Directory (AD) services, shared resources, printers, and VoIP infrastructure.

---

## Objective:
- Interconnect Tokyo and Osaka offices using a secure site-to-site VPN.
- Allow Osaka users full, seamless access to all services hosted in Tokyo.
- Maintain network segmentation using VLANs and strict firewall policies.

---

## VLAN Layout (Common for Tokyo and Osaka Offices)
To maintain a consistent structure and enable secure traffic control, **both offices use the following VLAN segmentation**:

- **VLAN 10**: IT Department (computers and printer) – `192.168.10.0/24`
- **VLAN 20**: HR Department (computers and printer) – `192.168.20.0/24`
- **VLAN 30**: Sales Department (computers and printer) – `192.168.30.0/24`
- **VLAN 40**: Production Department (computers and printer) – `192.168.40.0/24`
- **VLAN 50**: VoIP Phones – `192.168.50.0/24`
- **VLAN 60**: Servers (e.g., AD Domain Controller, File Servers, RODC) – `192.168.60.0/24`

> This VLAN design is implemented in **both Tokyo and Osaka offices** to allow seamless integration, simplified routing, centralized access control, and standardized GPO deployment.

---

## Step 9: Router ACL Configuration (Tokyo and Osaka)

Access Control Lists (ACLs) help enforce traffic policies at the router level, ensuring only authorized access between VLANs and between sites.

### **Theory: What Are Router ACLs?**
Router ACLs (Access Control Lists) are rule sets used to filter network traffic. They can permit or deny traffic based on source/destination IP, protocol type, and port numbers. ACLs are processed top-down and end with an implicit "deny all" if no rule matches.

Using ACLs, we can:
- Restrict communication between departments
- Allow specific VLANs access to shared resources (like servers or VoIP)
- Ensure security and policy enforcement without needing separate physical networks

### **Example ACLs for Cisco Router**

#### **Permit IT Full Access (VLAN 10)**
```
access-list 101 permit ip 192.168.10.0 0.0.0.255 any
```

#### **Permit HR, Sales, Production to Server VLAN Only (VLANs 20–40 to 60)**
```
access-list 102 permit ip 192.168.20.0 0.0.0.255 192.168.60.0 0.0.0.255
access-list 103 permit ip 192.168.30.0 0.0.0.255 192.168.60.0 0.0.0.255
access-list 104 permit ip 192.168.40.0 0.0.0.255 192.168.60.0 0.0.0.255
```

#### **Block Interdepartmental Access**
```
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 103 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 103 deny ip 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 104 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 104 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
```

#### **Permit VoIP (VLAN 50) to SIP Server (Assume on VLAN 60)**
```
access-list 105 permit udp 192.168.50.0 0.0.0.255 192.168.60.0 0.0.0.255 eq 5060
access-list 105 permit udp 192.168.50.0 0.0.0.255 192.168.60.0 0.0.0.255 range 10000 20000
```

### **Apply ACLs to VLAN Interfaces**
```
interface vlan 10
 ip access-group 101 in

interface vlan 20
 ip access-group 102 in

interface vlan 30
 ip access-group 103 in

interface vlan 40
 ip access-group 104 in

interface vlan 50
 ip access-group 105 in
```

> 🔒 **Note**: Always include a default implicit deny at the end of ACLs. Ensure to monitor logs to avoid over-restriction.

---

## Step 10: pfSense Firewall Rules, NAT, and Inter-VLAN ACL Logic

### **Theory: pfSense as a Firewall and VLAN Gateway**
pfSense is a powerful open-source firewall and router platform used in both SMB and enterprise environments. It allows deep control over traffic between networks (e.g., VLANs), including:
- Stateful packet inspection
- NAT policies
- Detailed firewall rules by interface (VLAN)
- Logging and packet capture for troubleshooting

Unlike router ACLs, pfSense uses a GUI with top-down rule processing per interface, which allows for more flexible and visually manageable configurations.

---

### **10.1: pfSense Inter-VLAN Firewall Rules**
In pfSense, each VLAN interface has its own firewall tab. Define rules per interface.

#### **IT VLAN (VLAN 10) – Allow Full Access**
- Interface: VLAN 10
- Rule:
  - Action: Pass
  - Source: `192.168.10.0/24`
  - Destination: `any`

#### **HR, Sales, Production VLANs – Allow to VLAN 60 Only**
- Interface: VLAN 20, 30, 40
- Rule:
  - Action: Pass
  - Source: Corresponding VLAN subnet (e.g., `192.168.20.0/24`)
  - Destination: `192.168.60.0/24`

#### **Block Interdepartmental Access**
- Add **block rules** above the allow rules for each interface:
  - Source: e.g., `192.168.20.0/24`
  - Destination: `192.168.30.0/24`, `192.168.40.0/24`

> 📌 **Tip**: pfSense rules are evaluated top-down. Place block rules above allow rules.

---

### **10.2: NAT Exceptions**

### **Theory: NAT in Inter-VLAN Communication**
By default, NAT (Network Address Translation) is used to allow private IP addresses to access the internet. However, NAT should not be applied between internal VLANs since it hides the true source IP, breaks security policies, and complicates logging.

#### **Configure NAT Exceptions in pfSense**
- Go to **Firewall > NAT > Outbound**
- Set mode to **Hybrid** or **Manual**
- Add rules to **disable NAT** between VLANs:
  - Source: `192.168.0.0/16`
  - Destination: `192.168.0.0/16`
  - Translation: `Do not NAT`

---

### **10.3: SIP/VoIP Considerations**

#### **Theory: VoIP & NAT Challenges**
VoIP protocols like SIP and RTP dynamically assign ports, which can be disrupted by NAT (port rewriting). To avoid dropped calls or one-way audio:

#### **Best Practices in pfSense**
- Enable **Static Port NAT** for VoIP VLAN to avoid port rewriting
- Enable **SIP ALG (Application Layer Gateway)** only if required by your VoIP provider

---

### **10.4: Logging and Testing**

- Enable logging on **block rules** to audit denied traffic
- Use **Diagnostics > Packet Capture** in pfSense to inspect live traffic flow
- Review logs in **Status > System Logs > Firewall** to fine-tune rules

Let me know if you’d like a web GUI walkthrough for pfSense setup or example screenshots!





## Updated Network Architecture Summary
- **Both Tokyo and Osaka** now use identical VLANs (`192.168.10.0/24` to `192.168.60.0/24`)
- **VPN tunnel** provides a secure, encrypted link between sites
- **DNS and AD services** centralized in Tokyo, with optional RODC in Osaka
- **Production and Sales departments in Tokyo** are fully included in VLANs 30 and 40 respectively
- **IT in Osaka** has full administrative access across both networks
- **Switch port mapping and DHCP relay setup** ensure scalability and centralized management

Let me know if you’d like VLAN-aware DNS conditional forwarding or router ACLs next!

