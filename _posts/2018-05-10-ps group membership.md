---
layout: post
title: "Check group membership with PowerShell"
tags: [powershell, code]
category: PowerShell
bigimg: "/img/head/code.jpg"
---

Check group membership from the current user without Active Directory PowerShell module.

<script src="https://gist.github.com/JackGruber/64aa088e0b52db15ef7ab185313974aa.js"></script>

get a list of alle user groups
```powershell
Get-Groups
```

Check group membership
```powershell
Get-Groups -isMember "DOMAIN\Administrators"
Get-Groups "Administrators"
```
