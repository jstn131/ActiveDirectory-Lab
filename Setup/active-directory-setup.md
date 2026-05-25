# Active Directory Setup Guide

## What is Active Directory?

Active Directory is a centralized database that stores information and resources about users, workstations, and networks. It handles user identities, devices, and permissions. It primarily manages two essential functions: Authentication (verifying who is able to access our resources) and Authorization (determining what exactly that user can access).

The server responsible for running Active Directory is called the Domain Controller (DC). The DC is the central authority of the domain — it authenticates users when they log in and enforces access policies across the entire network. Without the Domain Controller, Active Directory cannot function.

Think of Active Directory as a country and the Domain Controller as its leader. A country cannot function without a leader and a set of rules. The Domain Controller comes in and enforces those rules on its citizens to ensure the country runs normally and no one tries to break them.

---

## Prerequisites

Before getting started, note that I have already downloaded the required ISO files for both the Domain Controller and the client machine, as well as installed VirtualBox. This guide does not cover those initial download and installation steps — it documents the setup of my personal Active Directory home lab environment.

---

## Step 1 — Creating the Domain Controller VM

I created a VM in VirtualBox that will serve as my Domain Controller. Setup is straightforward — VirtualBox → New → configure your settings. Here are the specs for DC01:

| Setting | Value |
|---|---|
| Name | DC01 |
| OS | Windows Server 2022 (64-bit) |
| Memory | 4096 MB |
| Processors | 2 |
| Storage | 50GB |
| Unattended Installation | Unchecked |
| Folder Location | C:\Homelab\Domain Controller |

Unattended Installation was left unchecked so I could manually configure everything myself for full control and a better understanding of the setup process.

The VM was also configured with two network adapters — NAT for internet access and Internal Network (HomeLab-Network) for communication between virtual machines.

---

## Step 2 — Installing Windows Server 2022

Once the VM boots up you will see the Windows Server installation process. Click through the initial screens until you reach the operating system selection page.

<img width="1027" height="755" alt="Screenshot 2026-05-25 130514" src="https://github.com/user-attachments/assets/cefb9498-19e0-48e1-befd-4920d047013e" />

Select **Windows Server 2022 Standard Evaluation (Desktop Experience)** — that is the second option in the list. This version includes the full graphical user interface which makes managing Active Directory much easier. The first option is the Core version which is command line only and not ideal for this lab.

You will then see a screen asking for a Custom or Upgrade install. Since this is a brand new VM with no existing operating system, select **Custom** for a clean fresh installation.

<img width="1018" height="760" alt="Screenshot 2026-05-25 130922" src="https://github.com/user-attachments/assets/bd55e7a7-1278-44f5-b608-90dd5f8ac839" />

Once installation is complete, you will see a Customize Settings window prompting you to set a password for the built-in Administrator account. The username is automatically set to Administrator by Windows Server and cannot be changed at this step. Set a strong password and confirm it.

<img width="1027" height="762" alt="Screenshot 2026-05-25 131649" src="https://github.com/user-attachments/assets/553a2571-15e4-4fdb-a115-1cd227bead29" />

Once the password is set, Windows Server 2022 will finish loading and bring you to the desktop. Server Manager will automatically open on startup — this is the central hub for managing your server and will be used throughout this lab.

---

## Step 3 — Renaming the Server

By default Windows Server assigns a random computer name. To rename it go to:

**Server Manager → Local Server → click the computer name → Change → enter DC01 → OK → Restart**

Renaming the server before promoting it to a Domain Controller is important. The server name becomes part of the domain identity and is much harder to change after AD DS is installed.

---

## Step 4 — Configuring the Static IP

Before installing Active Directory, a static IP must be assigned to the Internal Network adapter. The DC needs a fixed IP address because client machines will point to it for DNS and authentication. If the IP changes, clients lose the ability to communicate with the domain.

To open Network Connections, run the following command in Command Prompt as Administrator:

```
ncpa.cpl
```

<img width="1023" height="768" alt="Screenshot 2026-05-25 132835" src="https://github.com/user-attachments/assets/5f570477-9ca8-4c1d-9c59-34957da458d9" />

This revealed two adapters — Ethernet and Ethernet 2. I first needed to identify which one was NAT and which one was the Internal Network. To do this I right-clicked **Ethernet → Status → Details**. The IPv4 address showed 10.0.2.15 which is the default VirtualBox NAT IP range, confirming this was the NAT adapter. This adapter was left on DHCP — it handles internet access through the host machine and does not need a static IP.

Now that Ethernet 2 was confirmed as the Internal Network adapter, I configured it with a static IP:

**Ethernet 2 → Properties → Internet Protocol Version 4 (TCP/IPv4) → Properties**

