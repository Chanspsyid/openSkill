---
name: feishu_installCli_skill
description: 分步引导安装飞书/Lark CLI、初始化配置、scope 授权和验证。单飞书员工使用官方式默认 profile，不使用代称；多飞书员工使用自定义 profile 名称。授权命令只使用 --scope；支持按管理员提供的 Scope A/B 分组授权。
---

# 安装飞书 CLI

## 输出规则

- 默认工作方式是分步引导用户完成安装和配置。
- 严禁一次性输出整套安装文档、完整流程或所有命令。
- 即使用户说“把命令都给我”，也只能输出当前步骤命令，并提示完成后继续下一步。
- 首次回复必须先询问设备类型和未来要连接的飞书数量；在用户回答前不要输出安装、初始化或授权命令。
- 询问飞书数量时，要说明：只连接 1 个飞书会更简单；如果现在或未来会连接多个飞书，需要使用 profile 区分。
- 如果用户已经主动说明设备类型或未来连接的飞书数量，只询问缺失项。
- 每次只输出当前步骤的命令和一句操作说明；步骤结束必须引导用户把执行结果告诉 Agent，无论成功或失败。
- 在确认用户已完成当前步骤目标前，不能进入下一步；如果用户贴出错误，先解释错误并继续处理当前步骤。
- 如果用户执行报错，停留在当前步骤，要求用户发送终端截图或完整文本报错；Agent 负责根据截图或报错继续排查，直到当前步骤成功后再进入下一步。
- 如果 Agent 具备本机命令执行能力，优先由 Agent 执行安装、检查、验证命令；如果不能执行，再让用户打开终端复制命令。
- 需要浏览器授权时，给用户授权链接并等待用户完成。
- 不要让用户把 App Secret 发到聊天里；只给本机隐藏输入方式。
- 用户手动替换命令占位符后，先让用户把修改后的命令发给 Agent 确认；Agent 检查无误后，再让用户执行。
- Agent 检查修改后的命令时，至少确认：没有 `APP_ID_HERE` / `PROFILE_NAME_HERE` / `PASTE_SCOPE_GROUP_A_HERE` 等占位符，没有左尖括号或右尖括号，App ID 以 `cli_` 开头，多飞书命令使用 `--profile PROFILE_NAME` 而不是 `-- PROFILE_NAME`。
- 检查命令时不得要求用户发送 App Secret；初始化命令只包含 App ID 和 profile，App Secret 由终端隐藏输入。
- scope 授权一次只输出一组命令；A 组完成后再输出 B 组。
- 输出给员工的命令使用 `APP_ID_HERE`、`PROFILE_NAME_HERE` 这类占位符，避免使用尖括号包住 APP_ID 这类写法。
- 如果用户从其他资料看到尖括号包住的内容，必须提醒：左右尖括号也是占位符的一部分，替换后命令里不能留下尖括号。
- 授权验证通过后，可以提供可选验收案例；验收案例会真实创建日程，必须先征得用户明确同意，不能作为默认必做步骤。

首次回复模板：

```text
我会一步一步带你完成。默认不会一次性发所有命令，每次只处理当前步骤。

先确认 2 件事：
1. 你的电脑是 Mac 还是 Windows？
2. 你未来会让 Agent 连接 1 个飞书，还是多个飞书？
   - 只连接 1 个：配置更简单，不需要 profile。
   - 现在或以后会连接多个：需要给每个飞书设置 profile，用来区分不同飞书。

可以直接回复：`Mac + 1个`、`Windows + 多个`，或者告诉我你不确定。
回复后我从第 1 步开始。
```

每一步结尾固定加一句：

```text
如果这一步需要你替换命令内容，先把改好的命令发给我确认，不要发 App Secret。我确认后你再执行。执行后把结果发给我；成功发结果，失败请发终端截图或完整报错。确认这一步完成后，我再带你做下一步。
```

## 基本规则

- 飞书CLI官方引导：https://www.feishu.cn/content/article/7623291503305083853
- 单飞书员工：按官方式默认 profile 配置，不使用代称。
- 多飞书员工：每个飞书一个 profile，用 `PROFILE_NAME_HERE` 占位符，实际填团队约定的 profile 名称。
- profile 名称建议只用小写英文、数字、短横线或下划线，例如 `company_a`、`work`、`personal`。
- 占位符要整段替换。例如把 `APP_ID_HERE` 替换为 `cli_xxxxxxxxxxxxxxxx`，不要把 `APP_ID_HERE` 留在命令里。
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
printf '%s\n' "$APP_SECRET" | lark-cli config init --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

复制下面命令到 PowerShell 执行后，终端会提示输入 `App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```powershell
$AppSecret = Read-Host -Prompt "App Secret" -AsSecureString
$Bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($AppSecret)
try {
  $PlainSecret = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
  $PlainSecret | lark-cli config init --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
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
SCOPES_A='PASTE_SCOPE_GROUP_A_HERE'
lark-cli auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
SCOPES_B='PASTE_SCOPE_GROUP_B_HERE'
lark-cli auth login --scope "$SCOPES_B"
```

Windows PowerShell：使用同一份 scope 字符串，把 `SCOPES_A='PASTE_SCOPE_GROUP_A_HERE'` 改成 `$SCOPES_A = 'PASTE_SCOPE_GROUP_A_HERE'`，命令改成 `lark-cli auth login --scope $SCOPES_A`。

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

只输出一份初始化模板。把 `PROFILE_NAME_HERE` 替换为团队约定的 profile 名称，把 `APP_ID_HERE` 替换为对应飞书的 App ID。
profile 名称建议只用小写英文、数字、短横线或下划线，例如 `company_a`。

macOS Terminal（zsh）：

复制下面命令到终端执行后，终端会提示输入 `PROFILE_NAME_HERE App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```zsh
read -s "APP_SECRET?PROFILE_NAME_HERE App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | lark-cli config init --name PROFILE_NAME_HERE --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
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

复制下面命令到 PowerShell 执行后，终端会提示输入 `PROFILE_NAME_HERE App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```powershell
$AppSecret = Read-Host -Prompt "PROFILE_NAME_HERE App Secret" -AsSecureString
$Bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($AppSecret)
try {
  $PlainSecret = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
  $PlainSecret | lark-cli config init --name PROFILE_NAME_HERE --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
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
SCOPES_A='PASTE_SCOPE_GROUP_A_HERE'
lark-cli --profile PROFILE_NAME_HERE auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
SCOPES_B='PASTE_SCOPE_GROUP_B_HERE'
lark-cli --profile PROFILE_NAME_HERE auth login --scope "$SCOPES_B"
```

Windows PowerShell：使用同一份 scope 字符串，把 `SCOPES_A='PASTE_SCOPE_GROUP_A_HERE'` 改成 `$SCOPES_A = 'PASTE_SCOPE_GROUP_A_HERE'`，命令改成 `lark-cli --profile PROFILE_NAME_HERE auth login --scope $SCOPES_A`。

### 步骤 6：验证

官方基础验证命令是 `auth status`；这里使用 `--verify` 做更完整检查。

```shell
lark-cli --profile PROFILE_NAME_HERE auth status --verify
```

## 可选验收案例：创建测试日程

授权验证成功后，只有在用户明确同意时，才执行这个验收案例。先告诉用户：这会真实创建一个半小时日程，标题为 `CLI 验证日程`。

执行规则：

- 使用用户当前本地时区，动态计算“明天”的日期；不要硬编码示例日期。
- 测试时间固定为明天 09:00-09:30。
- 默认只给当前用户自己创建日程，不邀请其他人。
- 先查忙闲；如果 09:00-09:30 有冲突，先告诉用户冲突，不要继续创建。
- 如果忙闲为空，再创建日程，并返回 `event_id`。
- 单飞书命令不带 `--profile`；多飞书命令必须带 `--profile PROFILE_NAME_HERE`。

单飞书命令模板：

```shell
lark-cli calendar +freebusy --start "YYYY-MM-DDT09:00:00+08:00" --end "YYYY-MM-DDT09:30:00+08:00" --as user --format json
lark-cli calendar +create --summary "CLI 验证日程" --start "YYYY-MM-DDT09:00:00+08:00" --end "YYYY-MM-DDT09:30:00+08:00" --as user --format json
```

多飞书命令模板：

```shell
lark-cli --profile PROFILE_NAME_HERE calendar +freebusy --start "YYYY-MM-DDT09:00:00+08:00" --end "YYYY-MM-DDT09:30:00+08:00" --as user --format json
lark-cli --profile PROFILE_NAME_HERE calendar +create --summary "CLI 验证日程" --start "YYYY-MM-DDT09:00:00+08:00" --end "YYYY-MM-DDT09:30:00+08:00" --as user --format json
```

## 权限列表转命令提示词

用户拿到管理员发的权限列表后，可以把下面提示词发给 AI，让 AI 整理成可直接复制执行的授权命令。

```text
请把下面飞书 CLI scope 权限列表整理成可以直接复制执行的授权命令。

我的场景：
- 终端：Mac Terminal（zsh）/ Windows PowerShell
- 是否多飞书：单飞书 / 多飞书
- profile 名称：PROFILE_NAME_HERE（单飞书留空；多飞书必填）

要求：
1. 只生成 `lark-cli auth login --scope` 命令，不要使用 `--recommend`、`--no-wait`、`--json`。
2. 如果权限数据包含 `scopes.user` 和 `scopes.tenant`，只使用 `scopes.user`。
3. 如果不是 JSON，按管理员标注为“用户授权”的 scope 处理；不确定时先提醒我确认。
4. 去重并保留原 scope 字符串，不翻译、不改名。
5. 如果 scope 超过 120 个，拆成 Scope A/B/C 多组，每组一条命令。
6. 单飞书命令不要带 `--profile`；多飞书命令必须带 `--profile PROFILE_NAME_HERE`。
7. 按我填写的终端只输出一种命令格式。
8. 只输出命令块和必要标题，不要解释。

权限列表：
PASTE_ADMIN_SCOPE_LIST_HERE
```
