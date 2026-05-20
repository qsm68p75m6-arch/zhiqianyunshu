# Claude Code 接入 DeepSeek 操作手册

## 1. 目标

本文档说明如何在 Windows 上为 `Claude Code` 配置：

1. 使用 `Anthropic` 官方模型
2. 使用 `DeepSeek` 的 Anthropic 兼容接口
3. 在 `Claude` 和 `DeepSeek` 之间快速切换
4. 处理常见配置错误

默认配置目录：`C:\Users\anoth\.claude\`

## 2. 配置原理

`Claude Code` 读取的是 `settings.json`。

切换模型时，不能只改 `model`，因为 `Claude` 和 `DeepSeek` 的认证方式不同：

1. `Claude 官方` 使用 `ANTHROPIC_API_KEY`
2. `DeepSeek 兼容接口` 使用 `ANTHROPIC_AUTH_TOKEN` 和 `ANTHROPIC_BASE_URL`

因此最稳妥的方式是维护两份独立配置文件，再用脚本切换。

## 3. 目录结构

```
settings.json
settings.claude.json
settings.deepseek.json
switch-claude.ps1
switch-deepseek.ps1
backup\\
```

## 4. Claude 官方配置

文件：`settings.claude.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "alwaysThinkingEnabled": true,
  "enabledPlugins": {},
  "env": {
    "ANTHROPIC_API_KEY": "你的 Claude 官方 API Key"
  },
  "hooks": {},
  "includeCoAuthoredBy": false,
  "language": "Chinese",
  "outputStyle": "default",
  "permissions": {
    "allow": [
      "Edit", "Glob", "Grep", "KillShell", "Read", "Task",
      "TodoWrite", "Write", "WebFetch", "WebSearch",
      "mcp__grok-search", "mcp__augment-context-engine"
    ],
    "deny": []
  },
  "model": "opus",
  "statusLine": {
    "command": "ccline",
    "padding": 0,
    "type": "command"
  }
}
```

## 5. DeepSeek 配置

文件：`settings.deepseek.json`

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "alwaysThinkingEnabled": true,
  "enabledPlugins": {},
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "你的 DeepSeek API Key",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic"
  },
  "hooks": {},
  "includeCoAuthoredBy": false,
  "language": "Chinese",
  "outputStyle": "default",
  "permissions": {
    "allow": [
      "Edit", "Glob", "Grep", "KillShell", "Read", "Task",
      "TodoWrite", "Write", "WebFetch", "WebSearch",
      "mcp__grok-search", "mcp__augment-context-engine"
    ],
    "deny": []
  },
  "model": "deepseek-v4-flash",
  "statusLine": {
    "command": "ccline",
    "padding": 0,
    "type": "command"
  }
}
```

## 6. 切换脚本

### switch-claude.ps1

```powershell
$claudeDir = "C:\Users\anoth\.claude"
$source = Join-Path $claudeDir "settings.claude.json"
$target = Join-Path $claudeDir "settings.json"
$backupDir = Join-Path $claudeDir "backup"

if (!(Test-Path $source)) {
    Write-Error "Missing file: $source"
    exit 1
}

if (!(Test-Path $backupDir)) {
    New-Item -Path $backupDir -ItemType Directory | Out-Null
}

Get-Content $source | ConvertFrom-Json | Out-Null

if (Test-Path $target) {
    $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
    $backupFile = Join-Path $backupDir "settings.backup.$timestamp.json"
    Copy-Item $target $backupFile -Force
    Write-Host "Backup created: $backupFile"
}

Copy-Item $source $target -Force

$config = Get-Content $target | ConvertFrom-Json
Write-Host "Switched to Claude config"
Write-Host "Model: $($config.model)"
Write-Host "Restart Claude Code to apply changes"
```

### switch-deepseek.ps1

