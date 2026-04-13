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
