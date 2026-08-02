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
