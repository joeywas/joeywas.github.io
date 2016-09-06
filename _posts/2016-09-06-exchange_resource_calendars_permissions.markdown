---
layout: post
title:  "Exchange Resource Calendar Permissions"
date:   2016-09-06 09:00:00 -0700
categories: exchange powershell
---

Recently we had an issue in my organization with all employees having more permissions on resource calendars than they should.   In our case, all employees were able to add/edit/delete entries on the resource calendars.   Our intent was to only allow specific employees this access, with all others required to add the resource to an appointment on their own calendar, in order to force them to go through the calendaring processor on the Exchange server.

The following PowerShell snippet will get all the room and equipment mailboxes, then get (or remove/add) Mailbox Folder permissions for the Calendar associated with that resource.    

{% highlight powershell %}
$resources = Get-Mailbox -ResultSize unlimited -Filter {(RecipientTypeDetails -eq 'RoomMailbox') -or (RecipientTypeDetails -eq 'EquipmentMailbox')} | select Name
foreach($resource in $resources){
   Get-MailboxFolderPermission "$($resource.Name):\Calendar" | select identity,user,accessrights
   #Add-MailboxFolderPermission -Identity "$($resource.Name):\Calendar" -User "Calendar Editors" -AccessRights "Editor"
   #Remove-MailboxFolderPermission -Identity "$($resource.Name):\Calendar" -User "All Employees"
   #Add-MailboxFolderPermission -Identity "$($resource.Name):\Calendar" -User "All Employees" -AccessRights "Reviewer"
}
{% endhighlight %}

[Spiceworks: Setting Access Permissions](https://community.spiceworks.com/topic/329032-setting-access-permissions-on-room-mailbox-calendar-folders)
[Office 365 Manage Room Mailboxes with PowerShell](http://o365info.com/room-mailbox-powershell-commands/)
