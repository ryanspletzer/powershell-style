# Security

Based on
[PoshCode/PowerShellPracticeAndStyle - Security](https://github.com/PoshCode/PowerShellPracticeAndStyle/blob/master/Best-Practices/Security.md)
(CC BY-SA 4.0).

## Always Use PSCredential for Credentials

Never store passwords in plain strings.
Use `[PSCredential]` objects, which store passwords as `SecureString`:

```powershell
param (
    [System.Management.Automation.PSCredential]
    [System.Management.Automation.Credential()]
    $Credential
)
```

Accept `PSCredential` as a parameter rather than calling `Get-Credential`
inside the function — this lets callers reuse stored credentials.

The `[Credential()]` attribute prompts for a password via a secure dialog
when a user passes only a username string.

### Passing Passwords to .NET APIs

Decrypt the credential during the call.
Do not store the cleartext in a variable:

```powershell
# Good — decrypt inline
$Insecure.SetPassword($Credential.GetNetworkCredential().Password)

# Bad — cleartext sits in memory
$password = $Credential.GetNetworkCredential().Password
$Insecure.SetPassword($password)
```

## Working with SecureString

For sensitive strings beyond credentials, use `[SecureString]`:

```powershell
# Prompt securely
$Secure = Read-Host -Prompt "Enter the API key" -AsSecureString
```

### Decrypting SecureString

When you must convert to plaintext, clean up immediately:

```powershell
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($Secure)
$plaintext = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
[System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR)
# Use $plaintext, then clear it
```

## Persisting Credentials to Disk

### Export-CliXml (Recommended)

Serializes credentials with DPAPI protection —
only the same user on the same machine can decrypt:

```powershell
# Save
Get-Credential | Export-CliXml -Path C:\creds\credential.xml

# Restore
$Credential = Import-CliXml -Path C:\creds\credential.xml
```

### ConvertFrom-SecureString

Encrypts a `SecureString` to a standard string for file storage.
Also uses DPAPI by default (same user, same machine):

```powershell
# Encrypt and save
$Secure = Read-Host -Prompt "Enter secret" -AsSecureString
ConvertFrom-SecureString -SecureString $Secure |
    Out-File -Path "${Env:AppData}\Sec.bin"

# Restore
$Secure = Get-Content -Path "${Env:AppData}\Sec.bin" |
    ConvertTo-SecureString
```

For shared secrets across machines,
use AES encryption with a shared key
(the `-Key` parameter on `ConvertFrom-SecureString`
and `ConvertTo-SecureString`).
