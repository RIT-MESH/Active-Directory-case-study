# Active-Directory-case-study

Below is a step-by-step guide for setting up Windows Server Active Directory (AD) to centrally manage computers and users across multiple departments, ensuring that no department can access another’s data, setting up users on department-specific computers, and integrating printers and VoIP devices into the domain environment.

---

## **Windows Server Active Directory Setup for Centralized Management**

### **Company Structure**
- **IT Department**: 4 computers
- **HR Department**: 4 computers
- **Sales Department**: 5 computers
- **Production Department**: 8 computers
- **Total**: 21 computers

### **Objectives**
1. Centrally manage all computers and users.
2. Restrict access so departments cannot access each other's data.
3. Set up and manage users on each department's computers.
4. Integrate printers and VoIP devices into the domain environment.

---

## **Theoretical Overview**
Active Directory (AD) is a Microsoft directory service for Windows domain networks that enables centralized management of users, computers, security, and resources. Key components include:
- **Domain Controller (DC)**: The server managing AD functions.
- **Organizational Units (OUs)**: Containers to organize users and computers.
- **Group Policy Objects (GPOs)**: Used to control settings across users and devices.
- **Security Groups**: Manage access to resources.

This guide walks through setting up AD to meet the company’s objectives, ensuring security and efficiency.

---

## **Step-by-Step AD Setup**

### **1. Install AD DS on Windows Server**
- Open **Server Manager** on your Windows Server.
- Navigate to **Manage** > **Add Roles and Features**.
- Select **Active Directory Domain Services (AD DS)** from the list of roles.
- Follow the wizard to complete the installation, and restart the server if prompted.

---

### **2. Promote Server to Domain Controller**
- After installation, a notification flag appears in Server Manager.
- Click the flag and select **Promote this server to a domain controller**.
- Choose **Add a new forest** and enter a domain name (e.g., `company.local`).
- Set a **Directory Services Restore Mode (DSRM)** password (used for recovery purposes).
- Complete the wizard and restart the server when finished.

---

### **3. Open Active Directory Tools**
- After the server restarts, access AD management tools:
  - **Active Directory Users and Computers (ADUC)**: For managing users, groups, and computers.
  - **Group Policy Management Console (GPMC)**: For creating and applying policies.
- Find these tools in the Start menu or via Administrative Tools.

---

### **4. Create Organizational Units (OUs)**
- In **ADUC**, right-click the domain (`company.local`) and select **New > Organizational Unit**.
- Create four OUs: **IT**, **HR**, **Sales**, and **Production**.
- These OUs will organize users and computers by department.

---

### **5. Create Security Groups**
- In each OU (or a central Groups OU), right-click and select **New > Group**.
- Create security groups: **IT_Group**, **HR_Group**, **Sales_Group**, and **Production_Group**.
- Set the group scope to **Global** and group type to **Security**.

---

### **6. Create User Accounts**
- In the appropriate OU (e.g., HR), right-click and select **New > User**.
- Enter user details (e.g., full name, login name) and set an initial password.
- Add each user to their respective security group (e.g., HR users to **HR_Group**) via the **Member Of** tab in user properties.

---

### **7. Join Computers to the Domain**
On each of the 21 computers:
- Go to **This PC > Properties > Advanced system settings > Computer Name > Change**.
- Select **Domain** and enter `company.local`.
- Provide domain admin credentials when prompted, then restart the computer.
- In **ADUC**, move each computer to its respective OU (e.g., IT computers to the IT OU).

---

### **8. Apply Group Policy Objects (GPOs)**
- Open **GPMC**, right-click an OU (e.g., HR), and select **Create a GPO in this domain, and Link it here**.
- Name the GPO (e.g., **HR_Policies**) and right-click it to **Edit**.
- Configure settings such as:
  - User restrictions (e.g., lock desktop settings).
  - Software installation policies.
  - Desktop background customization.
  - Drive mapping (see Step 10).

---

### **9. Create and Secure Shared Folders**
- On a file server (this could be the domain controller or a separate server), create department-specific folders (e.g., `\\server\HRDocs`, `\\server\SalesDocs`).
- Right-click each folder > **Properties > Sharing > Advanced Sharing** > Enable sharing.
- Go to the **Security** tab and set NTFS permissions:
  - Grant **Full Control** to the respective group (e.g., **HR_Group** for `HRDocs`).
  - Remove access for all other groups (e.g., deny **Sales_Group** access to `HRDocs`).

