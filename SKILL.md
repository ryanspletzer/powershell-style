---
name: powershell-style
description: |
  PowerShell best practices and style guide based on the community-driven
  PoshCode/PowerShellPracticeAndStyle. Provides naming conventions, function
  structure, error handling, output formatting, and code layout rules. Use
  when writing, reviewing, or refactoring PowerShell scripts (.ps1), modules
  (.psm1), manifests (.psd1), cmdlets, or advanced functions.
---

# PowerShell Style Guide

This skill provides guidelines for writing clean, maintainable PowerShell code.
Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle)
(CC BY-SA 4.0).

## Quick Reference

| Topic | Key Rule |
|-------|----------|
| Function names | `Verb-Noun` with approved verbs |
| Parameters | Full names, PascalCase, explicit types |
| Indentation | 4 spaces (not tabs) |
| Braces | Opening brace on same line (OTBS); `} else {`, `} catch {`, `} finally {` stay cuddled |
| Param blocks | Type attribute and variable each on their own line, blank line between parameters |
| Blank lines | One after a closing brace, unless `else`/`catch`/`finally` or another `}` follows |
| Line length | 115 characters max |
| Continuation | Never use backticks; splat, or break after a pipe or comma |
| Output | Objects via pipeline, not `Write-Host` |
| Documentation | Comment-based help inside every function |
| Performance | Measure with `Measure-Command`; prefer readability for small datasets |
| Security | Always use `[PSCredential]`, never plain-text passwords |

## Naming Conventions

### Commands

Use the **Verb-Noun** pattern with approved verbs:

```powershell
# Good
function Get-UserAccount { }
function Set-Configuration { }
function Test-NetworkConnection { }

# Bad
function GetUsers { }        # Missing hyphen
function User-Get { }        # Wrong order
function Retrieve-Data { }   # Unapproved verb (use Get)
```

Verify approved verbs with `Get-Verb`. Common verbs:

- **Get/Set** - Retrieve or modify state
- **New/Remove** - Create or delete resources
- **Start/Stop** - Control processes or services
- **Enable/Disable** - Toggle functionality
- **Test** - Verify conditions (returns Boolean)
- **Invoke** - Execute actions

### Parameters and Variables

- Use **PascalCase** for all public identifiers
- Use **full parameter names** in scripts (not aliases)
- Match common cmdlet parameter names

```powershell
# Good
function Get-ServerInfo {
    param(
        [string[]]
        $ComputerName,
        [PSCredential]
        $Credential,
        [int]
        $Port = 5985
    )
}

# Bad
param(
    [string[]]$servers,     # Should be ComputerName
    [PSCredential]$cred,    # Should be Credential
    [int]$p                 # Unclear abbreviation
)
```

### Path Handling

- Use **absolute paths** or `$PSScriptRoot` for script-relative paths
- Never use `~` (use `$env:USERPROFILE` or `$HOME`)
- Remember: .NET methods use `[Environment]::CurrentDirectory`, not PowerShell's location

```powershell
# Good
$configPath = Join-Path -Path $PSScriptRoot -ChildPath 'config.json'
$userPath = Join-Path -Path $env:USERPROFILE -ChildPath '.myapp'

# Bad
$configPath = '.\config.json'
$userPath = '~/myapp'
```

## Function Structure

### Always Use Advanced Functions

Every function should have `[CmdletBinding()]` and typed parameters:

```powershell
function Get-ServiceStatus {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [ValidateNotNullOrEmpty()]
        [string[]]
        $ServiceName,

        [Parameter()]
        [string]
        $ComputerName = $env:COMPUTERNAME
    )

    begin {
        # Initialization code
    }

    process {
        # Pipeline processing - output objects here
        foreach ($name in $ServiceName) {
            [PSCustomObject]@{
                Name = $name
                Computer = $ComputerName
                Status = (Get-Service -Name $name).Status
            }
        }
    }

    end {
        # Cleanup code
    }
}
```

### Key Function Rules

1. **Include `[CmdletBinding()]`** - Enables common parameters (-Verbose, -Debug, etc.)
2. **Declare `[OutputType()]`** - Documents return type for tooling
3. **Use `process {}` block** - Required when accepting pipeline input
4. **Output objects in `process {}`** - Not in `begin {}` or `end {}`
5. **No `return` keyword** - Just output the object directly

### Parameter Validation

