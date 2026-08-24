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
