---
layout: post
title: "UserViews missing from Project Server SE content database"
date: 2026-04-14 20:00:00 -07:00
categories: projectserver
tags:
- microsoft
- projectserver
- sql
- powershell
---

# Problem: UserViews missing from Project Server SE database

UserViews in the `pjrep` shema of a Project Server SE content database are missing. Running `Repair-SPProjectWebInstance` with `-RepairRule 7` does not work.

## Cause: More than one site in content database

The content database contains more than one site. This can happen if a site was soft deleted. Database stored procedure `pjrep.MSP_Epm_GenerateAllMultiValueAssociationViews` will skip creating user views if there is more than one site in the content database *even if the site was soft deleted*.

## Solution: Force Delete site from content database

- Identify Site ID that was soft deleted by executing this SQL against the content database

```sql
SELECT SiteID, WADMIN_DEFAULT_SITE_COLLECTION, WADMIN_IS_DELETED
FROM pjpub.MSP_WEB_ADMIN
WHERE WADMIN_IS_DELETED = 1
```
   
- Force delete site using SharePoint PowerShell

```powershell
# Reference to good PWA site
$GoodSite = Get-SPSite 'https://domainname.for.pwa.site/sites/sitename'
# get reference to the content database
$siteDatabase = $GoodSite.ContentDatabase
# Use the SiteID value from SQL in previous step
$SiteIDToDelete = 'SiteID from sql'
# force delete it
$siteDatabase.ForceDeleteSite($SiteIdToDelete, $false, $false)
```

- Execute a Repair using Sharepoint PowerShell

```powershell
Repair-SPProjectWebInstance -Identity 'https://domainname.for.pwa.site/sites/sitename' -RepairRule 7
```

- UserViews in the `pjrep` schema of the content database should now show up

## References

- [Deleting a corrupted site collection](https://sharepoint.stackexchange.com/questions/166321/deleting-a-corrupted-site-collection)
- [Missing User Views from PWA database](https://advaiya.com/how-to-get-the-missing-project-server-2016-user-views-from-pwa-database/)