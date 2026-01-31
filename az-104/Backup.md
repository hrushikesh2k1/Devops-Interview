# Azure Backup & Recovery Services Vault

## Purpose of this Module

Backup and Disaster Recovery are critical for every organization.
Microsoft Azure provides built-in services to:
- Take backups
- Schedule backups
- Replicate data
- Restore data
- Perform disaster recovery

This is done using a service called **Recovery Services Vault**.

---

## What is Azure Recovery Services Vault?

Recovery Services Vault is an Azure service used to:
- Take backup of:
  - Azure Virtual Machines
  - Azure File Shares
  - SQL Databases
  - SAP / SAP HANA
  - On-premises servers
  - Physical machines
  - Virtualized workloads (VMware, Hyper-V)

It is the central place where all backups and recovery operations are managed.

---

## Why Recovery Services Vault is needed

Without it:
- You cannot back up Azure VMs
- You cannot restore deleted or corrupted data
- You cannot perform disaster recovery

With it:
- Azure takes automatic backups
- Data is stored safely in Azure
- You can restore data anytime

---

## Step 1 – Create a Test Virtual Machine

A virtual machine is needed for testing backup.

Example:
- VM Name: Web
- Region: Central US
- OS: Windows Server 2016 or 2019
- Ports enabled:
  - 3389 (RDP)
- Username & Password required

This VM will be used to test backup and restore.

---

## Step 2 – Create Recovery Services Vault

Go to:
Azure Portal → Search → Recovery Services Vault → Create

Example:
- Name: training-recovery-vault
- Region: Same as VM
- Redundancy: GRS (Geo-Redundant Storage)
- Encryption: Microsoft Managed Key

This vault stores backup data.

---

## Step 3 – Open Recovery Services Vault

Inside the vault you will see:
- Backup
- Site Recovery
- Backup Items
- Monitoring
- Backup Jobs

This is where all backup operations are managed.

---

## Step 4 – Configure Backup

Go to:
Recovery Services Vault → Backup → Get Started

Choose:
- Where is your workload running? → Azure
- What do you want to back up? → Azure Virtual Machine

Select the VM: **Web**

---

## Step 5 – Backup Types

Azure provides two types of VM backup:

### Enhanced Backup
- Multiple backups per day
- 30 days retention
- Supports:
  - Trusted launch VMs
  - Premium SSD
  - Ultra Disk

### Standard Backup
- One backup per day
- Minimum 5 days retention

---

## Step 6 – Create Backup Policy

You can define:
- How often to take backup
- When to take backup
- How long to keep backup

Example Policy:
- Daily at 8:00 PM
- Retention: 30 days

This means:
Each backup will be kept for 30 days before deletion.

---

## Step 7 – Enable Backup

After selecting:
- VM
- Policy

Click:
Enable Backup

Azure will start protecting the VM.

---

## Step 8 – Check Backup Status

Go to:
Recovery Services Vault → Monitoring → Backup Jobs

This shows:
- Backup in progress
- Backup completed
- Failed jobs

---

## Step 9 – Run Backup Manually

If you want immediate backup:

Go to:
Recovery Services Vault → Backup Items → Azure Virtual Machine → Web

Click:
Backup Now

This creates an instant recovery point.

---

## Key Exam Points

- Recovery Services Vault is required for Azure backups
- Supports Azure and On-premises resources
- Supports VM, SQL, SAP, File Shares
- Backup can be scheduled or manual
- Backups can be restored later

---

## Interview One-Line Answer

Azure Recovery Services Vault is a centralized Azure service used to manage backup, restore, and disaster recovery for Azure and on-premises workloads.

---

End of Notes
