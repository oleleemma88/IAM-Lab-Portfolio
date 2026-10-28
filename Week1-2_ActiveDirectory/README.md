Week 1 — Task 1: What Is Active Directory?

Summary

A Windows domain is a group of users and computers managed by one business. Its purpose is to centralise administration in one repository called Active Directory (AD). The server running AD is the Domain Controller (DC).
The problem it solves
•	Small network (5 computers, 5 users): You can configure each machine by hand and fix problems on-site.
•	Growing network (157 computers, 320 users, 4 offices): Manual management doesn’t scale. You need a central system.
Main advantages
1.	Centralised identity management: Users across the whole network are configured from one place with minimum effort.
2.	Managing security policies: Policies are set in AD and applied to users and computers across the network.
Real-world example
At school or university, one username and password works on any campus computer. The machine forwards your login to AD, which checks your credentials. Your credentials don’t need to be stored on each machine. AD also deploys policies that stop you from accessing the control panel or gaining admin rights.
Diagram explained
<img width="975" height="461" alt="image" src="https://github.com/user-attachments/assets/ccb92692-2807-4fac-9b52-5f5b4b391891" />


The image shows the TryHackMe.com domain as one boundary containing:
•	Domain Controller: the central server that runs AD and handles authentication
•	File Server, Database Server: shared resources the domain manages
•	Workstation, Desktop: end-user computers that join the domain
Every device sits inside the same domain, so one central system manages all of them.
Scenario for this task
I’m the new IT admin at THM Inc. My job is to review the existing domain THM.local and make additional configurations using administrative credentials on a pre-configured Domain Controller.

Key takeaway: Active Directory lets one admin manage identities, computers, and policies for an entire organisation from a single place, instead of configuring each machine separately.


Week 1 — Task 2: Active Directory Objects, OUs and Groups
<img width="975" height="384" alt="image" src="https://github.com/user-attachments/assets/9e552282-13aa-44b2-aba0-ad4e71d37f1c" />
<img width="975" height="303" alt="image" src="https://github.com/user-attachments/assets/af8c338c-695d-46ed-828d-f7ea9ae0d145" />
<img width="975" height="385" alt="image" src="https://github.com/user-attachments/assets/bd5860db-d82a-4d4a-bf67-26b705790487" />


Summary

Active Directory Domain Services (AD DS) is a catalogue holding information about every object on the network. The main object types are:
•	Users: Security principals that can be authenticated and granted access to resources. They represent people (employees) or services (accounts that run applications like IIS or MSSQL, with only the privileges they need).
•	Machines: Created automatically when a computer joins the domain. Each has its own account, named after the computer followed by a dollar sign (for example, DC01$). Machine passwords rotate automatically.
•	Security groups: Collections of users and machines (and sometimes other groups) used to grant access to resources. Adding a user to a group gives them the group’s permissions.


Default groups to know

•	Domain Admins: full administrative control over the domain, including Domain Controllers.
•	Server Operators: administer Domain Controllers but cannot change admin group memberships.
•	Backup Operators: can read any file regardless of permissions, for backups.
•	Account Operators: create and modify other accounts.
•	Domain Users, Domain Computers, Domain Controllers: contain all user accounts, all computers, and all DCs respectively.


Organisational Units (OUs) vs Security Groups

Both classify users and computers, but for different purposes:
•	OUs apply policies. A user can belong to only one OU, because one user shouldn’t receive two conflicting sets of policies.
•	Security groups grant permissions to resources such as shared folders or printers. A user can belong to many groups.


What I saw in the lab

•	The domain thm.local contains a THM OU with five child OUs: IT, Management, Marketing, Research and Development, and Sales. OUs usually mirror the business structure so baseline policies can apply to whole departments.
•	Windows also creates default containers: Builtin (default groups), Computers (newly joined machines), Domain Controllers, Users, and Managed Service Accounts.
•	I opened Active Directory Users and Computers from the Start menu on the Domain Controller.
•	The IT OU contains three users: Claire, Mary and Phillip.


Key takeaway: OUs decide which policies apply to a user, and groups decide which resources a user can access. Keeping these two ideas separate is essential for IAM work.


Week 1 — Task 3: Deleting OUs, Fixing Users and Delegating Control

 <img width="886" height="380" alt="image" src="https://github.com/user-attachments/assets/09aa10d1-4fed-47e0-836b-4766e33ac049" />
 <img width="884" height="295" alt="image" src="https://github.com/user-attachments/assets/b7497cfc-6f7f-4ad9-a53b-4d8078241d33" />
 <img width="884" height="372" alt="image" src="https://github.com/user-attachments/assets/98f0576a-c08d-4905-85ac-e5c27242389c" />

Summary

In this task I acted as the new domain administrator at THM Inc. The business had recently changed, so my job was to make the AD match the new organisational chart.

The organisational chart

