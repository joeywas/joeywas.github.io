---
layout: post
title:  "Exchange Resource Calendar Permissions"
date:   2016-09-06 09:00:00 -0700
categories: exchange powershell
---

Get all the room and equipment mailboxes, then get (or remove/add) Mailbox Folder permissions for the Calendar

{% highlight powershell %}
$resources = Get-Mailbox -ResultSize unlimited -Filter {(RecipientTypeDetails -eq 'RoomMailbox') -or (RecipientTypeDetails -eq 'EquipmentMailbox')} | select Name
foreach($resource in $resources){
   Get-MailboxFolderPermission "$($resource.Name):\Calendar" | select identity,user,accessrights
   #Add-MailboxFolderPermission -Identity "$($resource.Name):\Calendar" -User "Calendar Editors" -AccessRights "Editor"
   #Remove-MailboxFolderPermission -Identity "$($resource.Name):\Calendar" -User "All Employees"
   #Add-MailboxFolderPermission -Identity "$($resource.Name):\Calendar" -User "All Employees" -AccessRights "Reviewer"
}
{% endhighlight %}



[Spiceworks: Setting Access Permissions]: https://community.spiceworks.com/topic/329032-setting-access-permissions-on-room-mailbox-calendar-folders
[Office 365 Manage Room Mailboxes with PowerShell]:   http://o365info.com/room-mailbox-powershell-commands/
