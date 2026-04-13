# Search and Delete Messages from Exchange User Mailboxes

This guide explains how to search for and permanently delete specific emails from one or multiple user mailboxes in Microsoft Exchange Server.

Common use cases include removing phishing emails, spam, or unwanted messages with a specific subject or sender.

> **⚠️ Important Warning**  
> Using the `-DeleteContent` switch **permanently deletes** the messages. They will **not** be moved to the Deleted Items folder and can only be recovered from a backup.

---

## Table of Contents

- [Prerequisites and Permissions](#prerequisites-and-permissions)
- [Export All User Mailboxes to CSV](#export-all-user-mailboxes-to-csv)
- [Search and Delete Messages](#search-and-delete-messages)
  - [Delete from a Single Mailbox](#delete-from-a-single-mailbox)
  - [Delete from Multiple Mailboxes using CSV](#delete-from-multiple-mailboxes-using-csv)
  - [Delete from All Mailboxes](#delete-from-all-mailboxes)
- [Search Only (Log Mode)](#search-only-log-mode)
- [Verify Results](#verify-results)
- [Limitations and Recommendations](#limitations-and-recommendations)

---

## Prerequisites and Permissions

To use the `-DeleteContent` parameter, your account must be a member of the **Discovery Management** role group and have the **Mailbox Import Export** role.

```powershell
# Add user to Discovery Management group
Add-ADGroupMember "Discovery Management" -Members "YourAdminUsername"

# Assign Mailbox Import Export role
New-ManagementRoleAssignment -Role "Mailbox Import Export" -User "YourAdminUsername"
```
> **Note**: After assigning roles, close and reopen the Exchange Management Shell.
---
## Export All User Mailboxes to CSV
To perform bulk search and delete operations, it is recommended to first export a list of all user mailboxes to a CSV file.
### Step 1: Create the temp folder
Create a folder named `temp` on the C: drive (if it doesn't already exist):
```powershell
C:\temp
```

### Step 2: Export all mailboxes
Run the following command in Exchange Management Shell (run as Administrator):
```powershell
Get-Mailbox -ResultSize Unlimited |
  Select-Object DisplayName, SamAccountName, PrimarySmtpAddress |
  Sort-Object PrimarySmtpAddress |
  Export-Csv "C:\temp\user_mailboxes.csv" -NoTypeInformation -Encoding UTF8
```

### Step 3: Verify and edit the CSV file
Open the file C:\temp\user_mailboxes.csv with Microsoft Excel.
Make sure the column PrimarySmtpAddress exists and contains the email addresses.
You can edit the CSV file and remove any users you do not want to include in the search/delete operation.

> **Tip**: Keeping a clean and up-to-date CSV file makes bulk operations much safer and easier to manage.
---
## Search and Delete Messages
### Delete from a Single Mailbox
```PowerShell
# Delete by Subject
Search-Mailbox -Identity "User Mailbox" `
  -SearchQuery 'Subject:"Type Your Subject"' `
  -DeleteContent -Force

# Delete by Sender and Date
Search-Mailbox -Identity "UserMailbox@YourDomain.com" `
  -SearchQuery 'From:"xxxx@domain.com" AND Sent:09/07/2025' `
  -DeleteContent -Force
```
### Delete from Multiple Mailboxes using CSV
```PowerShell
Import-Csv "C:\temp\user_mailboxes.csv" |
  ForEach-Object {
    Search-Mailbox $_.PrimarySmtpAddress `
      -SearchQuery 'Subject:"Saying goodbye is never easy"' `
      -DeleteContent -Force
  }
```
### Delete from All Mailboxes
```PowerShell
Get-Mailbox -ResultSize Unlimited |
  Search-Mailbox -SearchQuery 'Subject:"aryasasol.com Server Upgrade !"' `
    -DeleteContent -Force
```
---
