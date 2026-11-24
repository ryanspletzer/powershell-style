# PowerShell Output and Formatting

Guidance on output streams and data formatting.

## Output Streams

PowerShell has six output streams:

| # | Stream | Cmdlet | Preference Variable |
|---|--------|--------|---------------------|
| 1 | Output/Success | (implicit) | - |
| 2 | Error | `Write-Error` | `$ErrorActionPreference` |
| 3 | Warning | `Write-Warning` | `$WarningPreference` |
| 4 | Verbose | `Write-Verbose` | `$VerbosePreference` |
| 5 | Debug | `Write-Debug` | `$DebugPreference` |
| 6 | Information | `Write-Information` | `$InformationPreference` |

## When to Use Each Stream

### Output Stream (1)

Function results and pipeline data:

```powershell
function Get-DiskSpace {
    # Objects go to output stream automatically
    Get-CimInstance -ClassName Win32_LogicalDisk |
        ForEach-Object {
            [PSCustomObject]@{
                Drive = $_.DeviceID
                FreeBytes = $_.FreeSpace
            }
        }
}
```

### Error Stream (2)

Recoverable errors that shouldn't stop execution:

```powershell
function Get-ServiceInfo {
    param([string[]]$Name)

    foreach ($svc in $Name) {
        $service = Get-Service -Name $svc -ErrorAction SilentlyContinue
        if ($service) {
            $service
        } else {
            Write-Error -Message "Service not found: $svc" -Category ObjectNotFound
        }
    }
}
```

### Warning Stream (3)

Non-fatal issues the user should know about:

```powershell
function Update-Config {
    param([string]$Path)

    if (Test-Path $Path) {
        Write-Warning -Message "Config already exists - will be overwritten"
    }
    # Continue with update...
}
```

### Verbose Stream (4)

Detailed progress and decision information:

```powershell
function Sync-Data {
    [CmdletBinding()]  # Required for -Verbose support
    param([string]$Source, [string]$Destination)

    Write-Verbose -Message "Starting sync from $Source to $Destination"

    $files = Get-ChildItem -Path $Source
    Write-Verbose -Message "Found $($files.Count) files to process"

    foreach ($file in $files) {
        Write-Verbose -Message "Processing: $($file.Name)"
        # ...
    }

    Write-Verbose -Message 'Sync complete'
}
```

### Debug Stream (5)

Developer diagnostics and variable inspection:

```powershell
function Calculate-Value {
    [CmdletBinding()]  # Required for -Debug support
    param([int]$Input)

    Write-Debug -Message "Input value: $Input"

    $intermediate = $Input * 2
    Write-Debug -Message "After multiplication: $intermediate"

    $result = $intermediate + 10
    Write-Debug -Message "Final result: $result"

    $result
}
```

### Information Stream (6)

Structured status messages:

```powershell
function Install-Package {
    [CmdletBinding()]
    param([string]$Name)

    Write-Information -MessageData "Installing $Name..." -Tags 'Progress'
    # ...
    Write-Information -MessageData "Installation complete" -Tags 'Success'
}
```

## Write-Host: When to Use (and Not Use)

### Don't Use for Script Output

```powershell
# Bad - can't be captured, redirected, or suppressed
function Get-Status {
    Write-Host "Server is online"  # Goes directly to host
}

$result = Get-Status  # $result is empty!
```

### Acceptable Uses

Only for commands with `Show` or `Format` verbs:

```powershell
function Show-Banner {
    Write-Host '================================' -ForegroundColor Cyan
    Write-Host '   Welcome to My Application    ' -ForegroundColor White
    Write-Host '================================' -ForegroundColor Cyan
}

function Format-Report {
    param([switch]$Formatted)

    if ($Formatted) {
        Write-Host 'Report Summary' -ForegroundColor Green
        # Display-only formatting...
    }
}
```

## Object Output Best Practices

### Return One Type Per Function

```powershell
# Good - consistent output type
[OutputType([PSCustomObject])]
function Get-ServerStatus {
    [PSCustomObject]@{ Name = 'Server1'; Status = 'Online' }
    [PSCustomObject]@{ Name = 'Server2'; Status = 'Offline' }
}

# Bad - mixed types cause formatting issues
function Get-ServerStatus {
    'Server1 is online'  # String
    [PSCustomObject]@{ Name = 'Server2'; Status = 'Offline' }  # Object
}
```

### Output Raw Data