Use validation attributes instead of manual checks:

```powershell
param(
    [ValidateNotNullOrEmpty()]
    [string]
    $Name,

    [ValidateRange(1, 65535)]
    [int]
    $Port,

    [ValidateSet('Development', 'Staging', 'Production')]
    [string]
    $Environment,

    [ValidatePattern('^[a-z0-9]+$')]
    [string]
    $Identifier,

    [ValidateScript({ Test-Path $_ })]
    [string]
    $Path
)
```

## Code Layout

### Indentation and Braces

Use 4 spaces and One True Brace Style (OTBS).
The opening brace ends the line,
and `else`, `elseif`, `catch`, and `finally` share a line with the preceding closing brace:

```powershell
function Test-Example {
    [CmdletBinding()]
    param(
        [string]
        $Name
    )

    if ($Name) {
        Write-Verbose "Processing: $Name"
    } else {
        Write-Warning 'No name provided'
    }
}
```

### Param Block Layout

Put each parameter's attributes, type, and variable name on their own lines,
and separate parameters with a blank line.
The type attribute and the variable are never combined on one line
(`[string]$Name`) in a multi-parameter `param` block:

```powershell
param (
    # The distribution group to add members to.
    [Parameter(Mandatory)]
    [ValidateNotNullOrEmpty()]
    [string]
    $Identity,

    # Skip the first N rows.
    [Parameter()]
    [ValidateRange(0, [int]::MaxValue)]
    [int]
    $Skip
)
```

A single, simple parameter may stay on one line: `param([string]$Path)`.

### Blank Lines

