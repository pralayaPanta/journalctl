---
title: "PowerShell Basics #2: The Help System"
date: 2026-03-15T09:50:35+10:00
draft: false
slug: "powershell-help-system"
description: "Master Get-Help, Get-Command, and About_ articles to find any cmdlet and syntax without leaving your terminal."
categories: [""]
tags: ["PowerShell"]
series: ["PowerShell Basics"]
series_order: 2
featureimage: "images/posts/powershell-help-system.jpg"
---

> **TL;DR**
> - Experts don't memorize thousands of cmdlets â€” they use the built-in help system.
> - Always run Update-Help -Force as administrator to download local help documentation.
> - Master the 4-step terminal discovery pattern: Get-Command, help, help -Parameter, help -Examples.

Experts don't memorise thousands of PowerShell commands. Don Jones and Jeff Hicks cite a study in *Learn PowerShell in a Month of Lunches* â€” two groups of IT pros, beginners and experts, sat a written test on PowerShell. Scores were similar across both groups. Then they ran the test again with access to a running PowerShell session. The gap between the two groups became significant. The difference? Experts knew how to use the help system to find answers on the fly. 

The PowerShell help system is not documentation you read once and forget â€” it's a tool you use every single session.

## Prerequisites
- PowerShell 7+ installed (see [Post #1](https://journalctl.io/get-started-with-powershell/))
- Administrator rights on your machine (required for Update-Help)
- Internet access for initial help download

## Update Your Help Files First

Before running any Get-Help examples in this post, download the latest help files. Without this step, many commands return incomplete output or no content at all. Run this as administrator:

```powershell
Update-Help -Force
```

Some modules throw errors during the update â€” that's expected. It means the module author didn't configure updatable help. Ignore those and move on. Run this periodically as PowerShell modules update.

### Offline Machines (Save-Help)
If working in an air-gapped environment, use Save-Help on a connected machine, then copy the files across:

```powershell
# On the internet-connected machine
Save-Help -DestinationPath C:\PSHelp

# On the offline machine (as administrator)
Update-Help -SourcePath C:\PSHelp
```

## Exploring Cmdlet Documentation with Get-Help

Get-Help is your first stop when you don't know how a command works, what parameters it accepts, or what it returns.

```powershell
Get-Help -Name Get-Service
```

The output is structured into key sections:
- **NAME**  The command name
- **SYNOPSIS**  One-line summary
- **SYNTAX**  Available parameter sets
- **DESCRIPTION**  Detailed explanation
- **RELATED LINKS**  Online documentation & related cmdlets

### 1. Viewing Full Documentation
```powershell
help Get-Service -Full
```

### 2. Viewing Command Examples
```powershell
help Get-Service -Examples
```

### 3. Inspecting Parameter Details
```powershell
help Get-Service -Parameter ComputerName
```

## Finding Commands with Get-Command

When you don't know the exact cmdlet name, search by Verb, Noun, or wildcard pattern using Get-Command:

```powershell
# Find all cmdlets with Noun 'Service'
Get-Command -Noun Service

# Find all cmdlets with Verb 'Get'
Get-Command -Verb Get -Module Microsoft.Graph.Users

# Search by wildcard name pattern
Get-Command -Name *User* -CommandType Cmdlet
```

## The Terminal Discovery Pattern

```powershell
# Step 1: Discover candidate commands
Get-Command -Noun Service

# Step 2: Learn how to use a candidate cmdlet
help Get-Service -Full

# Step 3: Check specific parameter details
help Get-Service -Parameter ComputerName

# Step 4: Inspect real-world examples
help Get-Service -Examples
```

## Wrapping Up

| Command | Purpose | Key Parameter / Syntax |
|---|---|---|
| Update-Help | Download or refresh local help files | -Force, -SourcePath |
| help <cmdlet> | View interactive paged help documentation | -Full, -Examples, -Parameter |
| Get-Command | Search for cmdlets by name pattern or noun/verb | -Noun, -Verb, -Module |
| Get-Help about_* | Read conceptual PowerShell topics | bout_Execution_Policies, bout_Pipelines |