•	Management: Daniel, General Manager
•	Marketing: Mark, Marketing Specialist
•	Sales: Sophie (Sales Director) and Thomas (Sales Rep.)
•	IT: Phillip (IT Support), Mary (Server Admin) and Claire (Domain Admin)

Step 1: Remove the extra OU

The domain contained one OU that did not appear on the chart. It had been closed after budget cuts, so it needed to be removed.
•	Deleting it directly failed. OUs are protected from accidental deletion by default.
•	I enabled Advanced Features from the View menu in Active Directory Users and Computers.
•	In the OU’s Properties, on the Object tab, I unchecked the protection option.
•	I then deleted the OU and confirmed. Deleting an OU also removes every user, group and child OU inside it.

Step 2: Match the users to the chart

After removing the OU, some departments still had users that did not match the chart. I created and deleted user accounts until each OU matched the chart.

Step 3: Delegate password resets

Delegation gives a user specific permissions over an OU without needing a Domain Admin.
•	Phillip (IT Support) needs to reset passwords for low-privilege staff.
•	I chose the Sales OU and selected Delegate Control.
•	I entered phillip and used Check Names to confirm the account.
•	I selected the option to reset user passwords and finished the wizard.
•	The same approach could be repeated for Marketing and Management.

Step 4: Test the delegation

The final step is to log in as Phillip over RDP and reset Sophie’s password. This checks that the delegated permission works, and that Phillip can act on Sales users without holding Domain Admin rights.

Key takeaway: Delegation lets you give a support team only the permissions they need on specific OUs. This is the principle of least privilege, and it is a core IAM skill.




Week 1 — Task 4: Managing Computers in AD
<img width="975" height="510" alt="image" src="https://github.com/user-attachments/assets/97bfa687-7bad-4345-9fa3-237a4959fd88" />
<img width="975" height="498" alt="image" src="https://github.com/user-attachments/assets/da1240bc-2f95-413c-9b5d-17c212b8127c" />

  
Summary

Every machine that joins a domain, except Domain Controllers, is placed by default in the Computers container. Leaving all devices there is poor practice, because servers and user workstations usually need different policies.

Why separate devices?

The recommended starting point is to group devices by how they are used. There are three main categories:
1.	Workstations: the everyday PCs and laptops that users work and browse on. A privileged account should never be signed in to them.
2.	Servers: machines that provide services to users or other servers.
3.	Domain Controllers: manage the domain and hold the hashed passwords for every user. They are the most sensitive devices on the network. Windows already places them in their own OU.

What I saw in the lab

The Computers container held nine devices:
•	Laptops: LPT-PHILLIP, LPT-SOPHIE, LPT-THOMAS
•	PCs: PC-CLAIRE, PC-DANIEL, PC-MARK, PC-MARY
•	Servers: SRV-DB01, SRV-DB02, SVR-WEB01

What I did

1.	Created two new OUs directly under thm.local: Workstations and Servers.
2.	Moved the laptops and PCs from the Computers container into Workstations.
3.	Moved the servers into Servers.
The result is a clearer structure: Domain Controllers in their Windows-managed OU, workstations in their own OU, and servers in theirs. This lets me apply separate policies to each group later.

Key takeaway: Organising computers by role lets you apply the right security policy to each type of device. Servers and workstations should not share the same rules.



Week 1 — Task 5: Group Policy

<img width="739" height="419" alt="image" src="https://github.com/user-attachments/assets/71581aa5-5c9f-452d-8279-8dd26864bee8" />
<img width="739" height="313" alt="image" src="https://github.com/user-attachments/assets/3f6b425d-27d8-41c2-abe4-92c220486652" />
<img width="740" height="390" alt="image" src="https://github.com/user-attachments/assets/f1cbd4d9-5afc-4877-9d4e-5a2c99f460c7" />

   
Summary

Group Policy Objects (GPOs) are collections of settings linked to OUs or the domain. They let me push different configurations and security baselines to users and computers depending on where they sit in the hierarchy. A GPO can contain Computer Configuration settings, User Configuration settings, or both.

How GPOs work

•	Create GPOs under Group Policy Objects in Group Policy Management, then link them to the OU or domain where they should apply.
•	A GPO applies to the linked container and every sub-OU beneath it. For example, the Default Domain Policy, linked to thm.local, also affects Sales.
•	Security filtering narrows a GPO to specific users or computers. By default it applies to Authenticated Users.
•	Changes are distributed through the SYSVOL share on each Domain Controller. Computers pick up changes periodically, or I can run gpupdate /force to apply them immediately.

What I saw in the lab

The lab had three GPOs:
•	Default Domain Policy and RDP policy, linked to thm.local
•	Default Domain Controllers Policy, linked to the Domain Controllers OU
The Default Domain Policy contains only Computer Configuration settings, including password and account lockout rules.

What I configured

1. Restrict Control Panel Access
•	Created a GPO and edited it under User Configuration.
•	Enabled Prohibit access to Control Panel and PC settings.
•	Linked it to the Marketing, Management and Sales OUs. IT was left out, so IT users keep Control Panel access.

