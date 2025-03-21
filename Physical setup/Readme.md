Below is a detailed, step-by-step guide for physically setting up a network environment in preparation for an Active Directory (AD) installation. This is tailored for a company with four departments—IT (4 computers), HR (4 computers), Sales (5 computers), and Production (8 computers)—totaling 21 client computers, along with printers and VoIP phones. Each step is explained thoroughly to ensure clarity and precision.

---

## Detailed Step-by-Step Physical Setup Procedure for Active Directory Environment

This guide provides a comprehensive approach to physically setting up the network infrastructure for Active Directory (AD). It covers planning, hardware assembly, device connections, and network configuration to ensure a solid foundation for your AD deployment.

---

### Step 1: Network Planning
Careful planning is the foundation of a successful network setup. This step involves defining the scope of your network and establishing an IP addressing scheme.

- **Determine Departments, Users, and Devices**:
  - **Departments and Computers**:
    - IT: 4 computers
    - HR: 4 computers
    - Sales: 5 computers
    - Production: 8 computers
    - **Total Computers**: 21
  - **Printers**: Plan for at least 1 printer per department, totaling 4 printers.
  - **VoIP Phones**: Assume 1 phone per user (21 phones) unless specified otherwise by your organization.

- **Define IP Addressing Scheme**:
  - Use a private IP range like `192.168.1.0/24`, which provides 254 usable IP addresses—sufficient for your 47 devices (21 computers + 21 VoIP phones + 4 printers + 1 server).
  - **Static IP Range**: Reserve `192.168.1.1` to `192.168.1.50` for servers, printers, and VoIP phones.
  - **DHCP Range**: Allocate `192.168.1.51` to `192.168.1.254` for client computers to receive IPs dynamically.

- **Identify Static IP Requirements**:
  - Assign static IPs to key devices:
    - Server: `192.168.1.10`
    - Printers: `192.168.1.20` (IT), `192.168.1.21` (HR), `192.168.1.22` (Sales), `192.168.1.23` (Production)
    - VoIP phones: Optionally `192.168.1.30` to `192.168.1.50`

- **Consider VLANs (Optional)**:
  - VLANs can segment traffic for security and performance:
    - **VLAN 10**: Workstations (data)
    - **VLAN 20**: VoIP phones
    - **VLAN 30**: Printers
  - VLANs require managed switches and a router or Layer 3 switch for inter-VLAN routing.

---

### Step 2: Gather Required Hardware
Collect all necessary hardware to ensure you’re fully equipped for the setup.

- **1 Windows Server Machine**:
  - **Minimum Specs**: 8GB RAM, 2 CPU cores (16GB RAM recommended for better performance).
  - This will host Active Directory Domain Services (AD DS) and serve as the Domain Controller (DC).

- **Network Switches**:
  - **Total Devices**: 21 computers + 21 VoIP phones + 4 printers + 1 server = 47 devices.
  - Use two 24-port switches or one 48-port switch.
  - If using VLANs, ensure the switches are managed and support VLANs.

- **Wi-Fi Router**:
  - Optional for wireless access (e.g., laptops or mobile devices).
  - Must support your IP scheme and integrate with the network.

- **21 Client Computers**:
  - Desktops or laptops for users across the four departments, each with an Ethernet port.

- **Department Printers**:
  - 4 network-capable printers (1 per department) with Ethernet ports.

- **VoIP Phones**:
  - 21 phones (assuming 1 per user), with Power over Ethernet (PoE) support or power adapters.

- **Optional NAS or Backup Device**:
  - A Network Attached Storage (NAS) or backup server for data redundancy.

---

### Step 3: Server Setup
Prepare the server that will host Active Directory.

1. **Install Windows Server**:
   - Install Windows Server 2019 or 2022.
   - Follow the installation wizard, selecting an appropriate edition (e.g., Standard).

2. **Assign a Static IP Address**:
   - Navigate to **Control Panel** > **Network and Sharing Center** > **Change adapter settings**.
   - Right-click the Ethernet adapter > **Properties** > **Internet Protocol Version 4 (TCP/IPv4)** > **Properties**.
   - Set:
     - IP address: `192.168.1.10`
     - Subnet mask: `255.255.255.0`
     - Default gateway: Router’s IP (e.g., `192.168.1.1`)
     - Preferred DNS server: `192.168.1.10` (the server itself, as it will host DNS).
   - Save and apply settings.