---

### **10. Map Drives via GPO**
- In the GPO Editor for an OU (e.g., **HR_Policies**), navigate to:
  - **User Configuration > Preferences > Windows Settings > Drive Maps**.
- Right-click > **New > Mapped Drive**.
- Set the drive letter (e.g., H:) and path (e.g., `\\server\HRDocs`).
- Use **Item-Level Targeting** to apply this drive only to members of **HR_Group**.

---

### **11. Restrict Logon to Specific Computers**
- In **ADUC**, open a user’s properties (e.g., an HR user).
- Go to the **Account** tab > **Log On To…**.
- Specify the computers (e.g., HR computer names) where this user can log in, restricting access to only their department’s machines.

---

### **12. Enforce Password Policies**
- In **GPMC**, edit the **Default Domain Policy** (right-click > Edit).
- Navigate to **Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy**.
- Set:
  - Minimum password length (e.g., 8 characters).
  - Password complexity (e.g., require letters, numbers, symbols).
  - Maximum password age (e.g., 90 days).

---

### **13. Add and Manage Network Printers via GPO**
- Connect each department’s printer to the server and share it (e.g., `\\server\HRPrinter`).
- In **GPMC**, edit a GPO (e.g., **HR_Policies**).
- Go to **User Configuration > Preferences > Control Panel Settings > Printers**.
- Right-click > **New > Shared Printer**, and enter the printer path (e.g., `\\server\HRPrinter`).
- Use **Item-Level Targeting** to assign the printer to the respective group (e.g., **HR_Group**).

---

### **14. Integrate VoIP Devices**
- Assign static IP addresses to VoIP phones or reserve leases in your DHCP server.
- If VoIP management requires DNS, add records (e.g., A or CNAME) in the domain’s DNS server (accessible via **DNS Manager**).
- Optionally, create a separate OU (e.g., **VoIP_Devices**) in **ADUC** and move VoIP devices there if they support domain joining.
- Use GPOs to configure firewall rules or VLAN tagging for VoIP traffic:
  - Edit a GPO > **Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall**.

---

## **Additional Best Practices**
- **Backup AD**: Regularly back up the domain controller using Windows Server Backup, including the system state.
- **Monitor and Clean Up**: Periodically review ADUC for unused accounts or computers and disable/delete them.
- **Delegate Control**: In **ADUC**, right-click an OU > **Delegate Control** to allow department admins to manage their own users.

---

## **Conclusion**
This setup ensures:
1. **Centralized Management**: All 21 computers and users are managed via AD.
2. **Access Restriction**: NTFS permissions and GPOs prevent inter-department data access.
3. **User Management**: Users are restricted to their department’s computers.
4. **Device Integration**: Printers and VoIP devices are seamlessly integrated into the domain.

By following these steps, your organization will have a secure, scalable, and efficient AD environment tailored to its departmental structure.


Below are 25 additional scenarios to enhance your existing Windows Server Active Directory (AD) setup for a company with multiple departments. These scenarios build on a typical AD environment and cover diverse aspects such as user management, security, automation, compliance, and integration with modern technologies. Each scenario includes an objective and actionable steps to implement it, ensuring a comprehensive and robust AD infrastructure.

---

# **25 Additional AD Scenarios**



## **Advanced Active Directory Scenarios: Step-by-Step Guide**

This guide consolidates 25 advanced Active Directory scenarios into a single, structured resource. These scenarios cover security enhancements, infrastructure management, integration and automation, and user and device management, providing actionable steps to enhance your AD environment.

---

### **Security Enhancements**

#### **1. Fine-Grained Password Policies**
- **Objective**: Apply distinct password policies to specific users or groups for enhanced security.
- **Steps**:
  1. Open the **Active Directory Administrative Center (ADAC)** on a domain controller.
  2. Navigate to **System** > **Password Settings Container**.
  3. Click **New** > **Password Settings**.
  4. Define settings (e.g., minimum length, complexity).
  5. Under **Directly Applies To**, add target users or groups (e.g., `IT_Admins`).
  6. Click **OK** to apply.

#### **2. Certificate-Based Authentication**
- **Objective**: Use digital certificates for secure user and device authentication.
- **Steps**:
  1. Install **Active Directory Certificate Services (AD CS)** via **Server Manager**.
  2. Configure a Certification Authority (CA) during setup.
  3. Issue certificates to users/devices via **certmgr.msc** or GPOs.
  4. Configure services (e.g., VPN) to accept certificates for authentication.

