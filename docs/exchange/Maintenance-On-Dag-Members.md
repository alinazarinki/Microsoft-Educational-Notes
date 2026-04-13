### Performing Maintenance on DAG Members in Exchange Server

Before performing any software or hardware maintenance on a **DAG member**, you must first place the server into **maintenance mode**. Exchange Server provides two official scripts to safely handle DAG maintenance:

- **StartDagServerMaintenance.ps1**: Moves all active databases off the server, relocates critical DAG roles (such as the Primary Active Manager – PAM), and prevents them from moving back until maintenance is complete.
- **StopDagServerMaintenance.ps1**: Takes the DAG member out of maintenance mode and makes it an active target for databases and critical DAG roles again.

Both scripts support the following parameters:
- `-ServerName` (hostname or FQDN)
- `-WhatIf`

The scripts can be executed locally or remotely. The computer running the scripts must have **RSAT-Clustering** (Failover Cluster Management tools) installed.

> **Note**: The `RedistributeActiveDatabases.ps1` script is also available to redistribute active databases based on Activation Preference. However, starting with Exchange 2016 CU2, the DAG property `PreferenceMoveFrequency` automatically balances database copies. You only need to use `RedistributeActiveDatabases.ps1` if you have disabled this feature or want to perform manual balancing.

#### What StartDagServerMaintenance.ps1 Does

This script performs the following tasks:

- Sets `DatabaseCopyAutoActivationPolicy` to **Blocked** on the server (prevents automatic activation of database copies).
- Pauses the node in the Windows Failover Cluster (prevents it from becoming the PAM).
- Moves all active mailbox databases to other DAG members.
- If the server owns the Default Cluster Group, it moves the group (and therefore the PAM role) to another DAG member.

If any step fails, the script automatically rolls back all changes except for successfully moved databases.

---

### 1. Putting a DAG Member into Maintenance Mode

Follow these steps to prepare the server for maintenance (including draining transport queues and suspending client connectivity):

1. Check current DAG status and Primary Active Manager:
   ```powershell
   Get-DatabaseAvailabilityGroup -Status | Format-List Identity, PrimaryActiveManager
   ```

2. Drain the transport queues:
   ```powershell
   Set-ServerComponentState <ServerName> -Component HubTransport -State Draining -Requester Maintenance
   Restart-Service MSExchangeTransport
   ```

3. (Exchange 2016 only) Drain Unified Messaging calls:
   ```powershell
   Set-ServerComponentState <ServerName> -Component UMCallRouter -State Draining -Requester Maintenance
   ```

4. Navigate to the Exchange scripts folder:
   ```powershell
   CD $ExScripts
   ```

5. Run the maintenance script:
   ```powershell
   .\StartDagServerMaintenance.ps1 -ServerName <ServerName> -MoveComment "Maintenance" -PauseClusterNode
   ```
   > **Note**: This script may take several minutes to complete and might not show progress on the screen.

6. Redirect any remaining messages in the queues:
   ```powershell
   Redirect-Message -Server <ServerName> -Target <TargetServerFQDN>
   ```

7. Place the server into full maintenance mode:
   ```powershell
   Set-ServerComponentState <ServerName> -Component ServerWideOffline -State Inactive -Requester Maintenance
   ```

---

### 2. Verifying the Server is Ready for Maintenance

Run the following commands to confirm the server is fully prepared:

```powershell
# Check PAM location
Get-DatabaseAvailabilityGroup -Status | Format-List Identity, PrimaryActiveManager

# Verify server components (only Monitoring and RecoveryActionsEnabled should be Active)
Get-ServerComponentState <ServerName> | Format-Table Component, State -AutoSize

# Check activation policy
Get-MailboxServer <ServerName> | Format-List DatabaseCopyAutoActivationPolicy

# Verify cluster node is paused
Get-ClusterNode <ServerName> | Format-List

# Confirm transport queues are empty
Get-Queue
```

---

### 3. Taking the DAG Member Out of Maintenance Mode

After maintenance is complete, use the `StopDagServerMaintenance.ps1` script to return the server to production. This script performs the following:

- Resumes the cluster node
- Sets `DatabaseCopyAutoActivationPolicy` back to **Unrestricted**
- Resumes all mailbox database copies on the server

Follow these steps:

1. Bring the server back online for client connections:
   ```powershell
   Set-ServerComponentState <ServerName> -Component ServerWideOffline -State Active -Requester Maintenance
   ```

2. (Exchange 2016 only) Re-enable Unified Messaging:
   ```powershell
   Set-ServerComponentState <ServerName> -Component UMCallRouter -State Active -Requester Maintenance
   ```

3. Navigate to the scripts folder:
   ```powershell
   CD $ExScripts
   ```

4. Run the stop maintenance script:
   ```powershell
   .\StopDagServerMaintenance.ps1 -ServerName <ServerName>
   ```

5. Re-enable transport services:
   ```powershell
   Set-ServerComponentState <ServerName> -Component HubTransport -State Active -Requester Maintenance
   Restart-Service MSExchangeTransport
   ```

---

### 4. Final Verification – Ready for Production

Run these commands to confirm the server is fully operational:

```powershell
# Check DAG and PAM status
Get-DatabaseAvailabilityGroup -Status | Format-Table Name, PrimaryActiveManager, *Servers*, DatacenterActivationMode

# Verify all components are Active
Get-ServerComponentState <ServerName> | Format-Table Component, State -AutoSize

# Check mailbox database copy status
Get-MailboxDatabaseCopyStatus * | Sort-Object Status, CopyQueueLength -Descending |
  Format-Table Name, Status, ContentIndexState, CopyQueueLength, ReplayQueueLength, LastInspectedLogTime, MailboxServer -AutoSize
```

**Important Note**: If you installed an Exchange update and some components remain in an `Inactive` state, run the following commands:

```powershell
Set-ServerComponentState <ServerName> -Component ServerWideOffline -State Active -Requester Functional
Set-ServerComponentState <ServerName> -Component Monitoring -State Active -Requester Functional
Set-ServerComponentState <ServerName> -Component RecoveryActionsEnabled -State Active -Requester Functional
```

---
