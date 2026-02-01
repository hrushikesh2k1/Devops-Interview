“Azure Resource Graph allows us to query all Azure resources across subscriptions using KQL. 
We use it to generate inventory reports, find unused resources like unattached disks, 
identify stopped VMs, audit public IPs, and ensure governance using tags. 
The results can be exported to CSV or pinned to dashboards for management and cost control.”

Examples:
1. To list all resources
   ```KQL
   Resources
   | project name, type, location, resourceGroup, subscriptionId
   ```
2. Count all resources
```KQL
Resources
| summarize count()
```
3. Count resources by type
```KQL
Resources
| summarize count() by type
| order by count_ desc
```
4. List all virtual machines
```KQL
Resources
| where type == "microsoft.compute/virtualmachines"
| project name, location, resourceGroup
```
5. Find stopped or deallocated VM's
```KQL
Resources
| where type == "microsoft.compute/virtualmachines"
| extend powerState = tostring(properties.extended.instanceView.powerState.code)
| where powerState contains "deallocated"
```