Let callers decide formatting:

```powershell
# Good - raw data, flexible
function Get-DiskInfo {
    [PSCustomObject]@{
        Drive = 'C:'
        SizeBytes = 500GB      # Raw bytes
        FreeBytes = 100GB
    }
}

# Caller formats as needed
Get-DiskInfo | ForEach-Object {
    [PSCustomObject]@{
        Drive = $_.Drive
        SizeGB = [math]::Round($_.SizeBytes / 1GB, 2)
        FreeGB = [math]::Round($_.FreeBytes / 1GB, 2)
    }
}

# Bad - pre-formatted, inflexible
function Get-DiskInfo {
    [PSCustomObject]@{
        Drive = 'C:'
        Size = '500 GB'        # Can't do math
        Free = '100 GB'
    }
}
```

### Use OutputType Attribute

```powershell
function Get-UserInfo {
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param([string]$UserName)

    [PSCustomObject]@{
        Name = $UserName
        Created = Get-Date
    }
}
```

## Pipeline Output

### Stream Objects Immediately

```powershell
# Good - streams as processed
function ConvertTo-CustomObject {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        [string]
        $Line
    )

    process {
        # Output immediately, don't collect
        [PSCustomObject]@{
            Line = $Line
            Length = $Line.Length
        }
    }
}
```

### Don't Buffer Unnecessarily

```powershell
# Bad - buffers everything in memory
function Get-LargeData {
    $results = @()
    foreach ($item in $hugeCollection) {
        $results += [PSCustomObject]@{ Data = $item }
    }
    $results
}

# Good - streams without buffering
function Get-LargeData {
    foreach ($item in $hugeCollection) {
        [PSCustomObject]@{ Data = $item }
    }
}
```

## Format Cmdlets

### Only at End of Pipeline

```powershell
# Good - format at display time
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 | Format-Table

# Bad - format breaks pipeline
Get-Process | Format-Table | Sort-Object CPU  # Won't work as expected
```

### Use Format Files for Custom Types

In module manifest, reference `.format.ps1xml`:

```powershell
# MyModule.psd1
@{
    FormatsToProcess = @('MyModule.format.ps1xml')
}
```

```xml
<!-- MyModule.format.ps1xml -->
<Configuration>
  <ViewDefinitions>
    <View>
      <Name>MyModule.ServerInfo</Name>
      <ViewSelectedBy>
        <TypeName>MyModule.ServerInfo</TypeName>
      </ViewSelectedBy>
      <TableControl>
        <TableHeaders>
          <TableColumnHeader><Label>Name</Label></TableColumnHeader>
          <TableColumnHeader><Label>Status</Label></TableColumnHeader>
        </TableHeaders>
        <TableRowEntries>
          <TableRowEntry>
            <TableColumnItems>
              <TableColumnItem><PropertyName>Name</PropertyName></TableColumnItem>
              <TableColumnItem><PropertyName>Status</PropertyName></TableColumnItem>
            </TableColumnItems>
          </TableRowEntry>
        </TableRowEntries>
      </TableControl>
    </View>
  </ViewDefinitions>
</Configuration>
```

## Progress Reporting

### Use Write-Progress for Long Operations

```powershell
function Copy-ManyFiles {
    [CmdletBinding()]
    param([string]$Source, [string]$Destination)

    $files = Get-ChildItem -Path $Source
    $total = $files.Count
    $current = 0

    foreach ($file in $files) {
        $current++
        $percent = ($current / $total) * 100

        Write-Progress -Activity 'Copying files' `
            -Status "$current of $total" `
            -CurrentOperation $file.Name `
            -PercentComplete $percent

        Copy-Item -Path $file.FullName -Destination $Destination
    }

    Write-Progress -Activity 'Copying files' -Completed
}
```

### Don't Put Critical Info Only in Progress

Progress disappears after completion - also log important info:

```powershell
Write-Progress -Activity 'Processing' -Status 'Server1'
Write-Verbose -Message 'Processing Server1'  # Persists in verbose output
```

## Summary

| Do | Don't |
|----|-------|
| Output objects to pipeline | Mix types in output |
| Use Write-Verbose for details | Use Write-Host for data |
| Return raw data | Pre-format output |
| Use OutputType attribute | Buffer large results |
| Stream in process block | Use Format cmdlets mid-pipeline |

## Attribution

Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle).
Licensed under CC BY-SA 4.0.
