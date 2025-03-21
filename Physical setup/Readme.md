

## **Step-by-Step Guide to Setting Up a Network**

This guide establishes a VLAN-segmented network for a company with four departments—IT (4 computers), HR (4 computers), Sales (5 computers), and Production (8 computers)—totaling 21 client computers, plus 4 printers, 21 VoIP phones, and an AD server. VLANs are mandatory, and a firewall is configured to enforce access restrictions.

---

### **Network Diagram Description**
Imagine a diagram with:
- **Top Layer**: A router/Layer 3 switch (`192.168.x.1` for each VLAN) connected via a trunk link to a 48-port managed switch.
- **Middle Layer**: The switch with ports assigned to VLANs:
  - Ports 1–4: VLAN 10 (IT: 4 PCs, 1 printer)
  - Ports 5–8: VLAN 20 (HR: 4 PCs, 1 printer)
  - Ports 9–13: VLAN 30 (Sales: 5 PCs, 1 printer)
  - Ports 14–21: VLAN 40 (Production: 8 PCs, 1 printer)
  - Ports 22–42: VLAN 50 (VoIP: 21 phones)
  - Port 43: VLAN 60 (Server)
- **Connections**: Trunk ports between switches (if multiple) and to the router, access ports for devices.
- **Labels**: Each cable and port labeled (e.g., “IT_PC1”, “VLAN10_Printer”).

---

### **Step 1: Plan the VLAN Structure**
Define VLANs and IP schemes:

- **VLAN Layout**:
  - **VLAN 10**: IT Department (computers and printer) – `192.168.10.0/24`
  - **VLAN 20**: HR Department (computers and printer) – `192.168.20.0/24`
  - **VLAN 30**: Sales Department (computers and printer) – `192.168.30.0/24`
  - **VLAN 40**: Production Department (computers and printer) – `192.168.40.0/24`
  - **VLAN 50**: VoIP Phones (across all departments) – `192.168.50.0/24`
  - **VLAN 60**: Servers (AD server) – `192.168.60.0/24`

- **Static IP Assignments**:
  - Server: `192.168.60.10`
  - Printers: `192.168.10.20` (IT), `192.168.20.21` (HR), `192.168.30.22` (Sales), `192.168.40.23` (Production)
  - VoIP Phones: `192.168.50.30` to `192.168.50.50`

- **Labeling Note**: Use clear, consistent labels (e.g., “IT_PC1”, “VLAN20_Printer”) on cables, ports, and devices for troubleshooting and maintenance.

---

### **Step 2: Gather Hardware**
Ensure VLAN-capable equipment:
- **1 Server**: 8GB RAM (16GB recommended), 2 CPU cores.
- **Managed Switch**: 48-port switch (e.g., Cisco SG350-48) with PoE.
  - **PoE Note**: Verify the switch’s power budget supports all 21 VoIP phones (typically 15W per phone, 315W total).
- **21 Client Computers**: Ethernet-capable.
- **4 Printers**: Network-capable.
- **21 VoIP Phones**: PoE-supported.
- **Router/Layer 3 Switch with Firewall**: For inter-VLAN routing and firewall rules (e.g., Cisco RV340, Catalyst 3560, or a dedicated firewall like pfSense).

---

### **Step 3: Configure the Server**
Set up the AD server:
1. **Install Windows Server**: Use 2019 or 2022, following the installation wizard.
2. **Assign Static IP**:
   - IP: `192.168.60.10`
   - Subnet Mask: `255.255.255.0`
   - Gateway: `192.168.60.1` (router’s VLAN 60 interface)
   - DNS: `192.168.60.10` (critical for AD domain resolution).
3. **Connect**: Attach to a switch port tagged for VLAN 60 (e.g., port 43).

---

### **Step 4: Connect Devices**
Physically connect and assign VLANs:
- **Port Assignments**:
  - Ports 1–4: VLAN 10 (IT: 4 PCs, printer on port 4)
  - Ports 5–8: VLAN 20 (HR: 4 PCs, printer on port 8)
  - Ports 9–13: VLAN 30 (Sales: 5 PCs, printer on port 13)
  - Ports 14–21: VLAN 40 (Production: 8 PCs, printer on port 21)
  - Ports 22–42: VLAN 50 (VoIP: 21 phones)
  - Port 43: VLAN 60 (Server)
- **Static IPs**:
  - Server: `192.168.60.10`
  - Printers: As listed above.
  - VoIP Phones: `192.168.50.30`–`192.168.50.50`
- **Labeling**: Label each cable and port (e.g., “Sales_PC1_Port9”).

---

### **Step 5: Configure Network Equipment (Including Firewall)**
Set up VLANs, routing, DHCP, and firewall rules:

1. **Switch Configuration (e.g., Cisco SG350)**:
   - Access the switch via web interface or CLI.
   - Create VLANs: `vlan 10`, `vlan 20`, etc.
   - Assign ports as access ports:
     - `interface range gi1-4`, `switchport mode access`, `switchport access vlan 10`
   - Configure trunk ports:
     - For router: `interface gi48`, `switchport mode trunk`, `switchport trunk allowed vlan 10,20,30,40,50,60`
     - For inter-switch links (if multiple switches): Same as above.

