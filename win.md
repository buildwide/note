# test
if (!(Test-Path $PROFILE)) { New-Item -Type File -Path $PROFILE -Force }
code $PROFILE
# install
Install-Module PSReadLine -Force -SkipPublisherCheck -AllowClobber
Set-ExecutionPolicy RemoteSigned -Force
Update-Module PSReadLine

# =========>config in \Documents\WindowsPowerShell\Microsoft.PowerShell_profile
Import-Module PSReadLine

# 核心功能配置
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -MaximumHistoryCount 10000
Set-PSReadLineOption -HistorySavePath "$HOME\Documents\PowerShell\history.txt"

# 键位绑定
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
Set-PSReadLineKeyHandler -Key Tab -Function Complete

Write-Host "PowerShell Zsh-like autocomplete enabled!" -ForegroundColor Green
# =========>end file


# source
. $PROFILE

# 权限
icacls

## 1. 移除所有继承的权限并禁用继承（这是最关键的一步）
icacls ".\sl_dev.pem" /inheritance:r

## 2. 为你当前的用户授予【读取】权限（这是必需的一步）
whoami
icacls ".\sl_dev.pem" /grant:r "USERNAME:(R)"

## 3. （可选但推荐）为 SYSTEM 账户授予读取权限
icacls ".\sl_dev.pem" /grant "NT AUTHORITY\SYSTEM:(R)"

## 4. 验证最终的权限设置
icacls ".\sl_dev.pem"