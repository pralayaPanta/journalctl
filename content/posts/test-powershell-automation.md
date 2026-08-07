---
title: "Automating Entra ID Stale Account Cleanup with PowerShell Test Post"
date: 2026-07-31T08:14:30+10:00
draft: true
slug: "test-powershell-automation"
description: "A complete PowerShell 7+ Filter Left automation script to identify and disable inactive Entra ID user accounts."
categories: [""]
tags: ["PowerShell", "Entra ID", "Security", "Automation"]
series: ["PowerShell Automation"]
series_order: 1
featureimage: "images/posts/test-post-powershell-filter-left.jpg"
---

> **TL;DR**
> - Inactive user accounts increase your identity attack surface if left unmonitored in Entra ID.
> - Always use server-side "Filter Left" parameters (`-Filter`) with `Get-MgUser` rather than client-side piping.
> - This script audits user sign-in activity and safely generates a CSV report before disabling accounts.

Unmonitored, inactive user accounts are one of the primary targets for identity-based attacks. When employees leave an organization or contractor accounts expire, leaving those accounts enabled creates unnecessary security risk.

In this guide, we'll build a production-ready PowerShell 7 script using the Microsoft Graph SDK that applies senior-level **Filter Left** practices to identify and disable inactive Entra ID accounts.

## Prerequisites

- **PowerShell Version:** 7.2+ Core (`pwsh.exe`)
- **Module Required:** `Microsoft.Graph.Users` v2.0+
- **Graph API Permissions:** `User.ReadWrite.All`, `AuditLog.Read.All`
- **Minimum Role:** User Administrator or Privileged Authentication Administrator

## The PowerShell Script (Filter Left)

Instead of pulling all directory objects into memory and piping to `Where-Object` (which causes extreme latency in large tenants), we pass OData `-Filter` expressions directly to Microsoft Graph:

```powershell
<#
.SYNOPSIS
    Audits and disables inactive Entra ID user accounts older than 90 days.
.DESCRIPTION
    Uses Microsoft Graph SDK with Filter Left parameters for maximum performance.
#>

[CmdletBinding()]
param (
    [int]$DaysInactive = 90,
    [string]$ReportPath = "C:\Reports\InactiveUsers-$(Get-Date -Format 'yyyyMMdd').csv"
)

# 1. Authenticate to Microsoft Graph with required scopes
Connect-MgGraph -Scopes "User.ReadWrite.All", "AuditLog.Read.All" -NoWelcome

# 2. Calculate threshold date
$CutoffDate = (Get-Date).AddDays(-$DaysInactive).ToString("yyyy-MM-ddTHH:mm:ssZ")

# 3. Filter Left: Server-side OData query for enabled member users
$FilterQuery = "accountEnabled eq true and userType eq 'Member'"
Write-Host "Fetching enabled member accounts from Entra ID..." -ForegroundColor Cyan

$Users = Get-MgUser -Filter $FilterQuery `
    -Property "id", "displayName", "userPrincipalName", "accountEnabled", "signInActivity" `
    -All

Write-Host "Retrieved $($Users.Count) enabled users. Checking sign-in activity..." -ForegroundColor Green

# 4. Process inactive accounts
$InactiveReport = [System.Collections.Generic.List[PSObject]]::new()

foreach ($User in $Users) {
    $LastSignIn = $User.SignInActivity.LastSignInDateTime
    
    if (-not $LastSignIn -or $LastSignIn -lt (Get-Date).AddDays(-$DaysInactive)) {
        $InactiveReport.Add([PSCustomObject]@{
            UserId            = $User.Id
            DisplayName       = $User.DisplayName
            UserPrincipalName = $User.UserPrincipalName
            LastSignInDate    = if ($LastSignIn) { $LastSignIn } else { "Never Logged In" }
            Status            = "Inactive"
        })
    }
}

# 5. Export Report
$InactiveReport | Export-Csv -Path $ReportPath -NoTypeInformation
Write-Host "Audit report exported to $ReportPath ($($InactiveReport.Count) inactive users found)." -ForegroundColor Yellow
```

## How to Test Execution

Run the script in `-WhatIf` mode or review the CSV report at `$ReportPath` before executing administrative state changes in your tenant.

## Wrapping Up

| Cmdlet / Parameter | Purpose | Key Strategy |
|---|---|---|
| `Get-MgUser -Filter` | Query Entra ID users | Filter Left server-side (`accountEnabled eq true`) |
| `-Property` | Select specific Graph properties | Reduces payload size & speeds up API queries |
| `Connect-MgGraph` | Authenticate Graph SDK session | Require `User.ReadWrite.All` & `AuditLog.Read.All` |
| `Export-Csv` | Export audit trail | Generate CSV report before mutating account states |

