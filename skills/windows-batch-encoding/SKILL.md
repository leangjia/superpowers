---
name: windows-batch-encoding
description: Use when creating .bat files that contain Chinese characters, when .bat files fail to run with garbled text or parsing errors, or when PowerShell scripts (.ps1) with Chinese text produce syntax errors
---

# Windows Batch Encoding

## Overview

cmd.exe and PowerShell 5.1 on Chinese Windows use different default encodings. Getting encoding wrong causes .bat files to fail silently or display garbled Chinese text. The correct approach depends on the file type and content.

## Core Principles

1. **.bat files must NOT have UTF-8 BOM** — BOM (EF BB BF) breaks cmd.exe parsing, causing `'@echo' 不是内部或外部命令` error
2. **.bat files use ANSI encoding** — on Chinese Windows this is GBK (code page 936). cmd.exe reads .bat files using the system OEM code page
3. **.ps1 files MUST have UTF-8 BOM** — PowerShell 5.1 interprets BOM-less UTF-8 as ANSI/GBK, garbling Chinese characters
4. **.bat files need CRLF (`\r\n`) line endings** — Unix LF-only endings cause cmd.exe parsing failures
5. **Best practice**: Minimize Chinese in .bat files; use .ps1 scripts with UTF-8+BOM for the actual logic, and simple .bat wrappers to invoke them

## Encoding Matrix

| File Type | Chinese chars | Encoding | Line Endings | BOM | `chcp` needed |
|-----------|--------------|----------|--------------|-----|---------------|
| .bat | None | ANSI/GBK | CRLF | No | No |
| .bat | Chinese | GBK (936) | CRLF | No | No |
| .ps1 | Chinese | UTF-8 | CRLF or LF | Yes | N/A |
| .ps1 | None | UTF-8 | CRLF or LF | No | N/A |

## Recommended Pattern: .bat Wrapper + .ps1 Logic

### .bat (launcher, no Chinese, GBK encoding, CRLF line endings)

```bat
@echo off
powershell -NoProfile -ExecutionPolicy Bypass -File "%~dp0script.ps1"
if %errorlevel% neq 0 pause
```

### .ps1 (logic, UTF-8 with BOM, Chinese text OK)

```powershell
Write-Host "固定资产管理系统 - 启动" -ForegroundColor Cyan
# ... full PowerShell logic here ...
```

## How to Write Files with Correct Encoding

### Using PowerShell to create .bat files (GBK)

```powershell
$content = "@echo off`r`npowershell -File script.ps1`r`n"
[System.IO.File]::WriteAllText("output.bat", $content, [System.Text.Encoding]::GetEncoding(936))
```

### Using PowerShell to create .ps1 files (UTF-8 + BOM)

```powershell
$utf8bom = New-Object System.Text.UTF8Encoding $true
[System.IO.File]::WriteAllText("output.ps1", $content, $utf8bom)
```

### Using Python to create .bat files (GBK)

```python
with open('output.bat', 'w', encoding='gbk') as f:
    f.write(content)
```

### Using Python to create .ps1 files (UTF-8 + BOM)

```python
with open('output.ps1', 'w', encoding='utf-8-sig') as f:
    f.write(content)
```

### Verify encoding (Python)

```python
with open('file.bat', 'rb') as f:
    b = f.read()
    print(f'BOM: ({b[0]},{b[1]},{b[2]})')  # (239,187,191) for BOM
    print(f'CRLF count: {b.count(b"\r\n")}, LF-only: {b.count(b"\n") - b.count(b"\r\n")}')
```

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| UTF-8 BOM in .bat | `'@echo' 不是内部或外部命令` | Save as GBK without BOM |
| UTF-8 without BOM in .ps1 | Chinese garbled or syntax errors about `}` | Add UTF-8 BOM |
| LF-only line endings in .bat | `'rshell' 不是内部或外部命令` (truncated commands) | Use CRLF line endings |
| Using `$pid` in .ps1 | `无法覆盖变量 PID` | Use `$procId` instead (`$pid` is read-only) |
| Chinese text in .bat with `chcp 65001` | Text garbled without BOM on some systems | Use GBK encoding, skip `chcp` |
| Piping with `^|` called from PowerShell | Pipe not passed correctly to cmd.exe | Use .bat wrappers with minimal inline logic |

## PowerShell Reserved Variables

PowerShell has automatic variables that are read-only and will cause `SessionStateUnauthorizedAccessException` if you try to write to them:

- `$pid` — Process ID of current PowerShell process
- `$?` — Last command success status
- `$_` — Current pipeline object
- `$args` — Script arguments

Always use descriptive names like `$procId`, `$portCheck`, `$srvPid`, `$maxRetry`.

## Flask Debug Mode on Windows

When killing a Flask app that runs with `debug=True`:

- Flask's reloader spawns child processes
- Killing the child causes the watchdog to create a new child
- Use `taskkill /f /t /pid <parent>` to kill the process tree
- Or use `taskkill /f /fi "WINDOWTITLE eq <title>*"` targeting the window title set by `start "<title>"`
- Fallback: `taskkill /f /im python.exe` (only on dedicated machines)
