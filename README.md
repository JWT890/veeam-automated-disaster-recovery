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
Then go to a second computer, can be Linux and go to Network and find the similar Wifi network    