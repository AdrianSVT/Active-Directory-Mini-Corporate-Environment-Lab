# Active Directory Home Lab

## Overview

For this project, I built a home lab environment in VirtualBox that simulates a real-world corporate network using Active Directory. I provisioned a server on a virtual machine, installed Active Directory, and established my own domain. The virtual machine was configured with two network adapters: one to connect to the external internet and another to connect to the VirtualBox private network that client machines will use. After the virtual machine was set up, I installed Windows Server 2019 and assigned IP addressing for the internal network. The external network automatically receives IP addressing from my home router.

From there, I configured NAT and routing to allow clients on the private network to reach the internet through the domain controller. I then set up DHCP on the domain controller so that any new client machine added to the network can automatically receive an IP address. Before creating the CLIENT1 virtual machine, I ran a PowerShell script that automatically generates one thousand user accounts in Active Directory. Bellow is a visualization of what I will be doing in this lab.

<img width="1299" height="768" alt="Outline" src="https://github.com/user-attachments/assets/fedd228e-450d-4cdb-a39f-658b6a0a55fd" />

---

## Stage 1: Installing Windows Server 2019 & Configuring Active Directory

After installing Windows Server 2019, the first task was to assign IP addressing. I navigated to **Settings** -> **Network & Internet** -> **Status**. On the bottom of the window you will see the option **Change Adapter Options**. After clicking this option I was brought to the **Network Connections** window. From here I right-clicked the internal network adapter named (x_Internal_x), selected **Properties**, and double-clicked **Internet Protocol Version 4 (TCP/IPv4)** to open its properties. Next I selected **Use the following IP addresses** and entered the appropriate values as illustrated in the diagram.

<img width="752" height="538" alt="Assigning ip adress to internal NIC" src="https://github.com/user-attachments/assets/3dcc82e5-081a-4642-8e2b-048619cd9cfd" />


No default gateway was assigned to the internal network adapter because the domain controller will fulfill that role itself — routing traffic between the internal network and the internet. As for DNS, Active Directory requires DNS to function and will automatically install it during setup, so rather than pointing to an external DNS server, the domain controller points to itself. This is done by entering the loopback address `127.0.0.1` as the DNS server, which simply means "use this machine."

The next step was to install Active Directory. I navigated to **Add Roles and Features** from the **Server Manager Dashboard**. A new window called **Add Roles and Features Wizard** should pop up. I accepted the default settings of the wizard, and clicked **Next** until reaching the **Server Roles** section. I checked the box for **Active Directory Domain Services**, which opened a prompt to add the required features.

<img width="433" height="353" alt="Applied Active Directory Domain Services" src="https://github.com/user-attachments/assets/6fae95b1-fa6d-4228-a7bb-95c84a1e5c23" />

 After clicking **Add Feature** and proceeding through the rest of the wizard with default configurations, I clicked **Install** at the End.

<img width="427" height="322" alt="Installing Active Directory" src="https://github.com/user-attachments/assets/8803af4f-839f-4095-bff3-0f2d4396df66" />
 
Once the installation completed, a **Post-Deployment Configuration** was required to promote the server to a domain controller. 

<img width="232" height="199" alt="Promoting server to domain controller notification" src="https://github.com/user-attachments/assets/e076ce26-918d-4ce0-b97c-995815fcf690" />

In the configuration wizard, I selected **Add a new forest** and created a root domain named `mydomain.com`. After proceeding through the rest of the wizard with default configurations, I clicked **Install** at the End.

<img width="425" height="311" alt="Setting forest and addiing root domain name" src="https://github.com/user-attachments/assets/58f63f69-6cdb-462c-b14c-5dd9e4407fbd" />

### Creating a Dedicated Domain Admin Account

With the domain controller set up, I created a dedicated domain admin account. I opened **Active Directory Users and Computers** and created an organizational unit called `_ADMINS` to house the admin account. I then created a new user within that OU and assigned a password.

<img width="713" height="518" alt="Creating Organizational Unit 1" src="https://github.com/user-attachments/assets/3bd785c8-ca84-412d-86df-56d2600c8a55" />

<img width="612" height="438" alt="Creating Organizational Unit 2 _ADMINS" src="https://github.com/user-attachments/assets/2c28579e-244b-41e2-87ad-8da5b0c3ab20" />

I then created a new user within that OU and assigned a password. To create a new user I right clicked on **_ADMINS** -> **New** -> **User**. Once I was brought to the **New Object - User** window I populated the first name, last name, Full name, and logon name. I Then assigned a password for the Account.

<img width="602" height="438" alt="established admin user properties " src="https://github.com/user-attachments/assets/cc83ce1e-9a3d-46dc-b2ae-656323d8d2cc" />

<img width="595" height="419" alt="Creating New Admin User with password" src="https://github.com/user-attachments/assets/95f2a54e-5268-4c70-ab91-61396128cf6a" />

<img width="583" height="416" alt="Creating New Admin User compleation" src="https://github.com/user-attachments/assets/1acfa093-073e-4899-a006-717625d3537b" />




To grant admin privileges, I opened the user's properties, navigated to the **Member Of** tab, and clicked **Add**. I entered `domain admins` as the object name, clicked **Check Names** to resolve it to `Domain Admins`, and applied the change. The user was now a member of the Domain Admins group. I logged out of the domain controller and signed back in using the newly created domain admin account.

