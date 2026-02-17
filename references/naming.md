# PowerShell Naming Conventions

Detailed naming guidance from PoshCode/PowerShellPracticeAndStyle.

## Command Names

### Verb-Noun Pattern

All functions and cmdlets use `Verb-Noun` format:

```powershell
Get-Process      # Verb: Get, Noun: Process
Set-Location     # Verb: Set, Noun: Location
New-Item         # Verb: New, Noun: Item
```

### Approved Verbs

Use only approved verbs. Check with `Get-Verb`:

| Category | Verbs |
|----------|-------|
| Common | Add, Clear, Close, Copy, Enter, Exit, Find, Get, Hide, Join, Lock, Move, New, Open, Pop, Push, Remove, Rename, Reset, Search, Select, Set, Show, Skip, Split, Step, Switch, Undo, Unlock, Watch |
| Data | Backup, Checkpoint, Compare, Compress, Convert, ConvertFrom, ConvertTo, Dismount, Edit, Expand, Export, Import, Initialize, Limit, Merge, Mount, Out, Publish, Restore, Save, Sync, Unpublish, Update |
| Lifecycle | Approve, Assert, Build, Complete, Confirm, Deny, Deploy, Disable, Enable, Install, Invoke, Register, Request, Restart, Resume, Start, Stop, Submit, Suspend, Uninstall, Unregister, Wait |
| Diagnostic | Debug, Measure, Ping, Repair, Resolve, Test, Trace |
| Security | Block, Grant, Protect, Revoke, Unblock, Unprotect |

### Noun Guidelines

- Use **singular nouns** (not plural): `Get-Service` not `Get-Services`
- Use **PascalCase**: `Get-UserAccount` not `Get-useraccount`
- Be specific: `Get-ADUser` not `Get-User` (when in AD module)

## Parameter Names

### Standard Names

Match common cmdlet parameter names for consistency:

| Standard Name | Purpose | Don't Use |
|---------------|---------|-----------|
| `$ComputerName` | Target computer(s) | `$Server`, `$Machine`, `$Host` |
| `$Credential` | Authentication | `$Cred`, `$Auth` |
| `$Path` | File/folder path | `$FilePath`, `$Location` |
| `$LiteralPath` | Path without wildcards | `$ExactPath` |
| `$Name` | Resource identifier | `$ResourceName`, `$Id` |
| `$InputObject` | Pipeline input | `$Object`, `$Item` |
| `$Force` | Override protections | `$Overwrite` |
| `$PassThru` | Return modified object | `$Return` |

### Casing

- **PascalCase** for parameters: `$ComputerName`, `$ServiceName`
- **PascalCase** for public variables
- Optional **camelCase** for private/local variables: `$tempFile`, `$counter`

## Variable Names

### Public Variables

Use descriptive PascalCase names:

```powershell
$UserName = 'jsmith'
$ServerList = @('Server1', 'Server2')
$ConfigurationPath = '/etc/app/config.json'
```

### Private/Local Variables

Optionally use camelCase to distinguish from parameters:

```powershell
function Get-ProcessInfo {
    param([string]$Name)  # PascalCase parameter

    $matchingProcesses = Get-Process -Name $Name  # camelCase local
    $processCount = $matchingProcesses.Count

    # ...
}
```

### Avoid Abbreviations

```powershell
# Good
$ComputerName = 'Server01'
$SourcePath = '/data/input'
$MaxRetryCount = 3

# Bad
$cn = 'Server01'
$src = '/data/input'
$mrc = 3
```

## Script and Module Names

### Script Files

- Use Verb-Noun for runnable scripts: `Install-Application.ps1`
- Use descriptive names for library files: `StringUtilities.ps1`

### Modules

- Use company/project prefix for uniqueness: `Contoso.Security`
- Module name matches folder and .psm1 file: `MyModule/MyModule.psm1`

## Aliases

### In Scripts

**Never use aliases** in scripts or modules:

```powershell
# Good
Get-ChildItem -Path $directory | Where-Object { $_.Length -gt 1MB }

# Bad
gci $directory | ? { $_.Length -gt 1MB }
```

### In Interactive Use

Aliases are acceptable for interactive command-line use.

## Type Names

### Custom Types

Use namespace-qualified names:

```powershell
class MyCompany.Module.UserInfo {
    [string]$Name
    [datetime]$Created
}
```

### PSTypeName

For custom objects, use descriptive type names:

```powershell
[PSCustomObject]@{
    PSTypeName = 'MyCompany.DiskReport'
    Drive      = 'C:'
    FreeSpace  = 50GB
}
```

## Common Patterns

### Boolean Parameters

Use positive names (not negated):

```powershell
# Good
[switch]$Force
[switch]$Recurse
[switch]$PassThru

# Bad
[switch]$NoConfirm    # Use $Confirm with opposite default
[switch]$DontDelete   # Confusing double-negative
```

### Credential Parameters

Always use `[PSCredential]`:

```powershell
param(
    [Parameter()]
    [System.Management.Automation.PSCredential]
    [System.Management.Automation.Credential()]
    $Credential = [System.Management.Automation.PSCredential]::Empty
)
```

## Attribution

Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle).
Licensed under CC BY-SA 4.0.
