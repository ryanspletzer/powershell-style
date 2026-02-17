# PowerShell Error Handling

Philosophy and patterns for robust error handling.

## Error Types

### Terminating Errors

Stop execution and can be caught with try/catch:

- `throw` statements
- Cmdlets with `-ErrorAction Stop`
- .NET exceptions

### Non-Terminating Errors

Continue execution, write to error stream:

- Default cmdlet behavior
- `Write-Error` cmdlet

## Making Errors Terminating

### For Cmdlets

Use `-ErrorAction Stop`:

```powershell
try {
    Get-Item -Path '/nonexistent' -ErrorAction Stop
}
catch {
    Write-Warning "File not found: $($_.Exception.Message)"
}
```

### For Entire Scripts

Set preference variable:

```powershell
$ErrorActionPreference = 'Stop'
try {
    # All cmdlet errors now terminate
    Get-Item -Path '/nonexistent'
}
catch {
    # Handle error
}
finally {
    $ErrorActionPreference = 'Continue'  # Restore default
}
```

## Try-Catch-Finally Pattern

### Basic Structure

```powershell
try {
    # Code that might fail
    $result = Get-Content -Path $filePath -ErrorAction Stop
}
catch [System.IO.FileNotFoundException] {
    # Handle specific exception type
    Write-Warning "File not found: $filePath"
}
catch [System.UnauthorizedAccessException] {
    # Handle another specific type
    Write-Error "Access denied: $filePath"
}
catch {
    # Handle all other errors
    Write-Error "Unexpected error: $($_.Exception.Message)"
}
finally {
    # Always runs - use for cleanup
    if ($resource) {
        $resource.Dispose()
    }
}
```

### Capturing Error Details

**Important**: Capture `$_` immediately - it can be overwritten:

```powershell
catch {
    # Capture immediately
    $errorRecord = $_
    $errorMessage = $errorRecord.Exception.Message
    $errorType = $errorRecord.Exception.GetType().Name
    $errorLine = $errorRecord.InvocationInfo.ScriptLineNumber

    Write-Error "[$errorType] Line $errorLine : $errorMessage"
}
```

### Error Record Properties

| Property | Description |
|----------|-------------|
| `$_.Exception` | The .NET exception object |
| `$_.Exception.Message` | Error message text |
| `$_.Exception.InnerException` | Nested exception |
| `$_.InvocationInfo` | Where error occurred |
| `$_.TargetObject` | Object that caused error |
| `$_.FullyQualifiedErrorId` | Error identifier |
| `$_.CategoryInfo` | Error category |

## Anti-Patterns to Avoid

### Don't Use $? for Error Detection

```powershell
# Bad - $? only indicates completion, not success
Get-Item -Path '/file'
if (-not $?) {
    Write-Error 'Failed'  # May not trigger
}

# Good - use try/catch
try {
    Get-Item -Path '/file' -ErrorAction Stop
}
catch {
    Write-Error 'Failed'
}
```

### Don't Test Null as Error Detection

```powershell
# Bad - convoluted logic
$result = Get-SomeData
if ($null -eq $result) {
    # Was it an error or just no data?
}

# Good - explicit error handling
try {
    $result = Get-SomeData -ErrorAction Stop
    if ($result.Count -eq 0) {
        Write-Verbose 'No data found'
    }
}
catch {
    Write-Error 'Failed to retrieve data'
}
```

### Don't Suppress Errors Silently

```powershell
# Bad - errors disappear
Get-Item -Path '/file' -ErrorAction SilentlyContinue

# Better - capture for logging
$result = Get-Item -Path '/file' -ErrorAction SilentlyContinue -ErrorVariable getError
if ($getError) {
    Write-Verbose "Suppressed error: $($getError[0].Exception.Message)"
}
```

## Transaction Pattern

Handle entire operations as units:

```powershell
function Update-Configuration {
    [CmdletBinding()]
    param(
        [string]$Path,
        [hashtable]$Settings
    )

    # Put entire transaction in try block
    try {
        # Step 1: Read current config
        $config = Get-Content -Path $Path -Raw -ErrorAction Stop |
            ConvertFrom-Json

        # Step 2: Update settings
        foreach ($key in $Settings.Keys) {
            $config.$key = $Settings[$key]
        }

        # Step 3: Write back
        $config | ConvertTo-Json | Set-Content -Path $Path -ErrorAction Stop

        Write-Verbose 'Configuration updated successfully'
    }
    catch {
        Write-Error "Configuration update failed: $($_.Exception.Message)"
        # Entire transaction failed - no partial updates
    }
}
```

## Per-Operation Error Handling

Handle errors for discrete operations:

```powershell
function Test-Servers {
    [CmdletBinding()]
    param([string[]]$ComputerName)

    foreach ($computer in $ComputerName) {
        # Each server is handled independently
        try {
            $result = Test-Connection -ComputerName $computer -Count 1 -ErrorAction Stop

            [PSCustomObject]@{
                Computer = $computer
                Status = 'Online'
                ResponseTime = $result.ResponseTime
            }
        }
        catch {
            [PSCustomObject]@{
                Computer = $computer
                Status = 'Offline'
                Error = $_.Exception.Message
            }
        }
    }
}
```

## Throwing Errors

### For Terminating Errors

```powershell
# Simple message
throw 'Operation failed'

# With exception type
throw [System.InvalidOperationException]::new('Invalid state')

# Custom error record
$errorRecord = [System.Management.Automation.ErrorRecord]::new(
    [System.Exception]::new('Custom error'),
    'CustomErrorId',
    [System.Management.Automation.ErrorCategory]::InvalidOperation,
    $targetObject
)
throw $errorRecord
```

### For Non-Terminating Errors

```powershell
# Simple message
Write-Error 'Operation failed'

# With details
Write-Error -Message 'File not found' -Category ObjectNotFound -TargetObject $path

# Convert to terminating in catch block
catch {
    $PSCmdlet.ThrowTerminatingError($_)
}
```

## ErrorAction Values

| Value | Behavior |
|-------|----------|
| `Continue` | Write error, continue (default) |
| `Stop` | Terminate, can be caught |
| `SilentlyContinue` | Suppress, continue |
| `Ignore` | Suppress completely (no $Error) |
| `Inquire` | Prompt user |
| `Suspend` | Suspend workflow (workflows only) |

## Best Practices Summary

1. **Use try/catch** for operations that might fail
2. **Use `-ErrorAction Stop`** to make cmdlet errors catchable
3. **Capture `$_` immediately** in catch blocks
4. **Handle one operation per try block** for granular control
5. **Use finally** for cleanup that must always run
6. **Don't rely on `$?`** for error detection
7. **Log errors appropriately** - don't silently suppress
8. **Throw with context** - include meaningful messages

## Attribution

Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle).
Licensed under CC BY-SA 4.0.