<img width="220" height="288" alt="NEW Admin user properties" src="https://github.com/user-attachments/assets/82fdb008-1e69-4042-8c5d-611f79721d84" />




---

## Stage 2: Configuring NAT & Routing

With the domain controller in place, the next stage was to configure NAT and routing. This is a critical step because it enables client machines on the internal private network to access the internet by routing their traffic through the domain controller, just as traffic would flow through a router on a corporate network.

To set this up, I returned to the Server Manager dashboard and launched **Add Roles and Features**. Once I reached the **Server Roles** section, I checked the box for **Remote Access** and continued to the **Role Services** section, where I also checked **Routing**. After proceeding through the remaining steps, I clicked **Install**.

Once the installation was complete, I navigated to **Tools** and selected **Routing and Remote Access** from the drop-down menu. I right-clicked **DC(local)** and chose **Configure and Enable Routing and Remote Access**. Within the setup wizard, I selected **Network Address Translation (NAT)** and on the following page, chose **Use this public interface to connect to the Internet**. After completing the wizard, NAT was fully configured.

---

## Stage 3: Configuring the DHCP Server

With routing in place, the next step was to configure the DHCP server on the domain controller. This allows client machines to automatically receive an IP address when they join the network, enabling them to communicate and access the internet without any manual configuration.

I navigated to **Add Roles and Features** on the Server Manager dashboard, checked the **DHCP** box under **Server Roles**, and proceeded through the wizard to install it. After installation, I went to **Tools** and selected **DHCP** from the drop-down menu.

### Setting Up the DHCP Scope

From the DHCP control panel, I right-clicked **IPv4** and selected **New Scope**. I named the scope `172.16.0.100 - 200` to reflect its IP range, and in the following window, defined the address range as `172.16.0.100 - 200` with a `/24` prefix. Since this is a practice lab, I skipped the exclusion range window. I left the lease duration at the default of 8 days and proceeded to the DHCP options configuration.

Here, I set the default gateway to the domain controller's IP address (`172.16.0.1`), since it will be acting as the gateway for the internal network. The DNS settings were left as suggested by the wizard. On the final page, I chose to activate the scope immediately and clicked **Finish**.

To complete the setup, the DHCP server needed to be authorized before it could begin assigning IP addresses. I right-clicked the DHCP server and selected **Authorize**. The DHCP server was now fully configured and ready to serve client machines.

---

## Stage 4: Creating User Accounts Manually and in Bulk Using PowerShell

### Creating a User Account Manually

To create a user account manually, I followed the same process used when setting up the dedicated domain admin account in Stage 1. I started by creating an organizational unit called `Domain Users` to store the accounts. I then right-clicked the newly created OU, selected **New**, then **User**, and filled out the required information in the **New Object - User** window, including the user's first name, last name, logon name, and password. The new user account was successfully created.

### Bulk User Creation with PowerShell

To simulate a real enterprise environment, I also used a PowerShell script to generate one thousand user accounts in bulk. The script references a `names.txt` file containing one thousand randomized names, which it uses to create a corresponding user account for each entry.

I launched **PowerShell ISE** as an administrator and first enabled script execution on the server by running the following command:

```powershell
Set-ExecutionPolicy Unrestricted
```

I then navigated to the directory containing the script and the `names.txt` file to ensure the script could locate and read from it. With everything in place, I ran the script.

To verify the results, I navigated to **Active Directory Users and Computers** where I could see that the script had created a new organizational unit called `_USERS`, populated with all one thousand newly created accounts.

---

## Connecting a Windows 10 Client to the Domain

To complete the lab, I created a virtual machine called **CLIENT1** running Windows 10 Pro to simulate an endpoint device joining the corporate network. The VM's network adapter was set to connect to the **Internal Network**, placing it on the same private network as the domain controller.

### Verifying the Connection

Once the OS booted, I verified the connection to the domain controller in two ways.

**Method 1 — ipconfig on CLIENT1:**
I ran the following command in the command prompt on CLIENT1:

```cmd
ipconfig /all
```

The output confirmed that the machine had received the correct default gateway (`172.16.0.1`), that DHCP was enabled, and that the DNS suffix matched the domain (`mydomain.com`).

**Method 2 — DHCP Address Leases on the Domain Controller:**
I opened the DHCP console from the **Tools** menu on the domain controller, navigated to **IPv4**, and selected **Address Leases**. CLIENT1 appeared in the list and had been assigned an IP address from the DHCP scope, confirming that the DHCP server was working as expected.

### Testing Internet Connectivity

I tested internet connectivity by running a ping to `www.google.com` from CLIENT1's command prompt:

```cmd
ping www.google.com
```

The ping returned successful replies, confirming that the domain controller is correctly routing internet traffic for machines on the internal network.

### Joining the Domain

The final step was to rename the machine and join it to the domain. I navigated to **Settings → System → About** and clicked **Rename this PC (Advanced)**. In the **System Properties** window, I clicked **Change**, updated the computer name to `CLIENT1`, and entered `mydomain.com` as the domain to join. When prompted, I provided the credentials of a domain account with permission to join the domain.

To confirm successful enrollment, I navigated to **Active Directory Users and Computers** on the domain controller, expanded `mydomain.com`, and clicked **Computers**. CLIENT1 appeared in the list, confirming it had been successfully joined to the domain.

With that, the home lab was complete — a fully functional mini corporate network built from the ground up.
