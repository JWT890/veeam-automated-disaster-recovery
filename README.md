# veeam-automated-disaster-recovery

Disaster recovery is important since being able to recover from a disaster is important so that no data or saves are lost in the event of a disaster where important information that is priceless is lost

# Setup
This will require two computers this a main computer running VMWare and a second computer acting as a backup for disaster recovery. 
Start by on the first computer going to Settings -> Network -> Wi-Fi and click on properties and see this part: 
![Settings](./images/settings.png)  
Click on edit and be presented with the edit network IP settings pop up and change it from automatic to manual and select the option of IPv4 and see this:  
![Settings1](./images/settings1.png)
Put this for the boxes ->   
IP: 192.168.1.10    
Subnet: 255.255.255.0   
Gateway: 192.168.1.1    
DNS: 192.168.1.1 or 8.8.8.8 
Then go to a second computer, can be Linux and go to Network and find the similar Wifi network by going to the search menu and find Network, click on settings icon for Wi-FI and in the pop up select IPv4 and see this screen:
![Linux](./images/linux.png)    
Then enter this information ->
IP: 192.168.1.20    
Subnet: 255.255.255.0   
Gateway: 192.168.1.1    
DNS: 192.168.1.1    

If 192 doesn't work, then on Linux type ip addr to see Wi-Fi address, then hostname -I to get route along with ip route to see what the route is, likely will depend on what your router setup is.  
In this instance lets change it to, on Linux, to this:  
IP: 10.0.0.20   
Mask: 255.255.255.0 
Gateway: 10.0.0.1   
DNS: 10.0.0.1   
Then type nmcli connection show to see available networks, then type these commands:    
nmcli connection modify "Wi-Fi Name" ipv4.addresses 10.0.0.20/24    
nmcli connection modify "Wi-Fi Name" ipv4.gateway 10.0.0.1  
nmcli connection modify "Wi-Fi Name" ipv4.dns "10.0.0.1"    
nmcli connection modify "Wi-Fi Name" ipv4.method manual 
nmcli connection up "Wi-Fi Name".   
Last command should reset it and get it back up again.  
Then on Windows do this:    
IP: 10.0.0.10   
Mask: 255.255.255.0 
Gateway: 10.0.0.1   
DNS: 10.0.0.1   
Then on the Windows computer, go to Windows Defender Firewall and click on Advanced settings on the left:   
![Settings2](./images/settings2.png)    
Then scroll down till you see File and Printer Sharing and find the private one and right click on it and click on enable rule: 
![Rule](./images/rule.png)  
Then go to the Linux computer and ping the Windows computer
![Ping](./images/ping.png)  
Then on the Linux computer go to the command line and type sudo apt install samba -y, then type sudo ufw status to see the status, if off, type sudo ufw enable and rerun to see it if turned on. Next type sudo ufw allow 445/tcp, then type sudo ufw allow samba, then sudo ufw reload to reload and check the ufw status once again: 
![Status](./images/status.png)  
Then type mkdir -p ~/VeeamBackups to create the directory and type sudo nano /etc/samba/smb.conf after getting into it. 
Then scroll all the way down to the end of the smb.conf file and add this:  
![Setup](./images/setup.png)    
Then save it and in the command line type sudo smbpasswd -a your-username   
sudo systemctl restart smbd 
sudo systemctl enable smbd
Then go to the Windows computer in the file folder and type \\10.0.0.20\VeeamBackups here:  
![Bar](./images/bar.png)    
Then press enter and a login should pop up. Type in your Linux user and SMB password and should end up here:    
![Folder](./images/folder.png)  
Then go to VMWare Workstation and create a Proxmox VM in it by going into VMWare and click on file and clicking on new virtual machine and go through the setup such as Debian 12 64 bit for Guest OS, name of Proxmox VE, give it 8192 MB for memory, bridged adapter for network and 100 GB for the disk and click on create. 
Then go to settings -> Proccessors and enable the Virtualize Intel VT-x/EPT or AMD-V/RVI like below:    
![Virt](./images/virt.png)  
Then power on the VM and get it set up. If it doesn't work in VMWare, get it setup in VirtualBox and then go to go the Command prompt and type cd "C:\Program Files\Oracle\VirtualBox" and then type .\VBoxManage.exe modifyvm "Proxmox VE" --nested-hw-virt on and should enable Nested Virtualization.    
Then for the network in Proxmox:    
![Network](./images/network.png)    
Then wait for a few minutes to set it up and take note of the IP address and enter in into your browser and login with root and the password set:   
![Proxmox](./images/proxmox.png)    
Next go download a Ubuntu Server VM from: https://ubuntu.com/download/server    
Then for the VM we will be working with go to local (pve) and click on ISO image and click on upload and find the Ubuntu Server ISO that was downloaded and click on upload and wait for a few minutes for the transfer to occur:   
![Iso](./images/iso.png)    
Then click on the Create VM blue button in the right corner to create the Linux VM. 
Linux VM:   
Name: Test-Lab  
RAM: 8142 MB    
CPU: 2 Cores    
Disk: 50 GB 
Network: vmbr0  
Then when done make sure to turn off KVM hardware virtualization since it isn't Windows 11. Good configuration for it:  
![Config](./images/config.png)  
![Config1](./images/config1.png)    
Then click on start and wait for it initialize. 
After waiting for a few minutes, you will reach this:   
![Screen](./images/screen.png)  
Then go through the process of the steps of getting everything set up, then choose a name, server name, username and password. Then wait for it to finish setting up which will likely take a while:    
![Login](./images/login.png)    
Then on the second computer's end, go to VirtualBox and create a VM.    
First download Windows Server from this link, either 2019 or 2022 will do: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019. 
For the VM: 
Name: WinServer-Veeam   
Memory: 4046 MB
2 Processors    
EFI Enabled 
Bridged Adapter for Adapter 1   
80 GB for the disk and dynamic. 
Next get the VM up and running to get the Server on the second computer up  
In the VM on the Windows setup, choose the option of Windows Server 2019 edition desktop like so:   
![Setup1](./images/setup1.png)  
Then click on custom install which will take you to the disk space that was allocated and click on it and next which will take you to the installation screen:  
![Install](./images/install.png)    
Then wait a while for it to get up and install. 
After waiting a while, this screen will pop up: 
![Windows](./images/windows.png)    
Then set the admin password and hit finish and login to the Windows Server VM   
![Machine](./images/machine.png)    
Then open up a browser in the VM and download Edge or Chrome to not use Internet Explorer, then go to veeam.com/downloads to get Veeam Backup & Replication Community Edition or if downloaded previously, find the email and download from there and wait around 30-45 minutes for it to finish.   
After waiting a while, it will have downloaded and then right click on the ISO download 
![Option](./images/options.png) 
Then click on mount to mount the iso and see this afterwards    
![File](./images/file.png)  
Then click on setup and have it run which will have a popup to click next with this next:   
![Veeam](./images/veeam.png)    
Click on the first one and wait for it to initalize and click on accept for the license agreement, click on next on license screen and leave alone which will come up with a system configuration check screen and wait a while.    
After waiting a while, this screen will pop up: 
![More](./images/more.png)  
Click on next which you take you to this screen:    
![More1](./images/more1.png)    
Clcik on next going through the screens and if it asks about tight disk space, select yes and continue and click install when there and see this screen:    
![Install1](./images/install1.png)  
With it installing, it will be a while for it to get set up.    
After a while, this will pop up:    
![Failure](./images/failure.webp)   
Keep on clicking on retry or cancel until this pops up: 
![V](./images/v.png)    
Click on click here to open this folder for it: 
![Temp](./images/temp.png)  
Then run the service command to see what ones are not running:  
![V1](./images/v1.png)  
![V2](./images/v2.png)  
Then go to the start menu and look up Veeam Backup & Replication Console and open it up 
![V3](./images/v3.png)  
Then see this:  
![Backup](./images/backup.png)  
Click connect which will likely result in this: 
![Backup2](./images/backup2.png)
Then in Powershell admin run:   
Start-Service VeeamDistributionSVC, then Start-Service VeeamMountSVC, then VeeamWebSVC, then run Get-Service VeeamWebSvc, VeeamDistributionSvc, VeeamMountSvc | Select-Object Name, Status to get this: 
![Start](./images/start.webp)   
![Run](./images/run.webp)   
The red and stopped means that VeeamWebSVC is still not running, run the following in this image:   
![Run1](./images/run1.webp) 
The error log means that its taking a long time to initialize and is being killed in the process, It could also be this if running to check memory in VM:   
![Mem](./images/mem.webp)   
Turn off the VM and increase the base memory to 8192 MB of storage and power it back on. 
When back on run Get-Service VeeamWebSvc | Select-Object Name, Status and if it says stopped, run Start-Service VeeamWebSvc wait a few minutes then run Get-Service VeeamWebSvc | Select-Object Name, Status and should be running. 
Then click on the Backup and Replication one and click connect and click on trust this server and will prompt a login with username and password or sign in as current user. Click on current user and will load up into this:  
![Up](./images/up.png)  
Then go to the first computer and open up the console and type this:    
pveum user add veeam@pam --comment "Veeam Backup Service Account"   
This will create the account for the service one, next will be the roles:   
pveum role add VeeamBackup -privs "VM.Audit,VM.Backup,VM.Config.Disk,VM.Config.CDROM,VM.Config.CPU,VM.Config.Memory,Datastore.AllocateSpace,Datastore.Audit"    
Which will assign the roles to the account, next type:  
pveum aclmod / -user veeam@pam -role VeeamBackup.   
Then type pveum passwd veeam@pam which should pop up with this: 
![Error](./images/error.png)    
The solution is to to this: 
useradd -m veeam and pveum passwd veeam@pam and type the password to set which should work. 
Then go to the second computer in the VM if turned off make sure Veeam services are running, start any not running and run these commands to keep them up and then reboot:  
![C](./images/c.png)    
Then with Veeam Backup & Restore back up, click on the Backup Infrastructure button and click on backup repositories:   
![D](./images/d.png)    
And then click on add repositories and see this:    
[Options1](./images/options1.png)   
Click on Network Attached and see the option to click on SMB share which will pop up with this:q
![Name](./images/name.png)   
Name it LinuxMint-Samba-Repo and hit next and get here: 
![New](./images/new.png)     
For the share put \\10.0.0.20\VeeamBackups and enable the requires access credentials option and click on add and enter the username and password for the share which should be accepted and click next done till apply to review and then click apply and wait a few minutes.  
After a few minutes:    
![Done](./images/done.png)  
Then click on finish and it will pop up:    
![B2](./images/b2.png)  
