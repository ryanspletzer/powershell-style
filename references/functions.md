# PowerShell Function Structure

Detailed guidance on writing advanced functions.

## Basic Structure

Every function should follow this template:

```powershell
function Verb-Noun {
    [CmdletBinding()]
    [OutputType([TypeName])]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [ValidateNotNullOrEmpty()]
        [string[]]
        $ParameterName
    )

    begin {
        # One-time initialization
    }

    process {
        # Runs for each pipeline item - output objects here
    }

    end {
        # Cleanup after all items processed
    }
}
```

## CmdletBinding

Always include `[CmdletBinding()]`:

```powershell
[CmdletBinding(
    SupportsShouldProcess,           # Enable -WhatIf and -Confirm
    ConfirmImpact = 'Medium',        # Auto-confirm threshold
    DefaultParameterSetName = 'ByName'
)]
```

### Common Parameters Enabled

`[CmdletBinding()]` automatically adds:

- `-Verbose` / `-Debug`
- `-ErrorAction` / `-ErrorVariable`
- `-WarningAction` / `-WarningVariable`
- `-InformationAction` / `-InformationVariable`
- `-OutVariable` / `-OutBuffer` / `-PipelineVariable`

### SupportsShouldProcess

For functions that modify state:

```powershell
function Remove-TempFile {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
    param([string]$Path)

    process {
        if ($PSCmdlet.ShouldProcess($Path, 'Delete file')) {
            Remove-Item -Path $Path -Force
        }
    }
}
```

## OutputType

Declare return types for tooling support:

```powershell
# Single type
[OutputType([System.IO.FileInfo])]

# Multiple types
[OutputType([System.IO.FileInfo], [System.IO.DirectoryInfo])]

# Per parameter set
[OutputType([PSCustomObject], ParameterSetName = 'Default')]
[OutputType([string], ParameterSetName = 'AsString')]
```

## Parameter Declaration

### Parameter Attributes

```powershell
param(
    [Parameter(
        Mandatory,                          # Required parameter
        Position = 0,                       # Positional binding
        ValueFromPipeline,                  # Accept pipeline input
        ValueFromPipelineByPropertyName,    # Bind by property name
        HelpMessage = 'Computer name(s)'    # Help text
    )]
    [Alias('CN', 'Server')]                 # Alternative names
    [string[]]
    $ComputerName,

    [Parameter(ParameterSetName = 'ByPath')]
    [string]
    $Path,

    [Parameter(ParameterSetName = 'ByLiteralPath')]
    [string]
    $LiteralPath
)
```

### Validation Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `[ValidateNotNull()]` | Reject null | Required objects |
| `[ValidateNotNullOrEmpty()]` | Reject null/empty | Required strings |
| `[ValidateSet()]` | Limit to values | `[ValidateSet('A','B','C')]` |
| `[ValidateRange()]` | Numeric range | `[ValidateRange(1,100)]` |
| `[ValidateLength()]` | String length | `[ValidateLength(1,50)]` |
| `[ValidateCount()]` | Array count | `[ValidateCount(1,10)]` |
| `[ValidatePattern()]` | Regex match | `[ValidatePattern('^[a-z]+$')]` |
| `[ValidateScript()]` | Custom logic | `[ValidateScript({Test-Path $_})]` |

### Allow Attributes

Override mandatory validation:

```powershell
param(
    [Parameter(Mandatory)]
    [AllowNull()]                    # Permit $null
    [string]
    $OptionalName,

    [Parameter(Mandatory)]
    [AllowEmptyString()]             # Permit ""
    [string]
    $Description,

    [Parameter(Mandatory)]
    [AllowEmptyCollection()]         # Permit @()
    [string[]]
    $Tags
)
```

## Begin/Process/End Blocks

### When to Use Each

```powershell
function Get-FormattedData {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        [string[]]
        $InputData
    )

    begin {
        # Runs once before pipeline processing
        # Use for: Connection setup, resource initialization
        Write-Verbose -Message 'Initializing...'
        $results = [System.Collections.Generic.List[object]]::new()
    }

    process {
        # Runs for each pipeline item
        # Use for: Processing input, outputting objects
        foreach ($item in $InputData) {
            # Output objects here for streaming
            [PSCustomObject]@{
                Original = $item
                Upper = $item.ToUpper()
            }
        }
    }

    end {
        # Runs once after all pipeline items
        # Use for: Cleanup, final summary output
        Write-Verbose -Message 'Processing complete'
    }
}
```

### Pipeline Processing

For pipeline input, always use `process {}`:

```powershell
# Good - streams output
function ConvertTo-Upper {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        [string]
        $Text
    )

    process {
        $Text.ToUpper()  # Outputs immediately
    }
}

# Bad - buffers everything
function ConvertTo-Upper {
    param([string[]]$Text)

    $Text | ForEach-Object { $_.ToUpper() }  # Waits for all input
}
```

## Output Patterns

### Direct Output (Recommended)

```powershell
process {
    # Object is immediately sent to pipeline
    [PSCustomObject]@{
        Name = $item
        Status = 'Processed'
    }
}
```

### Avoid Return Keyword

```powershell
# Preferred
process {
    $result
}

# Avoid (works but unnecessary)
process {
    return $result
}
```

### Multiple Outputs

```powershell
process {
    foreach ($computer in $ComputerName) {
        # Each object streams immediately
        [PSCustomObject]@{
            Computer = $computer
            Status = Test-Connection $computer -Count 1 -Quiet
        }
    }
}
```

## Parameter Sets

Define mutually exclusive parameter groups:

```powershell
function Get-Resource {
    [CmdletBinding(DefaultParameterSetName = 'ByName')]
    param(
        [Parameter(ParameterSetName = 'ByName', Mandatory)]
        [string]
        $Name,

        [Parameter(ParameterSetName = 'ById', Mandatory)]
        [int]
        $Id,

        [Parameter(ParameterSetName = 'All')]
        [switch]
        $All
    )

    switch ($PSCmdlet.ParameterSetName) {
        'ByName' { Get-ResourceByName -Name $Name }
        'ById'   { Get-ResourceById -Id $Id }
        'All'    { Get-AllResources }
    }
}
```

## Comment-Based Help

Include help for discoverability:

```powershell
function Get-UserInfo {
<#
.SYNOPSIS
    Retrieves user information from Active Directory.

.DESCRIPTION
    The Get-UserInfo function queries Active Directory for user account
    information including name, email, and group memberships.

.PARAMETER UserName
    The SAM account name of the user to retrieve.

.PARAMETER IncludeGroups
    When specified, includes group membership information.

.EXAMPLE
    Get-UserInfo -UserName 'jsmith'
    Retrieves information for user jsmith.

.EXAMPLE
    'jsmith', 'ajones' | Get-UserInfo -IncludeGroups
    Retrieves information for multiple users with group membership.

.INPUTS
    System.String
    You can pipe user names to this function.

.OUTPUTS
    PSCustomObject
    Returns a custom object with user properties.

.NOTES
    Requires ActiveDirectory module.

.LINK
    https://docs.company.com/Get-UserInfo
#>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory, ValueFromPipeline)]
        [string]
        $UserName,

        [switch]
        $IncludeGroups
    )

    # Implementation...
}
```

## Attribution

Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle).
Licensed under CC BY-SA 4.0.