3. **Name the Server**:
   - Go to **Control Panel** > **System and Security** > **System** > **Change settings** > **Computer Name** tab.
   - Click **Change**, enter a name (e.g., `DC1`), and restart the server.

4. **Connect to the Switch**:
   - Use an Ethernet cable to connect the server’s network adapter to a switch port.

5. **Ensure Internet Connectivity**:
   - Test by opening a browser or running `ping google.com` to confirm access for updates.

---

### Step 4: Connect All Client Devices
Physically connect all devices to the network.

1. **Use Ethernet Cables**:
   - Connect each of the 21 computers, 4 printers, and 21 VoIP phones to the switches.
   - Label cables or ports (e.g., “IT_PC1”, “Sales_Printer”) for organization.

2. **Ensure IP Address Assignment**:
   - **Static IPs**:
     - Assign manually to the server, printers, and optionally VoIP phones.
     - Example:
       - Server: `192.168.1.10`
       - Printers: `192.168.1.20` to `192.168.1.23`
       - VoIP phones: `192.168.1.30` to `192.168.1.50`
   - **DHCP for Client Computers**:
     - Computers will receive IPs dynamically from the DHCP range (configured later).

---

### Step 5: Configure Network Settings
Set up network parameters for seamless communication.

1. **Set the Server’s IP as Preferred DNS**:
   - On each client device, set the DNS server to `192.168.1.10` (manually for now, or via DHCP later).

2. **Ensure All Devices Are on the Same Subnet**:
   - Confirm all devices use IPs in the `192.168.1.x` range with a subnet mask of `255.255.255.0`.

3. **Test Connectivity**:
   - From a client computer, run:
     - `ping 192.168.1.10` (to the server)
     - `ping 192.168.1.1` (to the router)
   - From the server, ping a client (e.g., `ping 192.168.1.51`).

---

### Step 6: Prepare Printers
Configure network printers for accessibility.

1. **Assign Static IPs to Printers**:
   - Access each printer’s control panel or web interface.
   - Set static IPs (e.g., `192.168.1.20` for IT, etc.).

2. **Connect Printers to the Network**:
   - Use Ethernet cables to connect each printer to the switch.

3. **Test Printing**:
   - From a workstation, add the printer via its IP and print a test page.
   - Test from the server as well.

---

### Step 7: Prepare VoIP Phones
Set up VoIP phones for communication.

1. **Connect VoIP Phones to the Network**:
   - Use a PoE switch or power adapters.

2. **Assign IPs**:
   - Manually set static IPs (e.g., `192.168.1.30` to `192.168.1.50`) or use DHCP reservations.

3. **Configure SIP Settings**:
   - Enter SIP server details on each phone to connect to your VoIP provider.

---

### Step 8: Optional – Configure VLANs for Segmentation
If using VLANs, segment the network for security and performance.

1. **Create VLANs on the Switch**:
   - Access the switch’s management interface.
   - Define VLANs (e.g., VLAN 10 for workstations, VLAN 20 for VoIP, VLAN 30 for printers).

2. **Assign Switch Ports to VLANs**:
   - Map ports to VLANs based on device type.

3. **Enable Inter-VLAN Routing**:
   - Configure your router or Layer 3 switch to route traffic between VLANs.

---

### Step 9: Verify Network Connectivity
Ensure all devices are connected and communicating.

1. **Ping Tests**:
   - From the server, ping each device (e.g., `ping 192.168.1.20`).
   - From a client, ping the server.

2. **Check Device Visibility**:
   - Use `arp -a` or a tool like Advanced IP Scanner to verify all devices are detected.

---

### Step 10: Final Checks Before AD Installation
Perform these checks to ensure readiness.

1. **Confirm All Devices Are Powered On and Connected**:
   - Verify all 47 devices are operational.

2. **Verify Server Configuration**:
   - Check the server’s static IP, internet access, and hostname.

3. **Document the Network**:
   - Record static IPs, switch port mappings, and device locations.

---

## Next Steps
With the physical setup complete, your network is ready for Active Directory Domain Services (AD DS) installation. Proceed to the software configuration phase to set up AD DS, organizational units (OUs), user accounts, and domain-joined computers.

This detailed setup ensures a robust and scalable network infrastructure for your Active Directory environment.
