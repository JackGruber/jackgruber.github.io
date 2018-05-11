---
layout: post
title: "Capture output from command line tools with PowerShell"
tags: [powershell, code, process]
category: PowerShell
bigimg: "/img/head/code.jpg"
---

There are several ways to execute an external process and capture and process its command output in PowerShell.

```powershell
$OutputVariable = (cmd.exe /c ping localhost) | Out-String
```

The problem with this short sample is, the **stderr** is not capture, only the **stdout**. 

With the example below it is possible to capture both outputs.
<script src="https://gist.github.com/JackGruber/142863ba8c76132d7b704b5decb8a8a8.js"></script>