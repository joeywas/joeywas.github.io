---
layout: post
title:  "Get-DbaPermission and the little gotchas"
date:   2021-10-20 22:00:00 -0700
categories: DBATools, PowerShell
---
DbaTools PowerShell module is the best thing since sliced bread when it comes to working with Microsoft SQL Server. When doing administrative work in SQL Server, Powershell with DbaTools is my go to resource. I have SQL Server Management Studio and Azure Data Studio installed, and they each have their place, but my daily driver is one (or more) PowerShell sessions with DbaTools.

That being said, we ran into a conundrum the other day with [Get-DbaPermission](https://docs.dbatools.io/Get-DbaPermission) output that was being used with [Write-DbaDbDataTable](https://docs.dbatools.io/Write-DbaDbTableData). 

It was taking much longer than expected to write data in the results to a SQL table.

{% highlight powershell %}
PS> $p = Get-DbaPermission -SqlInstance localhost -SqlCredential $creds
PS> $p | measure

Count             : 3527
Average           :
Sum               :
Maximum           :
Minimum           :
StandardDeviation :
Property          :

PS> $p | Get-Member

   TypeName: System.Data.DataRow

Name              MemberType            Definition
----              ----------            ----------
AcceptChanges     Method                void AcceptChanges()
BeginEdit         Method                void BeginEdit()
CancelEdit        Method                void CancelEdit()
ClearErrors       Method                void ClearErrors()
Delete            Method                void Delete()
EndEdit           Method                void EndEdit()
Equals            Method                bool Equals(System.Object obj)
GetChildRows      Method                System.Data.DataRow[] GetChildRows(string relationName), System.Data.DataRow[]…
GetColumnError    Method                string GetColumnError(int columnIndex), string GetColumnError(string columnNam…
GetColumnsInError Method                System.Data.DataColumn[] GetColumnsInError()
GetHashCode       Method                int GetHashCode()
GetParentRow      Method                System.Data.DataRow GetParentRow(string relationName), System.Data.DataRow Get…
GetParentRows     Method                System.Data.DataRow[] GetParentRows(string relationName), System.Data.DataRow[…
GetType           Method                type GetType()
HasVersion        Method                bool HasVersion(System.Data.DataRowVersion version)
IsNull            Method                bool IsNull(int columnIndex), bool IsNull(string columnName), bool IsNull(Syst…
RejectChanges     Method                void RejectChanges()
SetAdded          Method                void SetAdded()
SetColumnError    Method                void SetColumnError(int columnIndex, string error), void SetColumnError(string…
SetModified       Method                void SetModified()
SetParentRow      Method                void SetParentRow(System.Data.DataRow parentRow), void SetParentRow(System.Dat…
ToString          Method                string ToString()
Item              ParameterizedProperty System.Object Item(int columnIndex) {get;set;}, System.Object Item(string colu…
ComputerName      Property              System.Object ComputerName {get;set;}
Database          Property              string Database {get;set;}
Grantee           Property              string Grantee {get;set;}
GranteeType       Property              string GranteeType {get;set;}
GrantStatement    Property              string GrantStatement {get;set;}
InstanceName      Property              System.Object InstanceName {get;set;}
PermissionName    Property              string PermissionName {get;set;}
PermState         Property              string PermState {get;set;}
RevokeStatement   Property              string RevokeStatement {get;set;}
Securable         Property              string Securable {get;set;}
SecurableType     Property              string SecurableType {get;set;}
SqlInstance       Property              System.Object SqlInstance {get;set;}

PS> measure-command -Expression {$p | Write-DbaDataTable -SqlInstance localhost -SqlCredential $creds -Database tempdb -Table '#permtest2' -AutoCreateTable}

Seconds           : 34
Milliseconds      : 204
{% endhighlight %}

It takes 34 seconds to write 3,527 object properties. For larger objects it takes dramatically more time.  This is because the Write-DbaDbTableData makes a call to ConvertTo-DbaDataTable, which attempts to convert the object and *all* it's properties to a data table.

When inspecting $p with Get-Member, we don't see everything. Piping $p to a select displays everything that is going to be converted, which shows there are two Table and ItemArray properties. These properties contain hundreds of thousands of members, which bogs down ConvertTo-DbaDataTable

{% highlight powershell %}
PS> $p | select -first 1 *

ComputerName    : 92046c0318a2
InstanceName    : MSSQLSERVER
SqlInstance     : 92046c0318a2
Database        : master
PermState       : GRANT
PermissionName  : VIEW ANY COLUMN ENCRYPTION KEY DEFINITION
SecurableType   : DATABASE
Securable       : master
Grantee         : public
GranteeType     : DATABASE_ROLE
RevokeStatement :
GrantStatement  :
RowError        :
RowState        : Unchanged
Table           : {92046c0318a2, 92046c0318a2, 92046c0318a2, 92046c0318a2…}
ItemArray       : {92046c0318a2, MSSQLSERVER, 92046c0318a2, master…}
HasErrors       : False

{% endhighlight %}

To avoid having to convert the properties that contains hundreds of thousands of things, explicitly select the object properties to write to a table. 

{% highlight powershell %}
PS> measure-command -Expression {$p | select ComputerName,Database,Grantee,GranteeType,GrantStatement,InstanceName,PermissionsName,PermStat,RevokeStatement,Securable,SecurableType,SqlInstance | Write-DbaDataTable -SqlInstance localhost -SqlCredential $creds -Database tempdb -Table '#permtest3' -AutoCreateTable}

Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 1
Milliseconds      : 167
{% endhighlight %}

[DbaTools.io](http://dbatools.io)