#### **3. Auditing and Monitoring (Including User Activity Reporting)**
- **Objective**: Track AD changes and user activity for security and compliance.
- **Steps**:
  1. Open **Group Policy Management Console (GPMC)** and edit the **Default Domain Controllers Policy**.
  2. Go to **Computer Configuration** > **Policies** > **Windows Settings** > **Security Settings** > **Local Policies** > **Audit Policy**.
  3. Enable **Audit account management** and **Audit logon events**.
  4. Review logs in **Event Viewer** under **Windows Logs** > **Security**.
  5. For reports, use PowerShell (e.g., `Get-ADUser -Filter * -Properties LastLogonDate`) and export to CSV.

#### **4. Network Access Control (NAC) with AD**
- **Objective**: Restrict network access to authorized, AD-joined devices.
- **Steps**:
  1. Deploy **Network Policy Server (NPS)** as a RADIUS server.
  2. Configure NPS policies to authenticate devices against AD using certificates or credentials.
  3. Integrate with network switches or wireless access points to enforce rules.

#### **5. Secure Service Account Management**
- **Objective**: Protect and manage accounts used by applications or services.
- **Steps**:
  1. Create managed service accounts with `New-ADServiceAccount -Name "AppService"`.
  2. Assign to services and restrict permissions.
  3. Enable auditing and use gMSAs (see scenario 2) for automatic password rotation.

#### **6. AD for Compliance Audits (e.g., SOX, HIPAA)**
- **Objective**: Prepare AD for regulatory audits with logging and access controls.
- **Steps**:
  1. Enable auditing for **Account Management** and **Logon Events** in the **Default Domain Controller Policy**.
  2. Generate reports with PowerShell (e.g., `Get-ADUser` for inactive accounts).
  3. Archive logs and review permissions regularly.

#### **7. AD for Secure Remote Desktop Access**
- **Objective**: Enable secure RDP access for authorized users.
- **Steps**:
  1. Create a security group (e.g., `RDP_Users`) in **ADUC** and add users.
  2. Use GPOs to restrict RDP access: **Computer Configuration** > **Policies** > **Administrative Templates** > **Windows Components** > **Remote Desktop Services**.
  3. Enable Network Level Authentication (NLA).

---

### **Infrastructure Management**

#### **8. Domain and Forest Trusts**
- **Objective**: Enable resource sharing across AD domains or forests.
- **Steps**:
  1. Open **Active Directory Domains and Trusts**.
  2. Right-click your domain, select **Properties**, and go to the **Trusts** tab.
  3. Click **New Trust** and create a two-way or one-way trust.
  4. Validate by testing cross-domain authentication.
  5. (Optional) Configure selective authentication for restricted access.

#### **9. AD Sites and Services (Including High Availability and Business Continuity)**
- **Objective**: Optimize AD across locations with high availability.
- **Steps**:
  1. Open **Active Directory Sites and Services**.
  2. Create a site (e.g., `Branch_Office`) under **Sites**.
  3. Assign subnets (e.g., `192.168.1.0/24`) in **Subnets**.
  4. Configure site links for replication (e.g., every 60 minutes).
  5. For high availability, promote additional domain controllers (DCs) via **Server Manager** and monitor with `repadmin /replsummary`.
  6. For continuity, deploy redundant DCs and use **DFS Replication** for backups.

#### **10. AD Recycle Bin**
- **Objective**: Recover deleted AD objects without complex restores.
- **Steps**:
  1. Run in PowerShell:  
     `Enable-ADOptionalFeature -Identity "CN=Recycle Bin Feature,CN=Optional Features,CN=Directory Service,CN=Windows NT,CN=Services,CN=Configuration,DC=company,DC=local" -Scope ForestOrConfigurationSet -Target "company.local"`.
  2. Recover objects in **ADAC** under **Deleted Objects** by selecting **Restore**.

#### **11. AD for Edge Computing Environments**
- **Objective**: Extend AD to remote sites with limited connectivity.
- **Steps**:
  1. Deploy **Read-Only Domain Controllers (RODCs)** at edge locations.
  2. Configure replication in **AD Sites and Services**.
  3. Use cached credentials for offline access.

---

### **Integration and Automation**

