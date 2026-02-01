1. How do you troubleshoot Azure VM backup issues including restarting a failed backup.
   Points to check:
   1. Need to check if the VM is fully provisioned and the power state is running.
   2. Backup of VM cannot happen when VM is going through updates.
   3. Checking if any antivirus software currently performing a scan. This software prevents from taking a backup
   4. C drive>WindowsAzure>Logs>Plugins>WindowsAzureReciveryService. if this folder do not exit, then we should understand that VM is not provisioned fully.
   5. You can stop the backup, delete the backup and undelete the backup and then restarting the backup if backup is throwing more and more errors.
   6. you can install the backup agent in this case, but we need to check MARS (Microsoft Recovery Agent Service) agent compatibility with the VM.

2. How to create a back of azure resources?
   Steps for taking a backup
   1. Create a recovery vault service
   2. Add the service which you want to take the backup (Azure fileshare, VM,..)
   3. You can go to centralized service called "backup center" for all your backup items

3. Installing the MARS agent
   steps:
   1. Download the Agent from google
   2. Ask for Vault credentials----you need vault file/authentication file which tells you have permissions for recovery vault.
   3. click next next next, choose files to be backedup, choose retention policy.
   4. if there is a backup issue, just uninstall and reinstall the agent
   5. Next troubleshooting step is, check if backup agent service agent is running. win+r>services.msc
   6. check C drive>WindowsAzure>Logs>Plugins>WindowsAzureReciveryService. this should exists
   7. Check Event logs, check system logs errors, cloud backup logs

4. Troubleshooting scheduled backup issues?
   Things to note:
   1. Always know the retention period of the policies set for the backup in backup policies in backup center.
   2. Recovery vault and Vm must be in same region
   3. if you get a error with VM, mostl probably the VM is not started.
  
5. What are the one most important things you need to take when replicating things from on-premise to azure or between azure regions in case of site replication errors?
   1. Need to check if firewall is blocking outbound connectivity for URL's for site recovery
   2. Checking NSG rules on both ends (443 port is allows for HTTPS traffic).

7. How do you do VM replication to protect your azure VM for disaster recovery by replicating to another azure region?
   steps:
   1. Create a VM in East US region in different RG
   2. Create Recovery service vault in West US region in different RG
   3. Under recovery vault that is created select "site recovery".
   4. Fill source details-->select virtual machine-->replication settings->click and review
   5. you can test Failover in replicated items in recovery services vault
   6. You can find the detailed alerts, description, affected items going to recovery vault/alerts section
   7. 
  
8. 
