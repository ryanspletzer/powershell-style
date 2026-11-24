# Documentation and Comments

Based on
[PoshCode/PowerShellPracticeAndStyle - Documentation and Comments](https://github.com/PoshCode/PowerShellPracticeAndStyle/blob/master/Style-Guide/Documentation-and-Comments.md)
(CC BY-SA 4.0).

## General Commenting Rules

Comments that contradict the code are worse than no comments.
Always keep comments in sync with the code.

Comments should explain **why**, not **what**.
Well-written PowerShell is mostly self-explanatory:

```powershell
# Bad - describes what the code does
# Increment Margin by 2
$Margin = $Margin + 2

# Good - explains the reasoning
# The rendering box obscures a couple of pixels.
$Margin = $Margin + 2
```

## Block Comments

Use block comments for sections of code, indented to the same level.
Each line starts with `#` and a single space.

For long documentation blocks, use the `<# ... #>` syntax
with comment delimiters on their own lines:

```powershell
# Short block comments use single-line syntax.
# Each line starts with a hash and a space.

<#
    .SYNOPSIS
        Long comment blocks use multi-line syntax.
    .DESCRIPTION
        Particularly useful for function help that is
        frequently edited.
#>
```

## Inline Comments

Inline comments sit on the same line as code.
Separate them from the statement by at least two spaces
and align them with other inline comments in the same block:

```powershell
$Options = @{
    Margin   = 2    # The rendering box obscures a couple of pixels.
    Padding  = 2    # We need space between the border and the text.
    FontSize = 24   # Keep this above 16 so it's readable in presentations.
}
```

## Comment-Based Help

Every function should have comment-based help.
Write in simple language — be complete but concise.

### Location

Place help **inside** the function at the top,
so it stays with the code and is harder to forget:

```powershell
function Test-Help {
    <#
        .SYNOPSIS
            An example function to display how help should be written.

        .EXAMPLE
            Get-Help -Name Test-Help

            This shows the help for the example function.
    #>
    [CmdletBinding()]
    param (
        # This parameter doesn't do anything.
        # Aliases: MP
        [Parameter(Mandatory = $true)]
        [Alias("MP")]
        [String]
        $MandatoryParameter
    )

    <# code here ... #>
}
```

### Required Sections

At minimum, every function needs:

- **`.SYNOPSIS`** — a brief description (not just restating the function name)
- **`.EXAMPLE`** — at least one usage example per major use case

### Parameter Documentation

Document parameters directly above each parameter in the `param` block
(not in a separate `.PARAMETER` section).
This keeps docs close to the code they describe:

```powershell
param (
    # The computer name to query for service status.
    [Parameter(Mandatory = $true,
               ValueFromPipelineByPropertyName = $true,
               Position = 0)]
    [string]
    $ComputerName,

    # Maximum number of results to return.
    [int]
    $MaxResults = 100
)
```

### Full Template

```powershell
function Get-Example {
    <#
        .SYNOPSIS
            A brief description of the function or script.

        .DESCRIPTION
            A longer description.

        .INPUTS
            Description of objects that can be piped to the script.

        .OUTPUTS
            Description of objects that are output by the script.

        .EXAMPLE
            Example of how to run the script.

        .LINK
            Links to further documentation.

        .NOTES
            Detail on what the script does, if this is needed.
    #>
    [CmdletBinding()]
    param (
        # Description of this parameter.
        [Parameter(Mandatory)]
        [string]
        $Name
    )

    # Function body
}
```

### Tips

- Put detailed explanations in the `.NOTES` section, not the `.DESCRIPTION`
- If writing in a non-native language, prefer simpler words and sentences
- Help is shown via `help Get-Example` or `Get-Example -?`
- Run `help about_Comment_Based_Help` for the full specification
