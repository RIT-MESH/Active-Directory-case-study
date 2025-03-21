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


Below are 20 additional scenarios to enhance your existing Windows Server Active Directory (AD) setup for a company with multiple departments. These scenarios build on a typical AD environment and cover diverse aspects such as user management, security, automation, compliance, and integration with modern technologies. Each scenario includes an objective and actionable steps to implement it, ensuring a comprehensive and robust AD infrastructure.

---

# **20 Additional AD Scenarios**

### **1. Set Up Nested Groups for Simplified Permissions**
- **Objective**: Organize permissions hierarchically to reduce administrative overhead.
- **Steps**:
  - Create parent groups (e.g., `All_Employees`) and child groups (e.g., `Sales_Team`, `HR_Team`).
  - Add child groups as members of the parent group in **Active Directory Users and Computers (ADUC)**.
  - Assign resource permissions (e.g., file shares) to the parent group instead of individual child groups.

---

### **2. Configure Network Access Control (NAC) with AD**
- **Objective**: Restrict network access to authorized, AD-joined devices only.
- **Steps**:
  - Deploy **Network Policy Server (NPS)** as a RADIUS server.
  - Configure NPS policies to authenticate devices against AD using certificates or credentials.
  - Integrate with network switches or wireless access points to enforce access rules.

---

### **3. Set Up AD for Third-Party Application Authentication**
- **Objective**: Enable AD-based login for external applications (e.g., CRM tools).
- **Steps**:
  - Configure applications to use LDAP or Kerberos authentication with AD.
  - Create service accounts in AD for application integration.
  - Assign appropriate permissions to service accounts and test connectivity.

---

### **4. Implement Password Self-Service Reset (SSPR)**
- **Objective**: Allow users to reset their own passwords securely, reducing IT workload.
- **Steps**:
  - Deploy **Azure AD SSPR** or a third-party tool like **ManageEngine ADSelfService Plus**.
  - Require users to register for SSPR (e.g., phone number, security questions).
  - Configure policies to enforce MFA during password resets.

---

### **5. Set Up AD for Virtual Desktop Infrastructure (VDI)**
- **Objective**: Manage user profiles and authentication for virtual desktops.
- **Steps**:
  - Integrate AD with VDI platforms (e.g., VMware Horizon, Citrix).
  - Use **Roaming Profiles** or **User Profile Disks** via GPOs to store user settings.
  - Ensure domain controllers are accessible from the VDI environment.

---

### **6. Configure AD Trusts Between Domains**
- **Objective**: Enable resource sharing between separate AD domains (e.g., after a merger).
- **Steps**:
  - In **AD Domains and Trusts**, create a two-way or one-way trust between domains.
  - Validate trust by testing cross-domain authentication and resource access.
  - Configure selective authentication if restricting access is required.

---

### **7. Set Up AD for IoT Device Management**
- **Objective**: Securely manage identities for IoT devices within the AD environment.
- **Steps**:
  - Create an OU (e.g., `IoT_Devices`) for IoT computer accounts.
  - Use certificates or service accounts for device authentication.
  - Apply GPOs to enforce security settings (e.g., disable unused services).

---

### **8. Implement Group Policy Preferences for Printer Mapping**
- **Objective**: Automatically map printers to users based on their location or department.
- **Steps**:
  - In GPMC, create a GPO and go to **User Configuration > Preferences > Control Panel Settings > Printers**.
  - Add shared printers (e.g., `\\printserver\printer1`) and target them to specific groups or OUs.
  - Apply the GPO to the relevant OUs.

---

### **9. Set Up AD for Compliance Audits (e.g., SOX, HIPAA)**
- **Objective**: Prepare AD for regulatory audits with detailed logging and access controls.
- **Steps**:
  - Enable auditing for **Account Management** and **Logon Events** in the **Default Domain Controller Policy**.
  - Use PowerShell to generate compliance reports (e.g., `Get-ADUser` for inactive accounts).
  - Archive logs and review access permissions regularly.

---

### **10. Configure AD for High Availability with Additional DCs**
- **Objective**: Ensure AD remains operational during server failures.
- **Steps**:
  - Promote additional servers as domain controllers using **Server Manager**.
  - Distribute DCs across physical locations and configure DNS accordingly.
  - Monitor replication health with `repadmin /replsummary`.

