<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Set up Resources in Azure
- Ensure Connectivity Between the client and Domain Controller
- Install Active Directory (AD)
- Create an Admin account in AD
- Join Client-1 to my Domain
- Set up Remote Desktop for non-administrative users
- Create a non-administrative user and log into my domain

<h2>Prerequisites</h2>

- Create a Virtual Machine running on Windows Server 2022 OS named DC-1.
- Create a Virtual Machine running on Windows 10 OS named Client-1. Ensure both are on the same vnet
- Set NIC private address to be static on DC-1 under Virtual machines/ DC-1 | Network settings/ IP configurations

<h2>Deployment and Configuration Steps</h2>

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/0637f98a-d78d-48ef-9b1f-c41844db524b"/>
</p>
<p>
First, we want to make sure the two virtual machines are able to communicate with each other, so we'll remote desktop control into DC-1, open up Windows Defender Firewall with Advanced Security, open Inbound Rules, sort by Protocol, look for ICMPv4, and then enable "Core Networking Diagnostics - ICMP Echo Request (ICMPv4-in)" for Private and Domain Profiles, which are highlighted above. We can verify this worked by logging into client-1 and pinging DC-1's private IP address.
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/c2aa2df2-9c5a-4979-a5a2-916a05b7f76c"/>
</p>
<p>
Next we want to set up Active Directory (AD), so we'll go to server manager and click on "add roles and features". From here, we will advance through each screen till we get to "Server Roles, where we will select "Active Directory Domain Services". When you click it a pop up will appear and we will click the "Add Features" button before continuing through the setup and installing. When it is finished installing we will close the window.  
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/17a93d5d-0da9-4062-9cf4-ec083985fbca"/>
</p>
<p>
On the grey toolbar near the top left portion of the screen there will be a flag icon with a yellow alert. We will click that and then click "promote this server to a domain controller". This will make a screen pop up that is used to configure the domain. On the first page we will click the option to add a new forest, and name it whatever we want as long as we remember what it is. In this case we named it "mydomain.com". On the next screen it will ask us to create a password. Again, we can make the password whatever we want as long as we remember it. Then we will click through the remaining steps and install. After it finishes installing, we can close our connection.
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/a74a01bb-917f-4763-84a2-418da04f5ec0"/>
</p>
<p>
We will log back into DC-1, but this time log in by signing in with the context of the domain, meaning we will want to add your domain name to our user name to sign in. In our case it will be "mydomain.com\(username)". Once we are logged in, we can press the windows key and search "active directory users and computers" and open this. We will go to our domain name, and right click to add two new organizational units, one named "_EMPLOYEES" and one named "_ADMINS". Then in the _ADMINS folder we will add a new user. We'll call this user Jane Doe, and her username can be whatever we'd like. We must remember to uncheck the reset password upon login box. The username can also be whatever we'd like but in this case to indicate this will be an admin account we will make it "jane_admin".
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/bed37114-5387-44a3-8c33-582f77e3691d"/>
</p>
<p>
Now to actually make Jane an admin we must add her to the admin group. To do that we will right click Jane, click add to groups, and this pop up will appear. If we type "domain" into the box, and then click "Check Names" it will show us a few options. We will click "Domain Admins" and press okay to promote Jane to an admin. We will then log off from our remote desktop session.
</p>
<br />

<p>
<img src=https://github.com/CSanders000/configure-ad/assets/161166823/bc10cbf2-b501-43eb-8746-200e9e196808"/>
</p>
<p>
We are then going connect to DC-1 again and log in using our new admin account with context to the domain. So when logging in the username will look like the above.
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/9a731197-8a35-4943-b66c-03a0e03c8af0"/>
</p>
<p>
We will now minimize our session to change client-1's DNS server from the one it inherited from the virtual network to DC-1's DNS server. So we will go back to Azure, go to virtual machines, client one, then from here we will click network settings, ip configuration, DNS servers, and finally we get to this screen. We will select "custom" under DNS servers and then enter DC-1's private IP address. Then we will save and restart client-1. 
</p>
<br />

<p>
<img src=https://github.com/CSanders000/configure-ad/assets/161166823/2cb02a7a-0823-4a88-9121-2f41cfd3c7a3b"/>
</p>
<p>
Next we will log back into client-1 as our original user, go to Settings/Systems/Rename this PC (advanced). We will click "change" on the middle-right side of the System Properties box, then tap domain and enter the name of our domain. It will then have us log in as a user of the domain, so we will log in using Jane's credentials with context to the domain. After signing in client-1 will restart. 
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/6087c76b-3b71-4910-aa4a-59dedbef59a9"/>
</p>
<p>
To make sure client-1 sucessfully joined the domain, we will open our connection to DC-1, and open active directory users and computers. We can go to our domain, click the drop down arrow, and click on the "Computers" folder. If it worked successfully, you should see client-1 in this folder. 
</p>
<br />

<p>
<img src="https://github.com/CSanders000/configure-ad/assets/161166823/160a6213-3d0b-439a-b7d1-a5b4a702195e"/>
</p>
<p>
From here we will make sure any non-administrative users can connect to the domain. So we will switch back to client-1, go to Settings, System, then click on "Remote Desktop". Scroll to the bottom and click "select users that can remotely access this PC". A pop up will open and we will click add, then type in "domain users" and hit enter. Once we hit okay, it will allow any users connected to the domain to access it remotely. 
</p>
<br />

<p>
<img src=https://github.com/CSanders000/configure-ad/assets/161166823/acb1c925-2106-4299-bf98-a82e98c9df9f"/>
</p>
<p>
For the sake of getting familiar with active directory and logging in with different accounts, we can use this [script](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1) that will create several thousand random accounts for us to play around with. 
</p>
<br />

<p>
<img src=https://github.com/CSanders000/configure-ad/assets/161166823/21201646-b7cc-467a-b68d-90c86b255f1b"/>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />
