---
name: feishu_installCli_skill
description: 分步引导安装飞书/Lark CLI、初始化配置、scope 授权和验证。单飞书员工使用官方式默认 profile，不使用代称；多飞书员工使用自定义 profile 名称。授权命令只使用 --scope；支持按管理员提供的 Scope A/B 分组授权。
---

# 安装飞书 CLI

## 输出规则

- 默认工作方式是分步引导用户完成安装和配置。
- 严禁一次性输出整套安装文档、完整流程或所有命令。
- 即使用户说“把命令都给我”，也只能输出当前步骤命令，并提示完成后继续下一步。
- 首次回复必须先询问设备类型和飞书数量；在用户回答前不要输出安装、初始化或授权命令。
- 如果用户已经主动说明设备类型或飞书数量，只询问缺失项。
- 每次只输出当前步骤的命令和一句操作说明；步骤结束必须引导用户把执行结果告诉 Agent，无论成功或失败。
- 在确认用户已完成当前步骤目标前，不能进入下一步；如果用户贴出错误，先解释错误并继续处理当前步骤。
- 如果 Agent 具备本机命令执行能力，优先由 Agent 执行安装、检查、验证命令；如果不能执行，再让用户打开终端复制命令。
- 需要浏览器授权时，给用户授权链接并等待用户完成。
- 不要让用户把 App Secret 发到聊天里；只给本机隐藏输入方式。
- scope 授权一次只输出一组命令；A 组完成后再输出 B 组。

首次回复模板：

```text
我会一步一步带你完成。默认不会一次性发所有命令，每次只处理当前步骤。

先确认 2 件事：
1. 你的电脑是 Mac 还是 Windows？
2. 你是配置 1 个飞书，还是多个飞书？

回复后我从第 1 步开始。
```

每一步结尾固定加一句：

```text
执行后把结果发给我；成功或失败都可以。确认这一步完成后，我再带你做下一步。
```

## 基本规则

- 飞书CLI官方引导：https://www.feishu.cn/content/article/7623291503305083853
- 单飞书员工：按官方式默认 profile 配置，不使用代称。
- 多飞书员工：每个飞书一个 profile，用 `<PROFILE>` 占位符，实际填团队约定的 profile 名称。
- profile 名称建议只用小写英文、数字、短横线或下划线，例如 `company_a`、`work`、`personal`。
- 开始前确认本机能使用 `npm` 和 `npx`；如果不能用，先安装 Node.js。
- App Secret 只能在本机终端隐藏输入；不要写进聊天、文档、命令历史或 Skill。
- 授权命令只使用 `auth login --scope`。
- 公开仓库只保留 scope 占位符；内部权限清单由管理员私下提供。

## 场景 1：单飞书员工

### 步骤 1：确认执行方式

```text
如果 Agent 能执行本机命令：让 Agent 直接执行下一步命令。
如果 Agent 不能执行本机命令：Mac 打开 Terminal（终端）；Windows 打开 PowerShell，不要用 Command Prompt。
```

### 步骤 2：安装 CLI

```shell
npm install -g @larksuite/cli
npx -y skills add https://open.feishu.cn --skill -y
lark-cli --version
```

### 步骤 3：初始化配置

macOS Terminal（zsh）：

复制下面命令到终端执行后，终端会提示输入 `App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```zsh
read -s "APP_SECRET?App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | lark-cli config init --brand feishu --app-id "<APP_ID>" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

复制下面命令到 PowerShell 执行后，终端会提示输入 `App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

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

### 步骤 1：确认执行方式

```text
如果 Agent 能执行本机命令：让 Agent 直接执行下一步命令。
如果 Agent 不能执行本机命令：Mac 打开 Terminal（终端）；Windows 打开 PowerShell，不要用 Command Prompt。
```

### 步骤 2：安装 CLI

```shell
npm install -g @larksuite/cli
npx -y skills add https://open.feishu.cn --skill -y
lark-cli --version
```

### 步骤 3：初始化配置

只输出一份初始化模板。把 `<PROFILE>` 替换为团队约定的 profile 名称，把 `<APP_ID>` 替换为对应飞书的 App ID。
profile 名称建议只用小写英文、数字、短横线或下划线，例如 `company_a`。

macOS Terminal（zsh）：

复制下面命令到终端执行后，终端会提示输入 `<PROFILE> App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```zsh
read -s "APP_SECRET?<PROFILE> App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | lark-cli config init --name <PROFILE> --brand feishu --app-id "<APP_ID>" --app-secret-stdin
unset APP_SECRET
```

macOS 示例，假设 profile 名称是 `company_a`，App ID 是 `cli_xxxxxxxxxxxxxxxx`：

```zsh
read -s "APP_SECRET?company_a App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | lark-cli config init --name company_a --brand feishu --app-id "cli_xxxxxxxxxxxxxxxx" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

复制下面命令到 PowerShell 执行后，终端会提示输入 `<PROFILE> App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

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

## 权限列表转命令提示词

用户拿到管理员发的权限列表后，可以把下面提示词发给 AI，让 AI 整理成可直接复制执行的授权命令。

```text
请把下面飞书 CLI scope 权限列表整理成可以直接复制执行的授权命令。

我的场景：
- 终端：Mac Terminal（zsh）/ Windows PowerShell
- 是否多飞书：单飞书 / 多飞书
- profile 名称：<PROFILE>（单飞书留空；多飞书必填）

要求：
1. 只生成 `lark-cli auth login --scope` 命令，不要使用 `--recommend`、`--no-wait`、`--json`。
2. 如果权限数据包含 `scopes.user` 和 `scopes.tenant`，只使用 `scopes.user`。
3. 如果不是 JSON，按管理员标注为“用户授权”的 scope 处理；不确定时先提醒我确认。
4. 去重并保留原 scope 字符串，不翻译、不改名。
5. 如果 scope 超过 120 个，拆成 Scope A/B/C 多组，每组一条命令。
6. 单飞书命令不要带 `--profile`；多飞书命令必须带 `--profile <PROFILE>`。
7. 按我填写的终端只输出一种命令格式。
8. 只输出命令块和必要标题，不要解释。

权限列表：
<PASTE_ADMIN_SCOPE_LIST_HERE>
```
