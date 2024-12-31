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

- DC and Client Setup
- Installation of Active Directory
- Internal Configuration
- Account Management

<h2>Deployment and Configuration Steps</h2>

# DC and Client Setup

To set up this Active Directory Environment, we'll need to create two virtual machines: one serving as a domain controller (DC), and one as a client. Open up Azure and create a resource group. Within the resource group, create a virtual netowrk and a subnet. 

After that, again within that same resource group, create the virtual machine that will serve as the DC. Remember that in order for it to function as DC, the "image" needs to be specifically Micorsoft Server OS. Either one of the two in the image below will work.

![image](https://github.com/user-attachments/assets/837e62ab-c68d-4af1-af74-a541ae6da5b2)

As a side note, as much as you can, try to deploy all of these resources to the same regional area.

After taking note of your DC credentials, you need to set the DC's NIC private IP address to be static. To do this we'll need to open the VM within Azure then go to its Network settings.

![image](https://github.com/user-attachments/assets/43e6f9a4-ec14-402f-81a3-6b6c5c13debc)

Then you need to click on your network interface settings.

![image](https://github.com/user-attachments/assets/b5305161-8250-43c1-bca4-b29a4a8f27d7)

After that within the "IP Configurations" pane, you should see an IP address named "ipconfig1". Click on it to manage its settings. Now you should see the option to change the private address from dynamic to static. Go ahead and do such now then hit "Save" at the bottom to finish up.

![image](https://github.com/user-attachments/assets/204ffdf6-489f-471c-991d-f4ce02ce6ed1)


Now that we've established the DC's private IP address as static, we're going to modify a certain setting that will allow us to test for connectivity in a little bit. Let's remote into the DC itself and open up the Control Panel. 

Another side note: open openning/connecting a virtual machine for the first time, one of the initial configuration questions that will be asked is whether or not you want to "allow you PC to be discoverable by other PCs and devices on this network". Select "Yes" for this. Resuming:

After that, click on "System and Security" -> "Windows Defender Firewall" -> "Advanced Settings". From within here, there should be a link titled "Windows Defender Firewall Properties". 

![image](https://github.com/user-attachments/assets/4b30beff-928a-4806-8d30-f0e2d8d6b36c)

Click on it and a small window should pop it containing settings of the Windows firewall. In order to allow for connectivity testing, we're going to disable the firewall. This is rarely something that you would do, but for certain scenarios like ours, it'll be acceptable. 

In order to disable the firewall, go through each of the tabs within the window that contain a "Firewall state" section (Domain Profile, Private Profile, and Public Profile), and turn the state off for each of them. 

![image](https://github.com/user-attachments/assets/111227f5-8f02-44b9-b8c5-30367a53378c)

Click "Apply" then "OK" to confirm these edits. You should then see your Windows Defender Firewall window looking like this:

![image](https://github.com/user-attachments/assets/865814c3-c0c9-4ad0-b7ac-71dad6fbf3ef)

You can now close out of that window. We'll move on to setting up our client machine now.

Back in Azure, create another virtual machine, this time with a Windows 10 Pro "image"/OS. Also, make sure to configure it to be in the same geographical location as well as the same virtual network as the DC.

The next step will be to configure the client's DNS settings to the DC's private IP address. Of course for this you will need to locate the private IP address of the DC. Go again to the networks settings within the DC virtual machine settings. Again click on the network interface settings. This time however, we'll click on "DNS Servers": 

![image](https://github.com/user-attachments/assets/9e152613-6887-4f72-8112-b04b6cbacd62)

Change the setting from "Inherit from virtual network" to "Custom" then enter in the private IP address of your DC:

![image](https://github.com/user-attachments/assets/5067db87-8b02-4271-a8d5-8a45190cb6fa)

Don't forget to hit "Save" at the top when you're finished with that. Now we're ready to move to the last part of the Setup Section. From the Azure Portol, restart your client machine. Then remote log in to it. Using the command prompt, ping your DC's private IP address and ensure that it succeeded: 

![image](https://github.com/user-attachments/assets/528d556c-1351-446e-8340-ae3e35dfea70)

Finally, open PowerShell and run **ipconfig /all**. The output for the DNS settings should have your DC's private IP address:

![image](https://github.com/user-attachments/assets/a64c0075-e20f-48a7-b02e-ca61d7f7f08b)

And with that, you have successfully completed your domain controller and client setup. In the next section below, you will be guiding through installation of Active Directory itself.




# Installation of Active Directory

Remote into your DC. Since the domain controller is running on Windows Server OS, the initial view will be different from other OS's. You should immediately see the Server Manager open upon login:

![image](https://github.com/user-attachments/assets/32481eb9-60da-4a4d-b76e-39b1dae3d715)

Next, to turn this "blank" (so to speak) server into an Active Directory server, first click on "Add roles and features". For the "Before You Begin", "Installation Type", and "Server Selection" pages, select "Next". Then when you arrive at the "Server Roles" page, you will see a long list of potential services that you could configure to run on your server. We are concerned with Active Directory Domain Services, so that is what we'll select from the list:

![image](https://github.com/user-attachments/assets/09da0585-46b7-4246-8eb5-9505ab770b1e)

Upon selecting its check mark to enable the features, a pop-up window will appear asking to add additional features that are needed for Active Directory. Click "Add Features" then "Next" back in the original window. For "Features" and "AD DS" select "Next" and then within the Confirmation section, check the option for automatic restart:

![image](https://github.com/user-attachments/assets/d836a7dd-086b-4fa9-a6f4-fbcd15709722)

Now hit "Install" and wait for it to complete. Upon completion, you should see the flag icon at the top right has a hazard sign next to it:

![image](https://github.com/user-attachments/assets/1229a7df-4375-4dc4-85fa-0f806d2d2180)

Click on it and then on the "Promote this server to a domain controller" prmopt. From there, select the "Add a new forest" option and specify your desired domain name (i.e. {yourdomainname}.com):

![image](https://github.com/user-attachments/assets/fbdc21a2-56e7-4f8a-9159-f5c4c3f0f51a)

Hit "Next" to move on to "Domain Controller Options", then set your Directory Services Restore Mode password. Then keep clicking "Next" until you arrive at the "Prerequisites Check" page. If all checks out, you should see a small banner at the top of the window saying as much:

![image](https://github.com/user-attachments/assets/bfa743ff-76c2-45c4-ab19-4e6c915d028e)

That's your green light to go ahead and hit "Install" and wait for that to come to a close. You will be logged out since the device has to restart. Upon logging back in, since your virtual machine is now properly a domain controller, your initial credentials won't work. You'll have to add your domain name and a backslash to the front of your username (mydomain.com\example). Your password will still remain as it was. Log in as described above, and then we'll move on to Internal Configuration.



# Internal Configuration

Our first step in this section will be to create a Domain Admin user within the domain


Open up Active Directory Users and Computers either by searching it in the Windows search bar, or by click the start menu then locating to Windows Administrative Tools:

![image](https://github.com/user-attachments/assets/4026b7fe-b68f-491b-99e9-8ee0aa5b9512)

Right click **on** your domain name itself then select "New" and "Organizational Unit". Name it "_EMPLOYEES" (include the underscore in the front). Using the same technique, create another OU named "_ADMINS". 

Within the Employees folder (remember, right clicking on it) create a "user". We'll name them "Jane Doe" with a username of "jane_admin". The password is up to you, just remember to keep track of it of course:

![image](https://github.com/user-attachments/assets/cb62fd21-c3aa-45ec-a55b-e2df1c423a0c)

When moving on from username to password creation, deselect the check mark next to "User must change password at next login"

![image](https://github.com/user-attachments/assets/0c26c3d0-d045-4feb-b973-55c32b043ae3)

You should now have a new user named "Jane Doe" within your employees group.

Right click on Jane Doe and select the "move" option. Move her to the newly-created Admin Security Group. Open the Admins folder to confirm the move was succesful. 


<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />

<p>
<img src="https://i.imgur.com/DJmEXEB.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
</p>
<br />
