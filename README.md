# AD-Hardling-Auto-Report

Get-ADUser -Filter * -Property * | Select-Object * | Export-Csv "C:\ADUser\Ultimate_AD_Report.csv" -NoTypeInformation -Encoding UTF8


(Get-ADComputer -Filter *).Count

gpresult /h C:\ADUser\GPO_Hardening_Report.html


# Output File
$path = "C:\ADUser\AD_Summary_Report.txt"
New-Item -ItemType Directory -Path "C:\ADUser" -Force

# Domain Info
$domain = Get-ADDomain
$domainName = $domain.DNSRoot

# GPO Count
$gpoCount = (Get-GPO -All).Count

# Delegated Permissions (approx via ACL)
$delegation = (Get-Acl "AD:$($domain.DistinguishedName)").Access.Count

# Computer Info
$computerName = $env:COMPUTERNAME

# Domain Joined Computers Count
$computerCount = (Get-ADComputer -Filter *).Count

# DNS Servers
$dns = (Get-DnsClientServerAddress -AddressFamily IPv4).ServerAddresses -join ", "

# Last Patch Update Date
$patchDate = (Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 1).InstalledOn

# OU Count
$ouCount = (Get-ADOrganizationalUnit -Filter *).Count

# Locked Accounts Count
$lockedCount = (Search-ADAccount -LockedOut).Count

# OS License Info (Approx Install Date)
$osInstallDate = (Get-CimInstance Win32_OperatingSystem).InstallDate

# Create Report
@"
================ AD SUMMARY REPORT ================

Domain Name                : $domainName
Computer Name              : $computerName
Policies (GPO) Count       : $gpoCount
Delegated Permissions      : $delegation
Domain Joined Computers    : $computerCount
DNS Servers                : $dns
Last Patch Update Date     : $patchDate
OU Count                   : $ouCount
Locked Accounts Count      : $lockedCount
OS Install Date            : $osInstallDate

==================================================
"@ | Out-File $path

Write-Host "✅ Report Generated at $path"