2. Auto Lock Screen
   
•	Created a GPO under Computer Configuration and set the machine inactivity limit to 5 minutes.
•	Linked it to the thm.local root domain. Workstations, Servers and Domain Controllers inherit it. User-only OUs ignore the computer settings.

3. Password length (Default Domain Policy)
   
•	Changed the minimum password length to 10 characters under Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy.

Testing

I signed in over RDP as Mark (Marketing) with THM\Mark. Opening the Control Panel was denied by the administrator. Users in IT can still open it.

Key takeaway: A GPO’s effect depends on where it is linked and who it is filtered to. Linking at the right level lets one setting protect the whole organisation.




Week 1 — Task 6: Authentication (Kerberos and NetNTLM)
<img width="975" height="378" alt="image" src="https://github.com/user-attachments/assets/3c7cfdaf-cbda-4b7e-baa8-8992ba3137f7" />
<img width="975" height="373" alt="image" src="https://github.com/user-attachments/assets/925ed96f-df2b-4980-b959-2e97cceab3c3" />
<img width="975" height="311" alt="image" src="https://github.com/user-attachments/assets/2e3525c4-9e50-496a-b831-4de21724df07" />
<img width="975" height="324" alt="image" src="https://github.com/user-attachments/assets/ce3f59e3-33a9-4363-9ed3-bce278fbf46d" />

    
Summary

In a Windows domain, all credentials are stored on the Domain Controllers. When a user authenticates to a service with domain credentials, the service asks the DC to verify them. Two protocols handle this:
•	Kerberos: the default in any recent version of Windows.
•	NetNTLM: a legacy protocol kept for compatibility. Most networks still have it enabled, but it should be considered obsolete.

Kerberos

Kerberos uses tickets as proof of earlier authentication. The process has three steps:
1.	Request TGT: The user sends their username and an encrypted timestamp to the KDC, a service on the Domain Controller. The KDC returns a Ticket Granting Ticket (TGT) and a Session Key. The TGT is encrypted with the krbtgt account’s hash, so the user cannot read its contents.
2.	Request TGS: To reach a specific service, the user presents the TGT, their username, a timestamp, and the Service Principal Name (SPN), such as MSSQL/SRV. The KDC returns a Ticket Granting Service (TGS) and a Service Session Key. The TGS is encrypted with the service owner’s hash.
3.	Authenticate to the service: The user presents the TGS to the service. The service decrypts it with its own account hash and validates the session key.
The user never has to send their password again after the first step.

NetNTLM

NetNTLM uses a challenge-response exchange:
1.	The client sends an authentication request to the server.
2.	The server sends a random challenge.
3.	The client combines its NTLM hash with the challenge to produce a response, and sends it back.
4.	The server forwards the challenge and response to the Domain Controller.
5.	The DC recalculates the response and compares it. It returns allow or deny.
6.	The server tells the client the result.
   
The password hash never crosses the network. For a local account, the server can verify the response itself using the hash stored in its SAM database.



Key takeaway: Kerberos gives users a ticket once and lets them reach services without re-entering their password. NetNTLM is older and should be phased out where possible.



Week 1 — Task 7: Trees, Forests and Trust Relationships
<img width="881" height="359" alt="image" src="https://github.com/user-attachments/assets/8e18917b-9541-4ef3-9f7c-9e34515ce9ae" />
<img width="881" height="392" alt="image" src="https://github.com/user-attachments/assets/1c07894e-9d65-4434-9109-c46824102c48" />
 

Summary

A single domain works well at first, but as a company grows it may need several domains. Active Directory lets you join domains together, so each part of the business can be managed independently while the whole network stays connected.

Trees

A tree is a set of domains that share the same namespace and form a hierarchy. In the example, the root domain is thm.local, with two child domains:
•	uk.thm.local (managed by DC-UK)
•	us.thm.local (managed by DC-US)

Each domain has its own DC, users and computers. The UK IT team manages UK resources only and cannot manage US users. Policies can also be set independently in each domain.
The Enterprise Admins group has control over every domain in the enterprise. Each domain still has its own Domain Admins, who control only that domain.

Forests

A forest is a collection of trees with different namespaces. For example, if THM acquires MHT Inc., the two companies’ trees can join the same forest, while each keeps its own IT team.

Trust relationships

Trust relationships let users in one domain access resources in another.

•	One-way trust: If Domain AAA trusts Domain BBB, users in BBB can be authorised to access resources in AAA. The trust direction is the opposite of the access direction.
•	Two-way trust: Both domains authorise users from the other. Domains in the same tree or forest get two-way trusts by default.

A trust does not automatically grant access to everything. Administrators still decide which users and resources are authorised.

Key takeaway: Trees and forests let large organisations split management across domains, and trust relationships control which users can reach resources in other domains.


