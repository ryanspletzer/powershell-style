# PowerShell Style Skill

A Claude Code skill providing PowerShell best practices and style guidance based
on the community-driven
[PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle)
guide.

## Purpose

This skill helps Claude write clean, maintainable PowerShell code by providing:

- **Naming conventions** - Verb-Noun patterns, parameter naming, case rules
- **Function structure** - Advanced functions, CmdletBinding, validation
- **Error handling** - Try/catch patterns, terminating errors, best practices
- **Output formatting** - Stream selection, pipeline output, object design
- **Documentation** - Comment-based help, parameter docs, commenting rules
- **Performance** - Measurement, readability trade-offs, performance hierarchy
- **Security** - PSCredential, SecureString, credential persistence

## Installation

### For Claude Code Users

Copy this directory to your Claude Code skills folder or symlink it:

```bash
ln -s ~/git/powershell-style ~/.claude/skills/powershell-style
```

### For anthropics/skills Submission

This skill is structured for submission to
[anthropics/skills](https://github.com/anthropics/skills):

1. Fork the anthropics/skills repository
2. Copy this directory into your fork
3. Submit a pull request

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill file with quick reference and core guidance |
| `references/naming.md` | Detailed naming conventions |
| `references/functions.md` | Advanced function patterns |
| `references/error-handling.md` | Error handling philosophy |
| `references/output.md` | Output stream guidance |
| `references/documentation.md` | Comment-based help and commenting rules |
| `references/performance.md` | Performance measurement and trade-offs |
| `references/security.md` | Credentials, SecureString, and persistence |
| `LICENSE` | CC BY-SA 4.0 license |
| `README.md` | This file |

## Complementary Tools

This skill works alongside:

- **PowerShell Editor Services LSP** - Code intelligence, go-to-definition
- **PSScriptAnalyzer** - Static analysis for code issues

The skill provides higher-level design guidance while these tools catch
syntax-level issues.

## Quick Reference

| Topic | Key Rule |
|-------|----------|
| Function names | `Verb-Noun` with approved verbs |
| Parameters | Full names, PascalCase, explicit types |
| Indentation | 4 spaces (not tabs) |
| Braces | Opening brace on same line (OTBS) |
| Line length | 115 characters max |
| Output | Objects via pipeline, not `Write-Host` |

## Attribution

Based on [PoshCode/PowerShellPracticeAndStyle](https://github.com/PoshCode/PowerShellPracticeAndStyle)
by Don Jones, Matt Penny, Carlos Perez, Joel Bennett, and the PowerShell
community.

## License

[Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](LICENSE)

This license matches the original PoshCode guide, ensuring derivative works
maintain the same open licensing terms.