```powershell
$claudeDir = "C:\Users\anoth\.claude"
$source = Join-Path $claudeDir "settings.deepseek.json"
$target = Join-Path $claudeDir "settings.json"
$backupDir = Join-Path $claudeDir "backup"

if (!(Test-Path $source)) {
    Write-Error "Missing file: $source"
    exit 1
}

if (!(Test-Path $backupDir)) {
    New-Item -Path $backupDir -ItemType Directory | Out-Null
}

Get-Content $source | ConvertFrom-Json | Out-Null

if (Test-Path $target) {
    $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
    $backupFile = Join-Path $backupDir "settings.backup.$timestamp.json"
    Copy-Item $target $backupFile -Force
    Write-Host "Backup created: $backupFile"
}

Copy-Item $source $target -Force

$config = Get-Content $target | ConvertFrom-Json
Write-Host "Switched to DeepSeek config"
Write-Host "Model: $($config.model)"
Write-Host "Restart Claude Code to apply changes"
```

## 7. PowerShell 快捷命令

```powershell
function Switch-ClaudeConfig {
    param(
        [Parameter(Mandatory = $true)]
        [string]$SourceFile,
        [Parameter(Mandatory = $true)]
        [string]$ModeName
    )

    $claudeDir = "C:\Users\anoth\.claude"
    $source = Join-Path $claudeDir $SourceFile
    $target = Join-Path $claudeDir "settings.json"
    $backupDir = Join-Path $claudeDir "backup"

    if (!(Test-Path $source)) {
        Write-Error "Missing file: $source"
        return
    }

    if (!(Test-Path $backupDir)) {
        New-Item -Path $backupDir -ItemType Directory | Out-Null
    }

    Get-Content $source | ConvertFrom-Json | Out-Null

    if (Test-Path $target) {
        $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
        $backupFile = Join-Path $backupDir "settings.backup.$timestamp.json"
        Copy-Item $target $backupFile -Force
        Write-Host "Backup created: $backupFile"
    }
    Copy-Item $source $target -Force

    $config = Get-Content $target | ConvertFrom-Json
    Write-Host "Switched to $ModeName"
    Write-Host "Model: $($config.model)"
    Write-Host "Restart Claude Code to apply changes"
}

function use-claude {
    Switch-ClaudeConfig -SourceFile "settings.claude.json" -ModeName "Claude"
}

function use-deepseek {
    Switch-ClaudeConfig -SourceFile "settings.deepseek.json" -ModeName "DeepSeek"
}
```

## 8. PowerShell 执行策略

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
. $PROFILE
Get-Command use-claude
Get-Command use-deepseek
```

## 9. 日常使用流程

### 切到 Claude

```powershell
use-claude
```

### 切到 DeepSeek

```powershell
use-deepseek
```

## 10. JSON 常见错误

### enabledPlugins 类型错误

错误写法：`\"enabledPlugins\": []`
正确写法：`\"enabledPlugins\": {}`

### token 换行

错误写法：`\"ANTHROPIC_AUTH_TOKEN\": \"abc\\n\"`
正确写法：`\"ANTHROPIC_AUTH_TOKEN\": \"abc\"`

## 11. 配置校验

```powershell
Get-Content C:\Users\anoth\.claude\settings.json | ConvertFrom-Json | Out-Null
Get-Content C:\Users\anoth\.claude\settings.claude.json | ConvertFrom-Json | Out-Null
Get-Content C:\Users\anoth\.claude\settings.deepseek.json | ConvertFrom-Json | Out-Null
```

## 12. 排错顺序

1. 先检查 JSON 语法
2. 再检查 `enabledPlugins` 是否为对象
3. 再检查 token 是否单行
4. 再检查 `ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN` 是否有效
5. 再检查 `ANTHROPIC_BASE_URL` 是否正确
6. 最后检查 `model` 是否为当前接口支持的模型名

## 13. 结论

最稳的维护方式是：
1. `settings.claude.json` 专门给 Anthropic 官方
2. `settings.deepseek.json` 专门给 DeepSeek
3. `settings.json` 只作为当前生效文件
4. 用 PowerShell 脚本切换，不手工反复改同一个文件