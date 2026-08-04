## 27/07/2026
Navigated to Microsoft Azure and signed up for the Azure for Students. This subscription allowed me limited access to Microsoft Azure tools such as resource groups, virtual networks, virtual machines etc.
Created a resource group for the security operations center (SOC) home lab. Within the resource group I created a virtual network. Within the virtual network I created a virtual machine.
The virtual machine is hosted on the Microsoft servers.

In order to turn the virtual machine into a 'honeypot' to lure in real-world attackers, a couple configurations were changed.
1. Added an inbound security rule to the virtual network security group. This inbound security rule was configured to essentially allow everything: any source, any port, any destination, any protocol etc.
2. Turned off all windows defender firewall profiles.
2a. Pinged the virtual machine from own computer to confirm that we can reach the desktop via the internet. A successful ping meant that potential attackers could also reach the desktop.

## 02/08/2026
Looked through the security logs on the windows VM through the event viewer application. After having the VM running continuously there were a lot of audit failure logs, meaning that attackers were trying to interact with the VM.
Created a log analytics workspace in Microsoft Azure to receive the logs from the VM.
Added Microsoft Sentinel (SIEM of choice as it is built into Microsoft Azure already) to the newly created log analytics workspace. At this point, the log analytics workspace with Sentinel is not connected to the VM so we need to do that.

Up until this point I was following a guide on Microsoft Azure, however the guide was out of date. 
At this step the goal was to create a data collection rule which would link the VM to the log analytics workspace and Microsoft Sentinel. 
In the guide they had to download 'Microsoft Windows Security Events via AMA'. However after exploring Azure, I found out that they had built in the data collection rule into the log analytics workspace.

Configured and added a data collection rule (DCR) to the log analytics workspace. Whilst configuring the rule, I selected the relevant VM, which data logs I wanted to be collected/forwarded, and where the rule should pull the data logs from (Microsoft event viewer).
Although previous logs are not forwarded to the DCR, new logs will be forwarded to the newly created DCR. Now the VM and DCR and Microsoft Sentinel are connected.

## 04/08/2026
After trying to run a query to show all the logs forwarded from the VM, I expected all the logs to show however there were 0 results.
After troubleshooting for a bit, it seems that when I tried to access the content hub in Microsoft Sentinel, the page did not show up properly. Which is why I thought the guide was out of date. After reloading the content hub page to get it to display properly, I downloaded Windows Security Events. Then I navigated to the 'Data Connectors' tab, and within the 'Windows Security Event via AMA' I created a new DCR with the same configurations as I had before. 

So after creating the new DCR and comparing it with the previous DCR, both were created in the resource group. However, the DCR that I created using 'Windows Security Event via AMA' was actually connected to a resource (1st image). Whereas the previous DCR was only connected to a data source but not a resource (2nd image). The new DCR also had an automatically created tag that read 'createdBy: Sentinel'
<img width="260" height="115" alt="image" src="https://github.com/user-attachments/assets/f0cbe316-f2dd-40ba-a40b-1bc435173aae" />
<img width="252" height="108" alt="image" src="https://github.com/user-attachments/assets/151c7492-04dd-4d06-841e-7aafbbf84cff" />

To double check that the VM is connected to Microsoft Sentinel I navigated the the VM's 'Extension & Application' page and found that 'AzureMonitorWindowsAgent' had been automatically installed by Sentinel when we created the DCR.
