# Performance

Based on
[PoshCode/PowerShellPracticeAndStyle - Performance](https://github.com/PoshCode/PowerShellPracticeAndStyle/blob/master/Best-Practices/Performance.md)
(CC BY-SA 4.0).

## PERF-01: If Performance Matters, Test It

Use `Measure-Command` or the Profiler module to measure
on the actual hardware and PowerShell version where performance matters:

```powershell
# foreach is faster than ForEach-Object for in-memory collections
foreach ($result in Do-Something) {
    $result.PropertyOne + $result.PropertyTwo
}

# Pipeline version — slower but streams objects
Do-Something | ForEach-Object {
    $_.PropertyOne + $_.PropertyTwo
}
```

The `foreach` language construct is generally faster than piping
to `ForEach-Object`, but always measure in your specific context.

## PERF-02: Balance Performance with Readability

Performance gains on small datasets rarely justify less readable code.
For ten items, a 30% speedup is negligible —
use the approach that is easiest to read and maintain.

For large datasets, streaming matters.
Consider these approaches from most readable to most performant:

### Approach 1: Load Everything into Memory

Simple and readable, but loads the entire file at once:

```powershell
$content = Get-Content -Path file.txt

foreach ($line in $content) {
    Do-Something -Input $line
}
```

### Approach 2: Stream via Pipeline

Streams line-by-line through the pipeline,
avoiding memory buffering for large files:

```powershell
Get-Content -Path file.txt |
    ForEach-Object -Process {
        Do-Something -Input $_
    }
```

### Approach 3: Use .NET Directly

Best performance for very large files,
but harder to follow:

```powershell
$sr = New-Object -TypeName System.IO.StreamReader -ArgumentList file.txt

while ($sr.Peek() -ge 0) {
    $line = $sr.ReadLine()
    Do-Something -Input $line
}
```

### Approach 4: Wrap .NET in PowerShell Commands

The ideal balance — native PowerShell commands wrapping .NET classes:

```powershell
$handle = Open-TextFile -Path file.txt

while (-not (Test-TextFile -Handle $handle)) {
    Do-Something -Input (Read-TextFile -Handle $handle)
}
```

Writing wrapper commands requires more work
but provides both performance and readability.

## PERF-03: General Performance Hierarchy

From fastest to slowest (as a rough guideline):

1. **Language features** — `foreach`, `if`, operators
2. **Compiled .NET methods** — `[System.IO.File]::ReadAllLines()`
3. **PowerShell script** — inline expressions and script blocks
4. **Cmdlet/function calls** — overhead of parameter binding and pipeline

This hierarchy is counter-intuitive:
calling compiled cmdlets can be slower than inline script
due to invocation overhead.
The overhead is significant unless the cmdlet does substantial work per call.

**Always measure.** These are guidelines, not absolutes.