---

### **11. Set Up AD for Secure Remote Desktop Access**
- **Objective**: Enable secure RDP access for IT staff or remote workers.
- **Steps**:
  - Create a security group (e.g., `RDP_Users`) and add authorized users.
  - Use GPOs to restrict RDP access to `RDP_Users` under **Computer Configuration > Policies > Administrative Templates > Windows Components > Remote Desktop Services**.
  - Enable Network Level Authentication (NLA) for added security.

---

### **12. Implement Identity Lifecycle Management**
- **Objective**: Automate user account creation, updates, and deletions based on employee status.
- **Steps**:
  - Deploy **Microsoft Identity Manager (MIM)** or a similar tool.
  - Sync with HR systems to trigger account actions (e.g., disable accounts for terminated employees).
  - Log all lifecycle events for auditing.

---

### **13. Set Up AD for Edge Computing Environments**
- **Objective**: Extend AD authentication to remote or edge locations with limited connectivity.
- **Steps**:
  - Deploy **Read-Only Domain Controllers (RODCs)** at edge sites.
  - Configure site-specific replication in **AD Sites and Services**.
  - Use cached credentials for offline authentication.

---

### **14. Configure AD for Business Continuity During Outages**
- **Objective**: Maintain AD functionality during network or power disruptions.
- **Steps**:
  - Set up redundant domain controllers in separate physical locations.
  - Use **DFS Replication** for critical AD data and GPO backups.
  - Test failover scenarios with simulated outages.

---

### **15. Set Up AD Reporting for User Activity**
- **Objective**: Generate reports on user logins, group changes, and account status.
- **Steps**:
  - Enable auditing in AD and collect logs in **Event Viewer**.
  - Use PowerShell scripts (e.g., `Get-ADUser -Filter * -Properties LastLogonDate`) to create reports.
  - Export reports to CSV or integrate with a SIEM tool.

---

### **16. Implement Secure Service Account Management**
- **Objective**: Protect and monitor accounts used by applications or services.
- **Steps**:
  - Create managed service accounts (MSAs) with `New-ADServiceAccount`.
  - Assign MSAs to services and restrict their permissions.
  - Monitor usage with auditing and rotate passwords automatically.

---

### **17. Set Up AD for Multi-Language Support**
- **Objective**: Support users in different regions with localized AD experiences.
- **Steps**:
  - Install language packs on domain controllers and client devices.
  - Use GPOs to set regional settings (e.g., time zones, languages) based on user location.
  - Provide multilingual documentation for AD-related processes.

---

### **18. Configure AD for Integration with Exchange Server**
- **Objective**: Enable AD to manage email accounts and mailboxes.
- **Steps**:
  - Install Exchange Server and join it to the AD domain.
  - Use **Exchange Admin Center** or PowerShell to create mailboxes linked to AD users.
  - Sync AD attributes (e.g., email addresses) with Exchange.

---

### **19. Set Up AD for Secure File Share Access**
- **Objective**: Control access to file shares based on department or role.
- **Steps**:
  - Create security groups (e.g., `Finance_Files`) and assign users.
  - Set NTFS permissions on file shares to allow access only to relevant groups.
  - Use GPOs to map drives (e.g., `H:`) to shares for specific OUs.

---

### **20. Implement AD Automation with PowerShell Scripts**
- **Objective**: Automate repetitive AD tasks like user creation or group updates.
- **Steps**:
  - Write scripts (e.g., `New-ADUser` for user creation, `Add-ADGroupMember` for group updates).
  - Schedule scripts using **Task Scheduler** to run daily or on triggers (e.g., CSV import).
  - Log script actions to a file for auditing.

---

## **Conclusion**
These 20 additional scenarios enhance your Active Directory setup by addressing advanced user management, security, automation, and integration needs. Implementing these will create a more secure, efficient, and scalable AD environment tailored to your company’s diverse requirements. Each scenario can be adapted based on your specific infrastructure, whether on-premises, hybrid, or cloud-integrated, ensuring flexibility and resilience across multiple departments.
