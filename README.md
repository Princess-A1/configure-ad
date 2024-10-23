<p align="center">
image
</p>

<h1>On-Premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within two Azure virtual machines.<br/>


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)



<h2>Deployment and Configuration Steps</h2>

<h3>Step 1: Setup Resources in Azure</h3>

- Create two virtual machines
	- If you need help creating your virtual machines, please see my tutorial [here](https://github.com/Princess-A1/virtual-machine)
	- The first virtual machine will be the Domain Controller
		- Name: DC-1
		- Image: Windows Server 2022
		- Take note of the virtual network (vNet) that is automatically created
       
![Screenshot 164323](https://github.com/user-attachments/assets/374cc3d1-82c3-49e1-91f2-48a1ee8f7b6a)

	- Set DC-1's Virtual Network Interface Card (vNIC) private IP address to be static
		- Go to DC-1's network settings
		- Select Networking
		- Select the link next to Network Interface
		- Select IP Configurations > ipconfig1
		- Change the assignment from dynamic to static 
			- This ensures DC-1's IP address will not change
	   
![Screenshot 164131](https://github.com/user-attachments/assets/2bf567f4-cd8c-4541-9f4f-cc93b06977a0)
![Screenshot 164542](https://github.com/user-attachments/assets/249c5583-e3ec-48fa-a807-2208118b0310)
![Screenshot 164854](https://github.com/user-attachments/assets/a765d83a-a41a-401a-b34f-70c124a5f0b4)


	- The second virtual machine will be the Client
		- Name: Client-1
		- Image: Windows 10 Pro
		- Use the same resource group and vNet as DC-1

![Screenshot 165830](https://github.com/user-attachments/assets/7e807ceb-b8bf-4c7e-93ef-215812e01cce)
![Screenshot 170204](https://github.com/user-attachments/assets/3a1fa5a0-988f-4c35-830f-d2e294bfaa72)


<h3>Step 2: Ensure Connectivity Between the Client and Domain Controller</h3>

- Login to Client-1 using Microsoft Remote Desktop
- Search for Command Prompt and open it
- Ping DC-1's private IP Address (for example, 10.0.0.4)
	- Type "ping -t 10.0.0.4" into the command-line interface
	- The ping request continually  times out due to the firewall settings
		- To fix this, we need to enable ICMPv4 on DC-1's local Windows firewall

![Screenshot 170322](https://github.com/user-attachments/assets/effdd7c9-c2cd-4ebb-8091-2ed3780bdfed)

	
- Login to DC-1 using Microsoft Remote Desktop
	- Start > Windows Administrative Tools > Windows Defender Firewall with Advanced Security > Inbound Rules
	- Sort the list by protocols
	- Find "Core Networking Diagnostics" and "ICMPv4" and enable these two inbound rules

![Screenshot 170640](https://github.com/user-attachments/assets/def42f9d-24c5-4d93-adab-fc5f30e19de2)

- Log back into Client-1 and the command line will automatically begin pinging DC-1 successfully
    
![Screenshot 170520](https://github.com/user-attachments/assets/1a722a59-ce07-4868-a0d0-5bd1edcf9210)


<h3>Step 3: Install Active Directory</h3>

- Log back into DC-1
	- Open Server Manager
	- Select "Add Roles and Features" > Follow the prompts
	- At Server Roles, check "Active Directory Domain Services."
		- Ignore how the picture below already says "Installed"
	- Select Add Features > select Next
	- Complete the installation

![Screenshot 170846](https://github.com/user-attachments/assets/da0c7903-627a-4f6a-89b4-3a36ab147961)
![Screenshot 170938](https://github.com/user-attachments/assets/ab870e25-aeae-447a-b5f0-2eeb106f74b1)

- At the top right of the Server Manager Dashboard, click on the flag
- Select "Promote This Server to a Domain Controller"

![Screenshot 171126](https://github.com/user-attachments/assets/751397ff-b7f8-46bd-9990-88e97ee817a6)
	
 - Select "Add a New Forest"
 	- Root domain name: mydomain.com
- Select Next
- Create a password
- Select Next and follow the prompts
- Select Install to complete the installation

![Screenshot 113132](https://github.com/user-attachments/assets/76cef94d-ad36-4130-a9dc-b32c7e4a52e1)
	
- DC-1 will automatically restart
- Log back into DC-1 as user: mydomain.com\labuser               

![Screenshot 113301](https://github.com/user-attachments/assets/6b9399af-9126-4262-bafa-356506d2e97a)


<h3>Step 4: Create an Admin and Normal User Account in Active Directory v1.15.8</h3>
     
- On DC-1, open Server Manager
- Click Tools at the top-right of the screen
- Select Active Directory Users and Computers

![Screenshot 113334](https://github.com/user-attachments/assets/f4857405-5f84-49a5-aea9-914fd95b15dc)
	
- Right-click mydomain.com > New > Select Oranizational Unit (OU)
- Create two OUs
	- Name the first "_EMPLOYEES"
	- Name the second "_ADMINS"
	
![Screenshot 113448](https://github.com/user-attachments/assets/88add553-2cc8-4ae9-ba3e-c6987be62723)
	
- Right-click mydomain.com and click Referesh to sort the new organizational units to the top
- Go to the _ADMINS OU
- Right-click the name of the OU > New > User
	- First/Last name: Jane Doe
	- User login name: jane_admin
	- Select Next
	- Create a password
	- Uncheck all boxes
	- Select Next and then select Finish
![Screenshot 113622](https://github.com/user-attachments/assets/20b8f246-7700-4195-8df4-313aa2fcb15b)
![Screenshot 113736](https://github.com/user-attachments/assets/d79f0d87-f4f9-4970-8546-8f32d564978c)

- Go to the _ADMINS OU
- Right-click Jane Doe > select Properties
	- Click the tab named "Member of" > select Add
	- Type in the names of your domain administrators
	- Select "Check Names" > OK > Apply
- Log out of DC-1 as "labuser" and log back in as “mydomain.com\jane_admin”

![Screenshot 114027](https://github.com/user-attachments/assets/27a9556e-d57c-4bef-987a-d65dcf2a47f3)
![Screenshot 114156](https://github.com/user-attachments/assets/476edb52-a11e-44e3-bfeb-1be3b2026a16)
 

<h3>Step 5: Join Client-1 to your domain (mydomain.com)
</h3>

- Go back to the Azure portal
- Navigate to the Client-1 Virtual Machine
- On the left-hand side of the screen select Networking
- Select the link next to the NIC > select DNS Server > Custom
- Type in DC-1's private IP address
- Click Save
- After it is done updating, select Restart and select Yes

![Screenshot 114526](https://github.com/user-attachments/assets/b808428b-11c4-4863-b89c-050535ea28d8)
![Screenshot 114728](https://github.com/user-attachments/assets/636cffe9-4a8d-42bf-a584-58dfbff59a53)
![Screenshot 114935](https://github.com/user-attachments/assets/7449d9a5-4dfb-4f3e-97f3-ac7e4fc0ffc8)

- Log back into Client-1 using Microsoft Remote Desktop as the original local admin (labuser)
- Right-click the Start menu and select System
- On right-hand side of the screen, select Rename This PC (Advanced) > Change
- Under "Member of," select Domain
- Type "mydomain.com" and select OK
	- Username: mydomain.com\jane_admin
	- Type in password and press OK
- Restart the computer 			

![Screenshot 115056](https://github.com/user-attachments/assets/f7370d9a-6820-4ef8-a5bf-2f7fea5b3ecc)
![Screenshot 115419](https://github.com/user-attachments/assets/780c5a34-b2f3-427e-b880-1f5321c173b9)


<h3>Step 6: Setup Remote Desktop for non-administrative users on Client-1
</h3>

- Log back into Client-1
- Use mydomain.com\jane_admin
- Right-click the Start menu and select System
- On the right-hand side of the screen, select Remote Desktop
- Under User Accounts, click "Select Users That Can Remotely Access This PC > select Add
- Type in the name of your domain users
- Select "Check Names" > OK > OK

![Screenshot 115744](https://github.com/user-attachments/assets/afb999ed-1845-4610-971c-d678acf853da)
![Screenshot 120224](https://github.com/user-attachments/assets/391b455b-f95b-4d58-b687-15f9154ec2ca)


<h3>Step 7: Create as many additional users as you would like and attempt to log into Client-1 with one of the users' profiles
</h3>

- Log back into DC-1 as jane_admin
- Search for "Powershell_ise,"
- Right-click on Powershell_ise and open it as an administrator
- At the top-left of the screen select New Script and paste the contents of the following script into it
	- You can find the script [here](https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1)

![Screenshot 120549](https://github.com/user-attachments/assets/73ed5905-fbf2-4c72-a206-841fc4a5fd29)
![Screenshot 120638](https://github.com/user-attachments/assets/49f7e337-1380-415f-af89-2b599776fd31)

- Click the green arrow button near the top-middle of the screen
	- This will run the script
- Once the users have been created, go back to Active Directory Users and Computers > mydomain.com > _EMPLOYEES
		- You will see all the accounts that were created
- You can now log into Client-1 with one of the accounts that were created
	- Try logging into Client-1 as user "base.milu" using the password "Password1"

![Screenshot 120818](https://github.com/user-attachments/assets/747d3200-75bb-485b-b76d-82678dea1933)
![Screenshot 120859](https://github.com/user-attachments/assets/cd4dddc4-f674-4917-a1a0-0647a793c445)
![Screenshot 121004](https://github.com/user-attachments/assets/101851a9-a888-44ba-80b7-77dc1bdcde1a)
![Screenshot 121256](https://github.com/user-attachments/assets/af6d25ee-e184-4866-8350-b38d02a73073)
![Screenshot 121326](https://github.com/user-attachments/assets/b33a4c47-b1eb-48c3-a6db-12e672e32966)


You have implementated on-premises Active Directory and created users within an Azure virtual machine!