#### **12. Integration with Other Services (Including Exchange and Third-Party Apps)**
- **Objective**: Sync AD with services like Azure AD, Exchange, or third-party apps.
- **Steps**:
  1. For Azure AD: Install **Azure AD Connect**, run with **Express Settings**, and verify in the Azure portal.
  2. For Exchange: Install Exchange Server, join it to AD, and manage mailboxes via **Exchange Admin Center**.
  3. For third-party apps: Configure LDAP/Kerberos, create service accounts, and test connectivity.

#### **13. AD Federation Services (ADFS)**
- **Objective**: Enable single sign-on (SSO) across applications.
- **Steps**:
  1. Install **ADFS** via **Server Manager**.
  2. Configure with your domain (e.g., `adfs.company.com`).
  3. Set up a trust with Azure AD or another provider.
  4. Test SSO with a configured app.

#### **14. Identity Lifecycle Management**
- **Objective**: Automate user account management based on employee status.
- **Steps**:
  1. Deploy **Microsoft Identity Manager (MIM)**.
  2. Sync with HR systems to trigger account actions (e.g., disable terminated users).
  3. Log events for auditing.

#### **15. AD Automation with PowerShell Scripts**
- **Objective**: Automate repetitive AD tasks (e.g., user creation).
- **Steps**:
  1. Write scripts (e.g., `New-ADUser`, `Add-ADGroupMember`).
  2. Schedule via **Task Scheduler** (e.g., daily CSV imports).
  3. Log actions to a file.

---

### **User and Device Management**

#### **16. Group Policy Enhancements**
- **Objective**: Centralize configuration management with GPOs.
- **Steps**:
  1. Open **GPMC**, right-click an OU (e.g., `Sales`), and select **Create a GPO**.
  2. Name it (e.g., `Sales_Policies`) and edit.
  3. Configure settings (e.g., **Security Settings** > **User Rights Assignment**).

#### **17. Device Management (Including Printer Mapping)**
- **Objective**: Manage network devices like printers via AD.
- **Steps**:
  1. Share a printer on the print server.
  2. Create a group (e.g., `Printer_Access`) in **ADUC** and add users.
  3. Set permissions in **Printer Properties** > **Security**.
  4. Map via GPO: **User Configuration** > **Preferences** > **Printers**.

#### **18. Dynamic Access Control (DAC)**
- **Objective**: Automate access based on attributes.
- **Steps**:
  1. In **ADAC**, create a **Claim Type** (e.g., `Department`).
  2. Define a **Central Access Rule** (e.g., `Department = HR`).
  3. Apply via GPO to file servers.

#### **19. Nested Groups for Simplified Permissions**
- **Objective**: Organize permissions hierarchically.
- **Steps**:
  1. Create parent (e.g., `All_Employees`) and child groups (e.g., `Sales_Team`).
  2. Nest child groups in the parent via **ADUC**.
  3. Assign permissions to the parent group.

#### **20. Password Self-Service Reset (SSPR)**
- **Objective**: Enable secure self-service password resets.
- **Steps**:
  1. Deploy **Azure AD SSPR** or a third-party tool.
  2. Require registration (e.g., phone, questions).
  3. Enforce MFA for resets.

#### **21. AD for Virtual Desktop Infrastructure (VDI)**
- **Objective**: Manage authentication for virtual desktops.
- **Steps**:
  1. Integrate AD with VDI platforms (e.g., VMware Horizon).
  2. Use **Roaming Profiles** or **User Profile Disks** via GPOs.
  3. Ensure DC accessibility.

#### **22. AD for IoT Device Management**
- **Objective**: Securely manage IoT device identities.
- **Steps**:
  1. Create an OU (e.g., `IoT_Devices`).
  2. Use certificates or service accounts for authentication.
  3. Apply GPOs for security settings.

#### **23. AD Reporting for User Activity**
- **Objective**: Generate user activity reports.
- **Steps**:
  1. Enable auditing and collect logs in **Event Viewer**.
  2. Use PowerShell (e.g., `Get-ADUser -Filter * -Properties LastLogonDate`).
  3. Export to CSV or a SIEM tool.

#### **24. AD for Multi-Language Support**
- **Objective**: Support localized AD experiences.
- **Steps**:
  1. Install language packs on DCs and clients.
  2. Use GPOs to set regional settings.
  3. Provide multilingual documentation.

#### **25. AD for Secure File Share Access**
- **Objective**: Control file share access by role.
- **Steps**:
  1. Create security groups (e.g., `Finance_Files`) and add users.
  2. Set NTFS permissions on shares.
  3. Map drives via GPOs (e.g., `H:`).

---

