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

## 05/08/2026
Queried the logs using KQL to show certain security events. Expanded some of the events and using the IP address provided in the event, used IP Geolocation to get the approximate location of the attacker. 

Added a prebuilt CSV document, containing the approximate geographic location of certain IP ranges, to the watchlist in Sentinel. This enables us to use the information in the CSV directly in the logs using the 
``_GetWatchList("[watchlist name]")`` command in KQL.

Generated a KQL query to search for all failed login events from a single attacker, showing their location.
```
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == <attacker IP address>
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
```

Created a new query section within the workbook in Sentinel to visually display the location of attackers. I used the JSON command
```
{
	"type": 3,
	"content": {
	"version": "KqlItem/1.0",
	"query": "let GeoIPDB_FULL = _GetWatchlist(\"geoip\");\nlet WindowsEvents = SecurityEvent;\nWindowsEvents | where EventID == 4625\n| order by TimeGenerated desc\n| evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network)\n| summarize FailureCount = count() by IpAddress, latitude, longitude, cityname, countryname\n| project FailureCount, AttackerIp = IpAddress, latitude, longitude, city = cityname, country = countryname,\nfriendly_location = strcat(cityname, \" (\", countryname, \")\");",
	"size": 3,
	"timeContext": {
		"durationMs": 2592000000
	},
	"queryType": 0,
	"resourceType": "microsoft.operationalinsights/workspaces",
	"visualization": "map",
	"mapSettings": {
		"locInfo": "LatLong",
		"locInfoColumn": "countryname",
		"latitude": "latitude",
		"longitude": "longitude",
		"sizeSettings": "FailureCount",
		"sizeAggregation": "Sum",
		"opacity": 0.8,
		"labelSettings": "friendly_location",
		"legendMetric": "FailureCount",
		"legendAggregation": "Sum",
		"itemColorSettings": {
		"nodeColorField": "FailureCount",
		"colorAggregation": "Sum",
		"type": "heatmap",
		"heatmapPalette": "greenRed"
		}
	}
	},
	"name": "query - 0"
}
```
So here is where the guide video ends. However there is still a lot more to learn. Next steps, I would like to build an Automated Triage Playbooks (SOAR), write Custom Analytics Rules, expand the Attack Surface with a Linux Honeypot, integrate Threat Intelligence (IoCs).
