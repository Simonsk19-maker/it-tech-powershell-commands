# Microsoft 365 and Exchange Online — PowerShell Commands

## Connecting

```powershell
# Microsoft Graph (modern, covers users/mail/files/Teams/etc.)
Install-Module Microsoft.Graph -Scope CurrentUser
Connect-MgGraph -Scopes "User.Read.All", "Mail.ReadWrite"

# Exchange Online PowerShell (still needed for mailbox-level admin, migrations, transport rules)
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Connect-ExchangeOnline -UserPrincipalName admin@domain.onmicrosoft.com
```

## Users and licensing (Graph)

```powershell
# List all users
Get-MgUser -All | Select-Object DisplayName, UserPrincipalName

# Find one user
Get-MgUser -UserId "user@domain.com"

# See what licenses a user has
Get-MgUserLicenseDetail -UserId "user@domain.com"

# Assign a license
$license = @{ SkuId = "<sku-guid-from-Get-MgSubscribedSku>" }
Set-MgUserLicense -UserId "user@domain.com" -AddLicenses $license -RemoveLicenses @()

# See what SKUs the tenant has available and how many are used
Get-MgSubscribedSku | Select-Object SkuPartNumber, ConsumedUnits, PrepaidUnits
```

## Mailboxes (Exchange Online)

```powershell
# List all mailboxes
Get-Mailbox | Select-Object DisplayName, PrimarySmtpAddress, RecipientTypeDetails

# Mailbox size / item count
Get-MailboxStatistics -Identity "user@domain.com" | Select-Object DisplayName, TotalItemSize, ItemCount

# Grant someone Full Access to another mailbox (e.g. cover for absence)
Add-MailboxPermission -Identity "user@domain.com" -User "cover@domain.com" -AccessRights FullAccess

# Convert a user mailbox to a shared mailbox (e.g. when someone leaves)
Set-Mailbox -Identity "user@domain.com" -Type Shared

# Set an out-of-office / auto-reply
Set-MailboxAutoReplyConfiguration -Identity "user@domain.com" -AutoReplyState Enabled -InternalMessage "Out of office"
```

## Mailbox permissions audit (all mailboxes, all permission types)

Exchange Online has three separate, unrelated permission types worth
knowing about - a mailbox audit needs to check all three, not just one:

- **Full Access** — can open/read the whole mailbox as if it were their own
- **Send As** — can send mail that appears to come directly from that mailbox
- **Send on Behalf** — can send mail that shows as "X on behalf of Y"

```powershell
# List every mailbox (users, shared, rooms, equipment) with its type
Get-Mailbox -ResultSize Unlimited | Select-Object DisplayName, PrimarySmtpAddress, RecipientTypeDetails | Sort-Object RecipientTypeDetails

# List just shared mailboxes
Get-Mailbox -RecipientTypeDetails SharedMailbox -ResultSize Unlimited | Select-Object DisplayName, PrimarySmtpAddress

# Full Access permissions on ONE mailbox (excludes the built-in "NT AUTHORITY\SELF" entry every mailbox has)
Get-MailboxPermission -Identity "shared@domain.com" | Where-Object { $_.User -notlike "NT AUTHORITY*" -and $_.IsInherited -eq $false } | Select-Object User, AccessRights

# Send As permissions on ONE mailbox
Get-RecipientPermission -Identity "shared@domain.com" | Where-Object { $_.Trustee -notlike "NT AUTHORITY*" } | Select-Object Trustee, AccessRights

# Send on Behalf on ONE mailbox (this is a property of the mailbox itself, not a separate permission cmdlet)
(Get-Mailbox -Identity "shared@domain.com").GrantSendOnBehalfTo
```

### Full audit: every mailbox, every permission type, exported to CSV

```powershell
$results = foreach ($mbx in Get-Mailbox -ResultSize Unlimited) {

    $fullAccess = Get-MailboxPermission -Identity $mbx.Identity |
        Where-Object { $_.User -notlike "NT AUTHORITY*" -and $_.IsInherited -eq $false }

    $sendAs = Get-RecipientPermission -Identity $mbx.Identity |
        Where-Object { $_.Trustee -notlike "NT AUTHORITY*" }

    foreach ($perm in $fullAccess) {
        [PSCustomObject]@{
            Mailbox        = $mbx.PrimarySmtpAddress
            MailboxType    = $mbx.RecipientTypeDetails
            PermissionType = "Full Access"
            GrantedTo      = $perm.User
        }
    }
    foreach ($perm in $sendAs) {
        [PSCustomObject]@{
            Mailbox        = $mbx.PrimarySmtpAddress
            MailboxType    = $mbx.RecipientTypeDetails
            PermissionType = "Send As"
            GrantedTo      = $perm.Trustee
        }
    }
    foreach ($person in $mbx.GrantSendOnBehalfTo) {
        [PSCustomObject]@{
            Mailbox        = $mbx.PrimarySmtpAddress
            MailboxType    = $mbx.RecipientTypeDetails
            PermissionType = "Send on Behalf"
            GrantedTo      = $person
        }
    }
}

$results | Export-Csv "C:\MailboxPermissionsAudit.csv" -NoTypeInformation
```

Worth knowing before running this on a large tenant: it loops every
mailbox and makes 2 extra calls per mailbox, so it can take a while on
tenants with hundreds of mailboxes - fine for a small business, but not
something to run repeatedly on a big estate without expecting it to sit
for a few minutes.

## App-only (unattended script) auth against Graph

```powershell
# Client credentials flow - for scripts/automation, not interactive logins.
# Needs an Azure AD app registration with APPLICATION permissions (not
# Delegated) and admin consent granted.
$body = @{
    client_id     = "<app-id>"
    client_secret = "<app-secret>"
    scope         = "https://graph.microsoft.com/.default"
    grant_type    = "client_credentials"
}
$token = Invoke-RestMethod -Method Post -Uri "https://login.microsoftonline.com/<tenant-id>/oauth2/v2.0/token" -Body $body
$headers = @{ Authorization = "Bearer $($token.access_token)" }
Invoke-RestMethod -Uri "https://graph.microsoft.com/v1.0/users" -Headers $headers
```

## Useful diagnostic

```powershell
# Decode a JWT access token's "roles" claim - the reliable way to check
# whether an app registration's permission is actually Application type
# with consent granted, rather than trusting the Azure portal UI
$payload = $token.access_token.Split('.')[1].Replace('-','+').Replace('_','/')
switch ($payload.Length % 4) { 2 {$payload += '=='} 3 {$payload += '='} }
[System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($payload)) | ConvertFrom-Json | Select-Object roles
```