Follow a closing brace with a blank line
when another statement comes after it
(this mirrors StyleCop SA1513 in C#):

```powershell
if ($rowsRead -le $skipCount) {
    continue
}

if ($rowsRead -gt $lastRow) {
    break
}

[PSCustomObject]@{
    Member = $member
}

continue
```

Exceptions:

- No blank line before a chained keyword (`else`, `elseif`, `catch`, `finally`).
- No blank line before another closing brace.

Do not add blank lines between ordinary statements
just to precede a control statement;
use them sparingly to mark logical sections.

### Spacing

```powershell
# Around operators
$result = $a + $b
$isValid = $count -gt 0

# After commas
$array = @(1, 2, 3)
Get-Process -Name 'explorer', 'notepad'

# Inside braces and subexpressions
$value = $( Get-Date )
$scriptblock = { Write-Output 'Hello' }

# No space in variable delimiters
${env:PATH}
```

### Line Length and Continuation

Limit lines to 115 characters.
**Never use backticks for line continuation**, including in help examples.
A trailing space after a backtick silently breaks the command,
and the backtick is easy to miss and mistype.
Use splatting for long commands,
break after a pipe or comma,
or wrap an expression in parentheses:

```powershell
# Good - splatting
$params = @{
    Path        = $filePath
    Destination = $targetPath
    Force       = $true
    Recurse     = $true
}
Copy-Item @params

# Good - implied continuation after a pipe
$result = Get-ChildItem -Path $path |
    Where-Object { $_.Length -gt 1MB } |
    Sort-Object -Property Length -Descending

# Good - long string split inside parentheses
Write-Warning (
    "Column '$idColumn' not found in $file; " +
    'existing members cannot be prefiltered.'
)

# Bad - backtick continuation
Copy-Item -Path $filePath `
    -Destination $targetPath `
    -Force
```

## Error Handling

### Use Try-Catch with Terminating Errors

```powershell
function Get-RemoteData {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]
        $ComputerName
    )

    try {
        # Force terminating error with -ErrorAction Stop
        $session = New-PSSession -ComputerName $ComputerName -ErrorAction Stop

        Invoke-Command -Session $session -ScriptBlock {
            Get-Process
        }
    } catch {
        # Capture error immediately - $_ can be overwritten
        $errorRecord = $_
        Write-Error "Failed to connect to $ComputerName : $($errorRecord.Exception.Message)"
    } finally {
        if ($session) {
            Remove-PSSession -Session $session
        }
    }
}
```

### Error Handling Rules

1. **Use `-ErrorAction Stop`** to make cmdlet errors terminating
2. **Capture `$_` immediately** in catch blocks
3. **Avoid `$?`** - It only indicates completion, not success
4. **Handle one operation per try block** - Query one server, delete one file
5. **Don't test for null as error detection** - Use exceptions

## Output and Formatting

### Output Streams

| Stream | Cmdlet | Use Case |
|--------|--------|----------|
| Output | (implicit) | Function results, pipeline data |
| Verbose | `Write-Verbose` | Detailed progress, decisions |
| Warning | `Write-Warning` | Non-fatal issues |
| Error | `Write-Error` | Recoverable errors |
| Debug | `Write-Debug` | Developer diagnostics |
| Information | `Write-Information` | Structured status messages |

```powershell
function Process-Items {
    [CmdletBinding()]
    param([string[]]$Items)

    foreach ($item in $Items) {
        Write-Verbose "Processing: $item"

        if (Test-ShouldSkip -Item $item) {
            Write-Warning "Skipping $item - already processed"
            continue
        }

        # Output goes to pipeline
        [PSCustomObject]@{
            Item = $item
            ProcessedAt = Get-Date
        }
    }
}
```

### Output Rules

1. **Never use `Write-Host`** for script output (only for Show/Format verbs)
2. **Return one object type** per function
3. **Output raw data** - Let callers format as needed
4. **Use `[OutputType()]`** to document return types
5. **Don't mix strings with objects** in output

## Documentation and Comments

Place comment-based help **inside** the function at the top.
Every function needs at minimum `.SYNOPSIS` and `.EXAMPLE`.
Document parameters directly above each parameter in the `param` block:

```powershell
function Get-ServiceStatus {
    <#
        .SYNOPSIS
            Gets the status of specified services.

        .EXAMPLE
            Get-ServiceStatus -ServiceName 'WinRM'
    #>
    [CmdletBinding()]
    param (
        # The name of the service to query.
        [Parameter(Mandatory)]
        [string]
        $ServiceName
    )

    Get-Service -Name $ServiceName
}
```

Comments should explain **why**, not **what** —
well-written PowerShell is self-explanatory.

## Performance

Measure with `Measure-Command` before optimizing.
For small datasets, prefer readability over speed.

General performance hierarchy (fastest to slowest):

1. Language features (`foreach`, `if`, operators)
2. Compiled .NET methods
3. Inline PowerShell script
4. Cmdlet/function calls (invocation overhead)

For large files, stream via the pipeline or .NET `StreamReader`
instead of loading everything into memory with `Get-Content`.

## Security

Always use `[PSCredential]` for credentials — never plain strings:

```powershell
param (
    [System.Management.Automation.PSCredential]
    [System.Management.Automation.Credential()]
    $Credential
)
```

Accept credentials as parameters (don't call `Get-Credential` internally).
Decrypt passwords inline during API calls — don't store cleartext in variables.
Use `Export-CliXml` to persist credentials to disk (DPAPI-protected).

## Module Design

### Separation of Concerns

- **Tools**: Reusable functions in modules - accept parameters, output raw data
- **Controllers**: Scripts that orchestrate tools - may format output

```powershell
# Tool (in module) - raw data output
function Get-DiskInfo {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param([string]$ComputerName)

    Get-CimInstance -ClassName Win32_LogicalDisk -ComputerName $ComputerName |
        ForEach-Object {
            [PSCustomObject]@{
                Drive = $_.DeviceID
                SizeBytes = $_.Size          # Raw bytes, not formatted
                FreeBytes = $_.FreeSpace
            }
        }
}

# Controller (script) - formats for specific use
$report = Get-DiskInfo -ComputerName 'Server01' |
    Select-Object Drive,
        @{N='SizeGB'; E={[math]::Round($_.SizeBytes/1GB, 2)}},
        @{N='FreeGB'; E={[math]::Round($_.FreeBytes/1GB, 2)}}
$report | Format-Table
```

## References

For detailed guidance, see:

- [references/naming.md](references/naming.md) - Complete naming conventions
- [references/functions.md](references/functions.md) - Advanced function patterns
- [references/error-handling.md](references/error-handling.md) - Error handling philosophy
- [references/output.md](references/output.md) - Output stream guidance
- [references/documentation.md](references/documentation.md) - Comment-based help and commenting rules
- [references/performance.md](references/performance.md) - Performance measurement and trade-offs
- [references/security.md](references/security.md) - Credentials, SecureString, and disk persistence

## Attribution

Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle)
by Don Jones, Matt Penny, Carlos Perez, Joel Bennett, and the PowerShell community.
Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
