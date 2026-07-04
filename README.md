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
