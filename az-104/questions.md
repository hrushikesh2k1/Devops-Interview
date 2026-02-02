Is this the new role or replacement role?
When i am successful in this interview what is my focus going to be for first 90 days
How do you measure performace of the team over 6 months 

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
  
8. How to troubleshoot the DNS issues?
   steps:
   1. Log in into server, open DNS manager, right click on server and click on properties.
   2. There you can see Debug logging.
   3. Check the Log packets for debugging. specify the path.
   4. This will create a detailed log
   5. clear the cache DNS, ipconfig /flushdns
   6. Also, you can check the logs in event viewer>system> or Event viewer>Applications and service logs>DNS server

9. Troubleshooting Domain delegation issues?
   steps:
   1. Delegation should always done by public DNS name
   2. You should be creating the record in DNS server under Forward Lookup Zone, click on create Zone, after that add a Host record (A)

10. On-premises systems cannot authenticate to azure resources?
    steps:
    1. The first thing to look at is your tenant license, cause license comes with features, more premium the license more the features.
    2. On-premises users can't able to authenticate to azure, this is with Firewall. cause there are certain number of azure url's that need to be opened by firewalls for authentication.

11. How do you troubleshoot azure ad configuration issues?
    steps:
    1. Assign a usage location to users when creating to assign license.
    2. Applications which are authorize to authenticate with azure ad are showed in Enterprise apps.
   
12. Troubleshooting SSPR issues?
    steps:
    1. Microsoft 365 business premium, Azure ad premium p1 and p2 licenses only have the capability for SSPR. For Hydrid user password change, they need P1 or P2 license.
    2. SSPR is not automatically turned ON, you should Turn it ON.
    3. We have diagnose and solve problems blase in password reset menu.
   
13. Authentication of Hybrid identity
    types:
    1. Managed Authentication: Azure AD handles the authentication process by using locally stored hashed version of the password or sends the credentials
       to an on-premises software agent to be authenticated by the on-premises ADDS.
       1. Password hashed authentication
       2. Pass-through authentication
    Both can be used simultaneously
    2. Federated Authentication: Azure Ad redirects the client computer requesting authentication to another identity provider. When you want complex authentication (smart cards, etc.,)
   
14. Troubleshooting MFA issues?
    steps:
    1. For azure ad free, NO MFA option.
    2. Even after having proper license, need to enable MFA by "per-user MFA"

15. Troubleshooting Azure AD connect issues?
    steps:
    1. Go to azure ad> azure ad connect> Azure ad connect health>Duplicate Attribute error, Data mismatch, Data validation error...
    2. ADDS services agent should be installed in DC to see other errors
    3. You can check on troubleshoot blade in same azure ad connect
    4. Running the troubleshooting task in Azure ad connect wizard in DC, it will run the powershell script, generates the report for Object sync issues.
   
16. Troubleshooting issues with PHS and PTA
    steps:
    1. Check whether the sync status is enabled.
    2. Check PTA is enabled in same azuread connect page in azure portal
    3. check sync errors
    4. Ensure you are downloaded health agent is installed in server.
   
17. Explain Application proxy?
    Application proxy is a feature of azure ad that enables users to access on-prem web application from a remote client
    Azure ad application proxy sits in the middle of users and the on-prem web for authentication
    By this, our on-prem web is hosted internally by not ocnfiguring ports, secured inside, helps not setting up DMZ and reverse proxy setup
    Microsoft provides SaaS like service to on-prem web application, it is very secure
    Also it is not required to open any ports

18. Explain how application proxy works?
    1. After the user has accessed the application through endpoint, the user is directed to the Azure AD sign-in page.
    2. After successful sign-in, Azure AD sends a token to the user's client device.
    3. The client sends the token to the Application Proxy Service which retrieves the user principal name and security principal name from the token.
    4. Application Proxy then sends the request to the Application Proxy Connector.
    5. If you have configured Single Sign-On, the Connector performs any additional authentication required on behalf of the User.
    6. Then the Application Proxy Connector connects the User to the On-Prem Web Server.

19. Troubleshooting the issues with Application proxy service?
    steps:
    1. Check whether application proxy agent is installed in server, check in application proxy menu in Azure ad
    2. You can try running powershell cmd "Get-EventLog application -source "Microsoft AAD Application proxy Connector" ", check the microsoft page for error and recommended steps
       1. certificate expiry
       2. check if you are the local admin

