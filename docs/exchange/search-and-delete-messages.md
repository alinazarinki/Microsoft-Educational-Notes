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
Note: After assigning roles, close and reopen the Exchange Management Shell.

Export All User Mailboxes to CSV
Export a list of all mailboxes to a CSV file for bulk operations:
PowerShell# First create the C:\temp folder
Get-Mailbox -ResultSize Unlimited |
  Select-Object DisplayName, SamAccountName, PrimarySmtpAddress |
  Sort-Object PrimarySmtpAddress |
  Export-Csv "C:\temp\user_mailboxes.csv" -NoTypeInformation -Encoding UTF8
You can open the CSV file in Excel and remove any users you do not want to include.

Search and Delete Messages
Delete from a Single Mailbox
PowerShell# Delete by Subject
Search-Mailbox -Identity "cheraghia" `
  -SearchQuery 'Subject:"ثبت نام تولید کنندگان داخلی جدید"' `
  -DeleteContent -Force

# Delete by Sender and Date
Search-Mailbox -Identity "abdolrasool.galegirizadeh@aryasasol.com" `
  -SearchQuery 'From:"a.nowzari@bina-epc.com" AND Sent:09/07/2025' `
  -DeleteContent -Force
Delete from Multiple Mailboxes using CSV
PowerShellImport-Csv "C:\temp\user_mailboxes.csv" |
  ForEach-Object {
    Search-Mailbox $_.PrimarySmtpAddress `
      -SearchQuery 'Subject:"Saying goodbye is never easy"' `
      -DeleteContent -Force
  }
Delete from All Mailboxes
PowerShellGet-Mailbox -ResultSize Unlimited |
  Search-Mailbox -SearchQuery 'Subject:"aryasasol.com Server Upgrade !"' `
    -DeleteContent -Force

Search Only (Log Mode)
It is highly recommended to run a search in log mode before deleting:
PowerShellGet-Mailbox -ResultSize Unlimited |
  Search-Mailbox -SearchQuery 'From:"a.nowzari@bina-epc.com" AND Sent:09/07/2025' `
    -LogOnly -LogLevel Full

Verify Results
After deletion, you can verify using the following command:
PowerShellSearch-Mailbox -Identity "testuser@domain.com" `
  -SearchQuery 'Subject:"Saying goodbye is never easy"' `
  -EstimateResultOnly
Note: Deleted messages will not appear in the Deleted Items folder.

Limitations and Recommendations

The Search-Mailbox cmdlet is limited to 10,000 results per mailbox. For larger environments or more than 10,000 items, use the modern New-ComplianceSearch and New-ComplianceSearchAction cmdlets instead.
Always test your search query without the -DeleteContent switch first.
Use the -Force switch to avoid confirmation prompts for every mailbox.

Reference Article:
Search and Delete Messages from Exchange User Mailboxes
