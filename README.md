# PowerShell Style Skill

A Claude Code skill providing PowerShell best practices and style guidance,
based on the community-driven
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

### As a Claude Code plugin

This repository is both a plugin and its own single-plugin marketplace.
Add the marketplace, then install the plugin:

```text
/plugin marketplace add ryanspletzer/powershell-style
/plugin install powershell-style@ryanspletzer
```

The skill is then available as `powershell-style:powershell-style`.

To declare it in `~/.claude/settings.json` instead:

```json
{
  "extraKnownMarketplaces": {
    "ryanspletzer": {
      "source": {
        "source": "github",
        "repo": "ryanspletzer/powershell-style"
      }
    }
  },
  "enabledPlugins": {
    "powershell-style@ryanspletzer": true
  }
}
```

Pick up new commits with:

```text
/plugin marketplace update ryanspletzer
```

### As an Antigravity CLI plugin

`agy` reads the native manifest in `.antigravity-plugin/plugin.json`.
Install straight from GitHub:

```bash
agy plugin install https://github.com/ryanspletzer/powershell-style
```

The install is a copy, not a tracked marketplace entry.
Re-run the same command to pick up new commits.

### Local development

Load the working tree directly, without installing:

```bash
claude --plugin-dir ~/git/powershell-style
```

### As a standalone skill

Copy or symlink the skill directory into your skills folder:

```bash
ln -s ~/git/powershell-style/skills/powershell-style ~/.claude/skills/powershell-style
```

## Files

| File | Description |
|------|-------------|
| `.claude-plugin/plugin.json` | Claude Code plugin manifest |
| `.claude-plugin/marketplace.json` | Marketplace manifest listing this plugin |
| `.antigravity-plugin/plugin.json` | Antigravity CLI plugin manifest |
| `skills/powershell-style/SKILL.md` | Main skill file with quick reference and core guidance |
| `skills/powershell-style/references/naming.md` | Detailed naming conventions |
| `skills/powershell-style/references/functions.md` | Advanced function patterns |
| `skills/powershell-style/references/error-handling.md` | Error handling philosophy |
| `skills/powershell-style/references/output.md` | Output stream guidance |
| `skills/powershell-style/references/documentation.md` | Comment-based help and commenting rules |
| `skills/powershell-style/references/performance.md` | Performance measurement and trade-offs |
| `skills/powershell-style/references/security.md` | Credentials, SecureString, and persistence |
| `LICENSE` | CC BY-SA 4.0 license |
| `README.md` | This file |

## Complementary Tools

This skill works alongside:

- **PowerShell Editor Services LSP** - Code intelligence, go-to-definition
- **PSScriptAnalyzer** - Static analysis for code issues

The skill provides higher-level design guidance,
while these tools catch syntax-level issues.

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
by Don Jones, Matt Penny, Carlos Perez, Joel Bennett, and the PowerShell community.

## License

[Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](LICENSE)

This license matches the original PoshCode guide,
ensuring derivative works maintain the same open licensing terms.
