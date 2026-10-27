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