20. Troubleshoot the NSG configuration issues?
    steps:
    1. Open nwteork watcher. we have IpVerifyFlow, NSG Diagnostics, Next hop, Effective routes,
    2. You can create the NSG flow logs for VM to check the logs, you can Enable traffic analytics using log analytics workspace.
    3. You can see whole traffic analaysis to your VM using Traffic analytics blade under network watcher. (shows Total traffic, malicious traffic, blocked traffic, frequent conversations)
       
21. Explain ASG?
    In Azure, an Application Security Group (ASG) lets you group Virtual Machines (VMs) by application role (like 'web,' 'app,' 'db'),
    simplifying Network Security Group (NSG) rules by referencing these logical groups instead of individual IP addresses.

22. Explain Firwall?
    A firewall is a network security system—available as hardware or software—that monitors and controls incoming and outgoing network traffic based on predetermined security rules.
    we have
    1. Network rules: That allows to define inbound and outbound traffic rules
    2. DNAT: Destination Network address translation, lets to map the public Ip of firewall to private IP of VM. remote users hits the ip of firewall
    3. Application rules: An Application Rule in Azure Firewall is a Layer 7 (application layer) filtering rule designed to allow or deny outbound traffic based on
       Fully Qualified Domain Names (FQDNs), URLs, and protocols (HTTP, HTTPS, or MSSQL). Unlike network rules, they inspect application-layer traffic, allowing specific
       website access (e.g., *.microsoft.com) rather than just IP/Port combinations.
    4. You can send the firewall diagnostic log data to log analytic workspace
   
23. Troubleshoot azure firewall manager configuration issue?
    Azure firewall manager is thep place where we can know if any vNet is not protected by firewall and for troubleshooting

24. Troubleshooting VM latency issues?
    steps:
    1.  Go to the azure portal>VM>Monitoring section-- you can see the various metrics showing in graphs. If you want to see what are all the process using the VM in the moment,
        you can RDP into VM, go to resource Monitor under tools, Now you can check the specific executable using the CPU, DISK, Memory and Network.
    2. To check the latency, use ping <ip_add> (you can see time=66ms)
    3. To test the latency for VM's running on Windows, download latte.exe, allow through windows defender firewall
       Create folder called tools under C drive and copy the file.
    4. Ensure resource health of VM is healthy.
    5. Use diagnose and solve problems blade and run the troubleshoot.
    6. Go to Network watcher> connection troubleshoot balde. checks the connectivity and latency details.
    7. You can also check packet capture under network watcher to check any heavy traffic on your resources in case of DDOS attack, but you have to install wireshark for this

25. Troubleshoot bandwidth availability issue?
    steps:
    1. Azure gives some network requirements where we have to check the variables from our end like, our internet connectivity need to support 40 Mbps downstream and 5 Mbps upstream for single user session
    2. Avoid competing bandwidth intense traffic in the same network like video gaming
    3. Make sure you are connected to your nearest data center to less latency
    4. Run performance test in the system
    5. You can see the various metrics going to azure monitor service.(packets sent, bytes sent,..)
    6. If the packets sent/byte sent metrics continously receving high traffic, next step would be going to server>resource monitor and check which executable is actually receving the huge traffic, identifying
       the port. (might be brute force attack)
       What should be the next you find brute force attack
       1. Block the attacker's IP(find the malicious IP by going to Traffic analytics under Network watcher) under firewall policies.
       2. Using Azure bastion or VPN to a secure connection instead of directly opening port to internet.
       Next step is Assuming breach
       1. Identify successful logins that occured during the attack window.
       2. Change the credentials for all user accounts especially admin ones
       3. Revoke all active sessions to terminate any unuthorized access.
       4. Sacn for any backdoors, check for any newly created accounts, scheduled tasks, unfamilier scripts
       Prevent recurrence
       1. Enforcing MFA
       2. Configuring account lockout policies: system should lock account after a low number of failed attempts
       3. Enable automated threat response
       Post-incident action
       1. Monitor system logs to confirm attack is ceased
       2. if system is compromised considering restoring from a known good backup

 26. How to monitor the downtime/SLA in regards to V<'s
     steps:
     1. Go to Network watcher> Connection monitor> --this runs scheduled test to check the downtime.

27. Interpret Route tables
    