| Setting | Value |
|---|---|
| IP Address | 192.168.10.1 |
| Subnet Mask | 255.255.255.0 |
| Default Gateway | Blank |
| Preferred DNS Server | 127.0.0.1 |

The Preferred DNS Server is set to 127.0.0.1 — the loopback address — meaning the server points to itself for DNS. Once Active Directory is installed, the DC will also run the DNS service and handle all name resolution for the domain.

<img width="1007" height="769" alt="Screenshot 2026-05-25 133034" src="https://github.com/user-attachments/assets/99dbb1d8-ce7e-4fe3-b70a-88bfe2943be6" />

---

## Step 5 — Installing the AD DS Role

Go to **Server Manager → Manage → Add Roles and Features** and click Next through the first screen.

<img width="1024" height="767" alt="Screenshot 2026-05-25 134529" src="https://github.com/user-attachments/assets/54e1a3c7-e5ef-468e-8506-39c24576efea" />

Select **Role-Based or Feature-Based Installation** and hit Next. On the destination server screen, select **DC01** and hit Next.

<img width="1020" height="765" alt="Screenshot 2026-05-25 134714" src="https://github.com/user-attachments/assets/beb31f7a-0a60-4b6c-95d1-326d77bb9c9d" />

On the Server Roles screen, check **Active Directory Domain Services** — it is the second option in the list. A popup will appear asking to add required features — click **Add Features** and then hit Next.

<img width="1021" height="760" alt="Screenshot 2026-05-25 134823" src="https://github.com/user-attachments/assets/0c0d0867-b5a3-4461-acb5-b2a5608dcb79" />

On the Features page click Next — all necessary features were automatically included when we selected AD DS. Click Next again on the AD DS information screen as it is just an overview.

You will then see the Confirmation screen showing everything that will be installed including Active Directory Domain Services, Group Policy Management, and the Remote Server Administration Tools. If everything looks correct, click **Install** and let it run.

<img width="1019" height="760" alt="Screenshot 2026-05-25 135533" src="https://github.com/user-attachments/assets/13313bad-4d00-4576-bb57-56727ec85a43" />

---

## Step 6 — Promoting the Server to a Domain Controller

Once installation is complete, a blue link will appear saying **"Promote this server to a domain controller."** This is the most critical step — installing AD DS only installed the software. Promoting the server is what actually creates the domain and makes DC01 a real Domain Controller.

<img width="1025" height="760" alt="Screenshot 2026-05-25 135921" src="https://github.com/user-attachments/assets/e0e4759b-4d77-4242-80cd-71dec354cee5" />

Select **Add a new forest** since we are building a brand new domain from scratch. Set the root domain name to **homelab.local** and click Next.

On the Domain Controller Options screen everything stays the same — Forest and Domain functional levels remain at Windows Server 2016, DNS Server and Global Catalog stay checked. The only thing to add here is the **DSRM password**. DSRM stands for Directory Services Restore Mode — it is a special recovery password used if Active Directory ever fails and needs to be restored. This is separate from your Administrator password so store it somewhere safe.

<img width="1016" height="767" alt="Screenshot 2026-05-25 140035" src="https://github.com/user-attachments/assets/b29bbd0f-64c0-48d8-95a8-6c1cec7bfc64" />

On the DNS Options screen a warning will appear saying a DNS delegation cannot be created — this is completely normal. Our domain is private and internal so there is no parent zone to delegate to. Leave the checkbox unchecked and click Next.

On the Additional Options screen the NetBIOS domain name will automatically populate as **HOMELAB** based on our homelab.local domain name. Leave it as is and click Next.

The Paths screen shows where AD DS will store its critical files:
- **NTDS.dit** — the Active Directory database. Stores all user accounts, computer accounts, passwords, and group memberships. This is the file the Domain Controller references every time a user attempts to log in.
- **SYSVOL** — stores Group Policy files and logon scripts.

Leave all paths as default and click Next.

On the Review Options screen confirm everything looks correct and click Next. The prerequisites check will run automatically — if all checks pass, click **Install**.

Once installation completes the server will automatically restart. When it comes back up, log in as **HOMELAB\Administrator** instead of just Administrator. This confirms the domain promotion was successful and DC01 is now a fully functioning Domain Controller.

<img width="1020" height="759" alt="Screenshot 2026-05-25 145016" src="https://github.com/user-attachments/assets/78922002-c201-4751-9d5a-2ca8380e818d" />

---

## Step 7 — Verifying the Domain Controller

To confirm everything is working correctly, run the following command in CMD as Administrator:

```
dcdiag /test:services
```

If DC01 passes both the Connectivity and Services tests, Active Directory Domain Services is running correctly and homelab.local is active.
