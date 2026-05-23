---
name: feishu_installCli_skill
description: 输出飞书/Lark CLI 安装、初始化、scope 授权和验证命令。单飞书员工使用官方式默认 profile，不使用代称；多飞书员工使用自定义 profile 名称。授权命令只使用 --scope；支持按管理员提供的 Scope A/B 分组授权。
---

# 安装飞书 CLI

## 基本规则

- 飞书CLI官方引导：https://www.feishu.cn/content/article/7623291503305083853
- 单飞书员工：按官方式默认 profile 配置，不使用代称。
- 多飞书员工：每个飞书一个 profile，用 `<PROFILE>` 占位符，实际填团队约定的 profile 名称。
- 开始前确认本机能使用 `npm` 和 `npx`；如果不能用，先安装 Node.js。
- App Secret 只能在本机终端隐藏输入；不要写进聊天、文档、命令历史或 Skill。
- 授权命令只使用 `auth login --scope`。
- 公开仓库只保留 scope 占位符；内部权限清单由管理员私下提供。

## 场景 1：单飞书员工

### 步骤 1：打开终端

```text
Mac：打开 Terminal（终端）
Windows：打开 PowerShell，不要用 Command Prompt
```

### 步骤 2：安装 CLI

```shell
npm install -g @larksuite/cli
npx -y skills add https://open.feishu.cn --skill -y
lark-cli --version
```

### 步骤 3：初始化配置

macOS Terminal（zsh）：

```zsh
read -s "APP_SECRET?App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | lark-cli config init --brand feishu --app-id "<APP_ID>" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

```powershell
$AppSecret = Read-Host -Prompt "App Secret" -AsSecureString
$Bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($AppSecret)
try {
  $PlainSecret = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
  $PlainSecret | lark-cli config init --brand feishu --app-id "<APP_ID>" --app-secret-stdin
} finally {
  if ($Bstr -ne [IntPtr]::Zero) { [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($Bstr) }
  Remove-Variable AppSecret, PlainSecret -ErrorAction SilentlyContinue
}
```

### 步骤 4：检查配置

```shell
lark-cli profile list
```

### 步骤 5：授权

如果管理员提供的是 A/B 两组 scope，执行以下两组授权命令；如果管理员另有说明，以管理员提供的 scope 为准。

Scope A：

```zsh
SCOPES_A='<PASTE_SCOPE_GROUP_A>'
lark-cli auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
SCOPES_B='<PASTE_SCOPE_GROUP_B>'
lark-cli auth login --scope "$SCOPES_B"
```

Windows PowerShell：使用同一份 scope 字符串，把 `SCOPES_A='<PASTE_SCOPE_GROUP_A>'` 改成 `$SCOPES_A = '<PASTE_SCOPE_GROUP_A>'`，命令改成 `lark-cli auth login --scope $SCOPES_A`。

### 步骤 6：验证

官方基础验证命令是 `auth status`；这里使用 `--verify` 做更完整检查。

```shell
lark-cli auth status --verify
```

## 场景 2：多飞书员工

### 步骤 1：打开终端

```text
Mac：打开 Terminal（终端）
Windows：打开 PowerShell，不要用 Command Prompt
```

### 步骤 2：安装 CLI

```shell
npm install -g @larksuite/cli
npx -y skills add https://open.feishu.cn --skill -y
lark-cli --version
```

### 步骤 3：初始化配置

只输出一份初始化模板。把 `<PROFILE>` 替换为团队约定的 profile 名称，把 `<APP_ID>` 替换为对应飞书的 App ID。

macOS Terminal（zsh）：

```zsh
read -s "APP_SECRET?<PROFILE> App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | lark-cli config init --name <PROFILE> --brand feishu --app-id "<APP_ID>" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

```powershell
$AppSecret = Read-Host -Prompt "<PROFILE> App Secret" -AsSecureString
$Bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($AppSecret)
try {
  $PlainSecret = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
  $PlainSecret | lark-cli config init --name <PROFILE> --brand feishu --app-id "<APP_ID>" --app-secret-stdin
} finally {
  if ($Bstr -ne [IntPtr]::Zero) { [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($Bstr) }
  Remove-Variable AppSecret, PlainSecret -ErrorAction SilentlyContinue
}
```

### 步骤 4：检查配置

```shell
lark-cli profile list
```

### 步骤 5：授权

- 每个飞书应用的 scope 以管理员提供的信息为准，不要混用其他应用的 scope。

Scope A：

```zsh
SCOPES_A='<PASTE_SCOPE_GROUP_A>'
lark-cli --profile <PROFILE> auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
SCOPES_B='<PASTE_SCOPE_GROUP_B>'
lark-cli --profile <PROFILE> auth login --scope "$SCOPES_B"
```

Windows PowerShell：使用同一份 scope 字符串，把 `SCOPES_A='<PASTE_SCOPE_GROUP_A>'` 改成 `$SCOPES_A = '<PASTE_SCOPE_GROUP_A>'`，命令改成 `lark-cli --profile <PROFILE> auth login --scope $SCOPES_A`。

### 步骤 6：验证

官方基础验证命令是 `auth status`；这里使用 `--verify` 做更完整检查。

```shell
lark-cli --profile <PROFILE> auth status --verify
```