2. **Router/Layer 3 Switch Configuration (e.g., Cisco Catalyst 3560)**:
   - Define VLAN interfaces:
     - `interface vlan 10`, `ip address 192.168.10.1 255.255.255.0`
     - `interface vlan 20`, `ip address 192.168.20.1 255.255.255.0`
     - `interface vlan 30`, `ip address 192.168.30.1 255.255.255.0`
     - `interface vlan 40`, `ip address 192.168.40.1 255.255.255.0`
     - `interface vlan 50`, `ip address 192.168.50.1 255.255.255.0`
     - `interface vlan 60`, `ip address 192.168.60.1 255.255.255.0`
   - Enable routing: `ip routing`
   - Set trunk to switch: `interface gi0/1`, `switchport mode trunk`, `switchport trunk allowed vlan 10,20,30,40,50,60`

3. **DHCP Setup** (on server or router):
   - Scopes:
     - VLAN 10: `192.168.10.50`–`192.168.10.100`, Subnet Mask: `255.255.255.0`, Gateway: `192.168.10.1`, DNS: `192.168.60.10`
     - VLAN 20: `192.168.20.50`–`192.168.20.100`, Subnet Mask: `255.255.255.0`, Gateway: `192.168.20.1`, DNS: `192.168.60.10`
     - VLAN 30: `192.168.30.50`–`192.168.30.100`, Subnet Mask: `255.255.255.0`, Gateway: `192.168.30.1`, DNS: `192.168.60.10`
     - VLAN 40: `192.168.40.50`–`192.168.40.100`, Subnet Mask: `255.255.255.0`, Gateway: `192.168.40.1`, DNS: `192.168.60.10`
     - VLAN 50: `192.168.50.51`–`192.168.50.100`, Subnet Mask: `255.255.255.0`, Gateway: `192.168.50.1`, DNS: `192.168.60.10`
   - **DNS Note**: All clients must use `192.168.60.10` as their primary DNS server for AD domain resolution.

4. **Firewall Configuration**:
   - **Objective**: Restrict inter-VLAN traffic to enforce department isolation while allowing necessary access (e.g., IT to all, departments to server).
   - **Procedure (Using Cisco ACLs as an Example)**:
     - Access the router/Layer 3 switch CLI.
     - Define Access Control Lists (ACLs):
       - **Permit IT to All**: Allow IT VLAN to access all VLANs (e.g., for admin purposes).
         - `access-list 101 permit ip 192.168.10.0 0.0.0.255 any`
       - **Permit Departments to Server**: Allow HR, Sales, Production to VLAN 60 only.
         - `access-list 102 permit ip 192.168.20.0 0.0.0.255 192.168.60.0 0.0.0.255`
         - `access-list 103 permit ip 192.168.30.0 0.0.0.255 192.168.60.0 0.0.0.255`
         - `access-list 104 permit ip 192.168.40.0 0.0.0.255 192.168.60.0 0.0.0.255`
       - **Permit VoIP Traffic**: Allow VLAN 50 to SIP server (e.g., external or VLAN 60).
         - `access-list 105 permit ip 192.168.50.0 0.0.0.255 any`
       - **Deny Inter-Department Access**: Block HR, Sales, Production from each other.
         - `access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.30.0 0.0.0.255`
         - `access-list 102 deny ip 192.168.20.0 0.0.0.255 192.168.40.0 0.0.0.255`
         - `access-list 103 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255`
         - `access-list 103 deny ip 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255`
         - `access-list 104 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255`
         - `access-list 104 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255`
       - **Implicit Deny**: Cisco applies a default `deny any any` at the end.
     - Apply ACLs to VLAN interfaces:
       - `interface vlan 10`, `ip access-group 101 in`
       - `interface vlan 20`, `ip access-group 102 in`
       - `interface vlan 30`, `ip access-group 103 in`
       - `interface vlan 40`, `ip access-group 104 in`
       - `interface vlan 50`, `ip access-group 105 in`
     - **Note**: Adjust rules if VoIP requires specific ports (e.g., SIP uses UDP 5060).

---

### **Step 6: Set Up Printers and VoIP Phones**
- **Printers**:
  - Assign static IPs as listed.
  - Connect to respective VLAN ports (e.g., IT printer to port 4, VLAN 10).
- **VoIP Phones**:
  - Assign static IPs (`192.168.50.30`–`192.168.50.50`).
  - Connect to VLAN 50 ports (22–42).
  - Configure SIP settings.

---

### **Step 7: Test the Network**
Verify VLAN and firewall functionality:
- **Within VLANs**: Ping IT PC to IT printer (`192.168.10.20`).
- **Inter-VLAN**: Confirm IT can ping server (`192.168.60.10`), HR cannot ping Production (`192.168.40.x`).
- **Firewall**: Test denied access (e.g., HR to Sales should fail).

---

### **Conclusion**
This setup includes a VLAN-segmented network with a detailed firewall configuration using Cisco ACLs to enforce department isolation. The default gateway for each VLAN (e.g., `192.168.10.1` for VLAN 10) is the router’s interface IP, and all clients use `192.168.60.10` as their DNS server for AD resolution. Labeling, trunk ports, and PoE considerations are emphasized throughout, ensuring a robust foundation for AD deployment. Let me know if you’d like firewall examples for other devices (e.g., pfSense)!
