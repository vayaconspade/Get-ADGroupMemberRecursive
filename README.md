# Get-ADGroupMemberRecursive
Get AD group members recursively, tagged with root group DN and direct parent group DN.
Not completely polished, but most of the basics are here.

The code uses Microsoft's AD cmdlets and is compatible with PowerShell version 2; tested on Windows 7 SP1 with RSAT SP1.

With multiple groups, you need to use something like
```powershell
$GroupResults = $Groups | ForEach-Object { Get-ADGroupMemberRecursive $_ } | ...
```

Some interesting information came up after I posted this on Reddit, among other things about limits on the number of users introduced in ADWS - you can read it here:
https://www.reddit.com/r/PowerShell/comments/68dlg1/love_reinventing_wheels_get_ad_group_members/

From at least Windows Server 2008 R2, the Get-ADGroupMember has a -Recursive parameter that does everything my code does, except for the "parent group DN" tagging. It's said it suffers from the group member limit that I've now worked around (ostensibly).

You can actually _pipe_ multiple groups to the function (the -Identity parameter takes only a single group/string), but when doing that you cannot trust the "RootGroupDN" as it will always be set to the very last group you piped in, for all the entries. All other info is accurate, though, for all the objects, such as "DirectParentGroupDN". I'll see if I can work around this. It seems a bit odd how I'm having to do it.

```
PS C:\temp> Get-ADGroup 'TestGroupB' | Get-ADGroupMemberRecursive

DistinguishedName   : CN=testuser0300,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0300
SamAccountName      : testuser0300
DisplayName         : 
DirectParentGroupDN : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0301,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0301
SamAccountName      : testuser0301
DisplayName         : 
DirectParentGroupDN : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local


PS C:\temp> 'TestGroupB' | Get-ADGroupMemberRecursive


DistinguishedName   : CN=testuser0300,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0300
SamAccountName      : testuser0300
DisplayName         : 
DirectParentGroupDN : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0301,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0301
SamAccountName      : testuser0301
DisplayName         : 
DirectParentGroupDN : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
```

Screenshot example. Showing TestGroupA and that an infinite loop is avoided.

![alt tag](/Get-ADGroupMemberRecursive-screenshot-example.png)

Example with nested groups. Play with Select-Object and Sort-Object -Unique, etc.

```
Get-ADGroupMemberRecursive -Identity 'TestGroupA'


DistinguishedName   : CN=testuser0001,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0001
SamAccountName      : testuser0001
DisplayName         : 
DirectParentGroupDN : CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0002,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0002
SamAccountName      : testuser0002
DisplayName         : 
DirectParentGroupDN : CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0300,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0300
SamAccountName      : testuser0300
DisplayName         : 
DirectParentGroupDN : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0301,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0301
SamAccountName      : testuser0301
DisplayName         : 
DirectParentGroupDN : CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0100,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0100
SamAccountName      : testuser0100
DisplayName         : 
DirectParentGroupDN : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0101,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0101
SamAccountName      : testuser0101
DisplayName         : 
DirectParentGroupDN : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0200,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0200
SamAccountName      : testuser0200
DisplayName         : 
DirectParentGroupDN : CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local

DistinguishedName   : CN=testuser0201,OU=TestUsers,OU=TestOU,DC=whatever,DC=local
Name                : testuser0201
SamAccountName      : testuser0201
DisplayName         : 
DirectParentGroupDN : CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local
RootGroupDN         : CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
```

Verbose output for the same example as above.

```
PS C:\temp> Get-ADGroup 'TestGroupA' | Get-ADGroupMemberRecursive -Verbose
VERBOSE: [CN=testuser0100,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0100,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=testuser0101,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0101,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing group. Parent group: CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local.
VERBOSE: [CN=testuser0300,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0300,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=testuser0301,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0301,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing group. Parent group: CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local.
VERBOSE: [CN=testuser0200,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0200,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=testuser0201,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0201,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local] Already processed.
VERBOSE: [CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local] Already processed.
VERBOSE: [CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local] Processing group. Parent group: CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local.
VERBOSE: [CN=testuser0001,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0001,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local array.
VERBOSE: [CN=testuser0002,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Processing ...
VERBOSE: [CN=testuser0002,OU=TestUsers,OU=TestOU,DC=whatever,DC=local] Adding non-group element to CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local array.

..... output here unless you export/redirect/suppress or assign to a variable .....

VERBOSE: Exporting main data hash to $Global:STGroupHashTemp.
```

Example use.

```
PS C:\temp> $Result2 = Get-ADGroupMemberRecursive -Identity 'TestGroupA'

PS C:\temp> $Result2.Count
8

PS C:\temp> $Result2.Name 
testuser0001
testuser0002
testuser0300
testuser0301
testuser0100
testuser0101
testuser0200
testuser0201

PS C:\temp> $Result2.DirectParentGroupDN
CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local

PS C:\temp> $Result2.DirectParentGroupDN | Sort-Object -Unique
CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupB,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupC,OU=Groups,OU=TestOU,DC=whatever,DC=local
CN=TestGroupE,OU=Groups,OU=TestOU,DC=whatever,DC=local

PS C:\temp> $Result2[0].RootGroupDN
CN=TestGroupA,OU=Groups,OU=TestOU,DC=whatever,DC=local
```
