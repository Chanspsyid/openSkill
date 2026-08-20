---
name: feishu_installCli_skill
description: 分步引导安装飞书/Lark CLI、检查本机依赖和已有 CLI/profile 配置、初始化配置、scope 授权和验证。单飞书员工使用官方式默认 profile，不使用代称；多飞书员工使用自定义 profile 名称。授权命令只使用 --scope；支持按管理员提供的 Scope A/B 分组授权、更新已有 scope，以及中国境内网络的 npmmirror/Gitee 镜像分流。
---

# 安装飞书 CLI

## 输出规则

- 默认工作方式是分步引导用户完成安装和配置。
- 严禁一次性输出整套安装文档、完整流程或所有命令。
- 即使用户说“把命令都给我”，也只能输出当前步骤命令，并提示完成后继续下一步。
- 首次回复必须先询问设备类型和未来要连接的飞书数量；在用户回答前不要输出安装、初始化或授权命令。
- 询问飞书数量时，要说明：只连接 1 个飞书会更简单；如果现在或未来会连接多个飞书，需要使用 profile 区分。
- 如果用户已经主动说明设备类型或未来连接的飞书数量，只询问缺失项。
- 用户回答设备和飞书数量后，先做“启动检查与分流”，检查本机依赖、是否已安装 `lark-cli`、以及已有 profile；不要直接进入安装命令。
- 如果已安装 `lark-cli`，先呈现当前 CLI 版本和已有 profile 的 `name`、`appId`、`user`、`tokenStatus`，能查到应用名称时也展示应用名称；再让用户选择：无需重复安装、配置另一个飞书、或更新某个飞书的 scope。
- 每次只输出当前步骤的命令和一句操作说明；步骤结束必须引导用户把执行结果告诉 Agent，无论成功或失败。
- 在确认用户已完成当前步骤目标前，不能进入下一步；如果用户贴出错误，先解释错误并继续处理当前步骤。
- 如果用户执行报错，停留在当前步骤，要求用户发送终端截图或完整文本报错；Agent 负责根据截图或报错继续排查，直到当前步骤成功后再进入下一步。
- 如果 Agent 具备本机命令执行能力，优先由 Agent 执行安装、检查、验证命令；如果不能执行，再让用户打开终端复制命令。
- 如果用户设备是 Windows，输出给 PowerShell 的所有飞书 CLI 命令都必须解析并调用 `lark-cli.cmd` 的绝对路径；只有 Mac 命令才查找 `lark-cli`。
- 如果 `npm install -g @larksuite/cli` 成功但 `lark-cli` / `lark-cli.cmd` 找不到，先进入“CLI 路径与 PATH 自修复”，不要让用户重复安装。
- 修复 PATH 时必须同时做到：当前终端可继续执行（设置临时 PATH 或使用绝对路径）和未来新终端可用（幂等写入 shell profile 或用户级 Path）。如果无法持久写入，仍要使用解析出的 CLI 绝对路径继续当前流程。
- 如果用户在中国境内，或安装命令出现下载超时、网络连接失败、GitHub/Gitee 无法访问，先进入“境内网络与镜像处理”，不要反复重试原命令。
- 需要浏览器授权时，给用户授权链接并等待用户完成。
- 不要让用户把 App Secret 发到聊天里；只给本机隐藏输入方式。
- 用户手动替换命令占位符后，先让用户把修改后的命令发给 Agent 确认；Agent 检查无误后，再让用户执行。
- Agent 检查修改后的命令时，至少确认：没有 `APP_ID_HERE` / `PROFILE_NAME_HERE` / `PASTE_SCOPE_GROUP_A_HERE` 等占位符，没有左尖括号或右尖括号，App ID 以 `cli_` 开头，多飞书命令使用 `--profile PROFILE_NAME` 而不是 `-- PROFILE_NAME`。
- 检查命令时不得要求用户发送 App Secret；初始化命令只包含 App ID 和 profile，App Secret 由终端隐藏输入。
- scope 授权一次只输出一组命令；A 组完成后再输出 B 组。
- 输出给员工的命令使用 `APP_ID_HERE`、`PROFILE_NAME_HERE` 这类占位符，避免使用尖括号包住 APP_ID 这类写法。
- 如果用户从其他资料看到尖括号包住的内容，必须提醒：左右尖括号也是占位符的一部分，替换后命令里不能留下尖括号。
- 每完成一个飞书的初始化和授权后，必须先查看当前安装情况，再决定继续配置下一个飞书或结束。
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
回复后我会先检查电脑环境和已有飞书 CLI 配置，确认是否需要安装。
```

每一步结尾固定加一句：

```text
如果这一步需要你替换命令内容，先把改好的命令发给我确认，不要发 App Secret。我确认后你再执行。执行后把结果发给我；成功发结果，失败请发终端截图或完整报错。确认这一步完成后，我再带你做下一步。
```

## 基本规则

- 飞书CLI官方引导：https://www.feishu.cn/content/article/7623291503305083853
- 单飞书员工：按官方式默认 profile 配置，不使用代称。
- 多飞书员工：每个飞书一个 profile，用 `PROFILE_NAME_HERE` 占位符，实际填团队约定的 profile 名称。
- `lark-cli` 软件本身只需要安装一次；连接多个飞书时不是重复安装 CLI，而是新增不同 profile。
- profile 名称建议只用小写英文、数字、短横线或下划线，例如 `company_a`、`work`、`personal`。
- 占位符要整段替换。例如把 `APP_ID_HERE` 替换为 `cli_xxxxxxxxxxxxxxxx`，不要把 `APP_ID_HERE` 留在命令里。
- 开始前确认本机能使用 `npm` 和 `npx`；如果不能用，先安装 Node.js。
- 同时确认本机已安装 `git`（用 `git --version` 验证）；从 GitHub 安装 Skill 依赖 `git clone`，没有 git 会失败。Windows 默认不带 git，需单独安装 Git for Windows。
- 中国境内员工如果 npm/npx 下载慢、超时或失败，先把 npm registry 设置为 `https://registry.npmmirror.com`。
- 中国境内员工优先使用已同步的 Gitee 镜像仓库 `https://gitee.com/wenider/open-skill.git` 安装 Skill；不要推荐员工使用来历不明的公共 GitHub 代理地址。
- Windows PowerShell 中调用飞书 CLI 时优先解析 `lark-cli.cmd` 的绝对路径，不要使用 `lark-cli`；这样可以避开公司电脑常见的 PowerShell 脚本执行策略限制。
- 安装后如果 PATH 未生效，不要把它判定为安装失败；先定位 npm 全局 bin 目录，能找到 CLI 绝对路径就继续配置。
- Agent 在同一轮流程中如果解析到 `LARK_CLI_BIN`（Mac）或 `$LarkCliBin`（Windows），后续所有 CLI 命令都优先替换为该绝对路径；如果换了新终端或新进程，先重新执行“CLI 命令前置解析”。
- App Secret 只能在本机终端隐藏输入；不要写进聊天、文档、命令历史或 Skill。
- 授权命令只使用 `auth login --scope`。
- 公开仓库只保留 scope 占位符；内部权限清单由管理员私下提供。
- 员工不需要在飞书开发者后台新建应用或提交新的 CLI 应用申请；App ID、App Secret 和 scope 应由管理员从已有 CLI 应用中提供。
- 如果用户在浏览器或后台看到“创建应用”“申请应用”“开发者后台配置”等页面，先停止操作，让用户发截图；不要引导员工提交新应用申请。

## 启动检查与分流

### 步骤 1：确认执行方式

```text
如果 Agent 能执行本机命令：让 Agent 直接执行检查命令。
如果 Agent 不能执行本机命令：Mac 打开 Terminal（终端）；Windows 打开 PowerShell，不要用 Command Prompt。
```

### 步骤 2：检查前置环境和现有 CLI

Mac / Windows 都按当前终端执行等价检查；一次只输出当前设备对应命令。

Mac Terminal（zsh）：

```zsh
node --version
npm --version
npx --version
git --version
npm config get registry
npm config get prefix
LARK_CLI_BIN="$(command -v lark-cli || true)"
NPM_GLOBAL_BIN="$(npm config get prefix)/bin"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
printf 'NPM_GLOBAL_BIN=%s\nLARK_CLI_BIN=%s\n' "$NPM_GLOBAL_BIN" "${LARK_CLI_BIN:-}"
if [ -n "$LARK_CLI_BIN" ]; then
  "$LARK_CLI_BIN" --version
  "$LARK_CLI_BIN" profile list
else
  echo "CLI_NOT_FOUND"
fi
```

Windows PowerShell：

```powershell
node --version
npm --version
npx --version
git --version
npm config get registry
npm config get prefix
$NpmGlobalBin = npm config get prefix
$LarkCliBin = (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
"NPM_GLOBAL_BIN=$NpmGlobalBin"
"LARK_CLI_BIN=$LarkCliBin"
if ($LarkCliBin) {
  & $LarkCliBin --version
  & $LarkCliBin profile list
} else {
  "CLI_NOT_FOUND"
}
```

Agent 需要判断：

- `node` / `npm` / `npx` 不可用：先让用户安装 Node.js，再回到本步骤。
- `git` 不可用：先让用户安装 Git，再回到本步骤。
- `lark-cli` 不可用但 npm 全局 bin 中存在 CLI：进入“CLI 路径与 PATH 自修复”，修复后继续当前流程，不要重复安装。
- `lark-cli` 不可用且 npm 全局 bin 中也不存在 CLI：进入对应场景的“安装 CLI（仅未安装时）”。
- `lark-cli profile list` 已有配置：整理当前配置，再让用户选择下一步。
- `npm config get registry` 是官方 npm 源且用户在中国境内，或安装时出现网络失败：进入“境内网络与镜像处理”。
- Windows 报错 `PSSecurityException`、`UnauthorizedAccess`、`无法加载文件 ... lark-cli.ps1` 或提示系统禁止运行脚本：不要改执行策略，先执行“CLI 命令前置解析”，用解析出的 `& $Cli` 重试。
- Windows 报错 `lark-cli.cmd` 找不到、`not recognized`、`CommandNotFoundException`：先进入“CLI 路径与 PATH 自修复”，不要改执行策略，不要重复安装。

### 步骤 3：已有配置时让用户确认下一步

如果已安装 `lark-cli`，用简短列表呈现当前配置：

```text
检测到已安装 lark-cli：vX.X.X

当前已有飞书配置：
- profile/name: xxx；appId: cli_xxx；用户: xxx；状态: valid/needs_refresh/unknown；应用名称: xxx（如能查到）

请选择下一步：
a. 无需重复安装，直接使用现有配置
b. 配置另一个飞书（需要新的 profile 名称，建议小写英文）
c. 更新某个飞书的 scope 授权
```

分流规则：

- 选 `a`：如果状态是 `valid`，说明无需重复安装；可询问是否执行可选验收案例。若状态不是 `valid`，引导到验证或更新 scope。
- 选 `b`：多飞书场景要求用户提供新的 `profile 名称` 和 `App ID`，再进入多飞书“初始化配置”；单飞书场景如果要新增另一个飞书，先切换到多飞书流程。
- 选 `c`：先确认要更新哪个 profile；单飞书默认不带 `--profile`，多飞书必须带 `--profile PROFILE_NAME_HERE`，然后进入“授权”。

### 防止误申请新应用

当用户说“我是不是要申请应用”“页面让我新建应用”“管理员让我申请 CLI 应用”时，先澄清：

- 普通员工只需要使用管理员提供的已有 CLI 应用信息，不需要新建应用。
- 缺少 App ID、App Secret 或 scope 时，应找管理员补齐，不要自己创建应用。
- 管理员应发放一份资料包：飞书名称、建议 profile 名称、App ID、App Secret 的安全交付方式、Scope A/B/C、以及“不要新建应用，只做授权”的提醒。
- 如果确实要新增公司级 CLI 应用，应由飞书管理员统一创建、开权限和发布；不要让每个员工各自提交申请。

## CLI 路径与 PATH 自修复

触发条件：

- `npm install -g @larksuite/cli` 成功，但 `lark-cli --version` / `lark-cli.cmd --version` 报 command not found、not recognized、CommandNotFoundException。
- 启动检查中 npm 全局 bin 下存在 CLI 文件，但当前终端 PATH 没有包含该目录。
- 员工反复安装 CLI 后仍提示找不到命令。

处理原则：

- 先确认全局包是否已安装，再定位 npm 全局 bin；不要重复安装。
- 当前终端必须立即可用：Mac 设置 `LARK_CLI_BIN`，Windows 设置 `$LarkCliBin`，后续命令优先用绝对路径。
- 持久 PATH 修复必须幂等：Mac 使用 marker 写入 shell profile；Windows 只追加缺失的用户级 Path。
- 如果持久 PATH 写入失败，仍使用绝对路径继续初始化、授权和验证。

Mac Terminal（zsh）：

```zsh
npm list -g @larksuite/cli --depth=0
NPM_GLOBAL_BIN="$(npm config get prefix)/bin"
LARK_CLI_BIN="$(command -v lark-cli || true)"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then
  echo "未找到 lark-cli，请确认 npm install -g @larksuite/cli 是否成功。"
  exit 1
fi
case ":$PATH:" in *":$NPM_GLOBAL_BIN:"*) ;; *) export PATH="$NPM_GLOBAL_BIN:$PATH";; esac
START_MARKER="# >>> lark-cli npm global bin >>>"
END_MARKER="# <<< lark-cli npm global bin <<<"
for PROFILE_FILE in "${ZDOTDIR:-$HOME}/.zprofile" "${ZDOTDIR:-$HOME}/.zshrc"; do
  if [ -e "$PROFILE_FILE" ] || [ "$(basename "$PROFILE_FILE")" = ".zprofile" ]; then
    if grep -Fq "$START_MARKER" "$PROFILE_FILE" 2>/dev/null; then
      sed -i.bak "/$START_MARKER/,/$END_MARKER/d" "$PROFILE_FILE" && rm -f "$PROFILE_FILE.bak"
    fi
    {
      printf '\n%s\n' "$START_MARKER"
      printf 'export PATH="%s:$PATH"\n' "$NPM_GLOBAL_BIN"
      printf '%s\n' "$END_MARKER"
    } >> "$PROFILE_FILE"
  fi
done
"$LARK_CLI_BIN" --version
```

Windows PowerShell：

```powershell
npm list -g @larksuite/cli --depth=0
$NpmGlobalBin = npm config get prefix
$LarkCliBin = (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) {
  Write-Error "未找到 lark-cli.cmd，请确认 npm install -g @larksuite/cli 是否成功。"
  exit 1
}
$UserPath = [Environment]::GetEnvironmentVariable("Path", "User")
$NormGlobalBin = [IO.Path]::GetFullPath($NpmGlobalBin).TrimEnd("\")
$UserParts = @($UserPath -split ";" | Where-Object { $_ } | ForEach-Object {
  try { [IO.Path]::GetFullPath($_).TrimEnd("\") } catch { $_.TrimEnd("\") }
})
if ($UserParts -notcontains $NormGlobalBin) {
  $NewUserPath = if ([string]::IsNullOrWhiteSpace($UserPath)) { $NpmGlobalBin } else { "$($UserPath.TrimEnd(';'));$NpmGlobalBin" }
  [Environment]::SetEnvironmentVariable("Path", $NewUserPath, "User")
}
if (@($env:Path -split ";" | Where-Object { $_ }) -notcontains $NpmGlobalBin) { $env:Path = "$NpmGlobalBin;$env:Path" }
& $LarkCliBin --version
```

Agent 需要确认：

- `npm list -g @larksuite/cli --depth=0` 显示已安装，或虽然 `npm list` 失败但 npm 全局 bin 中能找到 CLI 文件；只有两者都不存在时才回到“安装 CLI（仅未安装时）”。
- `LARK_CLI_BIN` / `$LarkCliBin` 非空，且 `--version` 成功。
- 后续同一终端中所有命令都优先使用解析出的绝对路径，例如 `"$LARK_CLI_BIN" profile list` 或 `& $LarkCliBin profile list`。
- 如果后续步骤在新的终端执行，且 `LARK_CLI_BIN` / `$LarkCliBin` 变量不存在，先重新执行下面的“CLI 命令前置解析”片段，仍找不到时再回到本段自修复；不要先裸跑普通 `lark-cli` / `lark-cli.cmd`。
- PATH 修复完成后不要要求员工重开终端才能继续；可以提醒新终端也会生效。

### CLI 命令前置解析

后续所有初始化、授权、验证和验收命令都必须先解析 CLI 路径。命令块如果可能在新终端或新进程执行，必须内置对应系统的前置解析片段，不要依赖上一条命令遗留的 shell 变量。

Mac / Linux / Git Bash：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
```

## 境内网络与镜像处理

触发条件：

- 用户在中国境内，且 npm/npx 下载很慢或失败。
- 命令报错包含 `ETIMEDOUT`、`ECONNRESET`、`network timeout`、`fetch failed`、`Could not resolve host`。
- 安装 Skill 时无法访问 GitHub，或需要给中国境内员工提供更稳定的安装地址。

### 步骤 1：设置 npm 镜像

Mac Terminal 和 Windows PowerShell 使用同一组命令：

```shell
npm config set registry https://registry.npmmirror.com
npm config get registry
```

Agent 需要确认：

- 输出是 `https://registry.npmmirror.com` 或 `https://registry.npmmirror.com/`。
- 确认后回到原步骤，重新执行刚才失败的 npm/npx 命令。

如果用户后续要恢复官方 npm 源，再单独给这条命令：

```shell
npm config set registry https://registry.npmjs.org
```

### 步骤 2：GitHub 不通时使用 Gitee 镜像仓库

默认使用已同步的 Gitee 镜像仓库：

```shell
npx -y skills add https://gitee.com/wenider/open-skill.git -g --skill feishu_installCli_skill --agent '*' -y
```

如果公司已有内部 Git 服务，也可以使用内部镜像仓库；把下面命令里的 `SKILL_REPO_URL_HERE` 整段替换为内部仓库地址：

```shell
npx -y skills add SKILL_REPO_URL_HERE -g --skill feishu_installCli_skill --agent '*' -y
```

不要引导员工使用随机公共 GitHub 代理。

Agent 需要确认：

- 使用 Gitee 默认命令时，不需要替换仓库地址。
- 使用内部 Git 镜像时，`SKILL_REPO_URL_HERE` 已被完整替换。
- 镜像仓库里包含 `feishu_installCli_skill/SKILL.md`。
- 安装成功后，继续进入“启动检查与分流”。

## 场景 1：单飞书员工

### 步骤 1：确认执行方式

```text
如果 Agent 能执行本机命令：让 Agent 直接执行下一步命令。
如果 Agent 不能执行本机命令：Mac 打开 Terminal（终端）；Windows 打开 PowerShell，不要用 Command Prompt。
```

### 步骤 2：安装 CLI（仅未安装时）

只有“启动检查与分流”确认 `npm list -g @larksuite/cli --depth=0` 未安装，且 npm 全局 bin 中也没有 `lark-cli` / `lark-cli.cmd` 文件时，才执行安装。只是 `command not found` / `not recognized` 时先进入“CLI 路径与 PATH 自修复”，不要重复安装。
中国境内员工或 npm/npx 下载失败时，先进入“境内网络与镜像处理”的 npm 镜像步骤，再回到本步骤。

```shell
npm install -g @larksuite/cli
npx -y skills add https://open.feishu.cn --skill -y
```

安装命令成功后，必须立即执行“CLI 路径与 PATH 自修复”中的对应设备命令来验证 CLI。不要只用 `lark-cli --version` 判断安装成败。

### 步骤 3：初始化配置

macOS Terminal（zsh）：

复制下面命令到终端执行后，终端会提示输入 `App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
read -s "APP_SECRET?App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | "$LARK_CLI_BIN" config init --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

复制下面命令到 PowerShell 执行后，终端会提示输入 `App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$AppSecret = Read-Host -Prompt "App Secret" -AsSecureString
$Bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($AppSecret)
try {
  $PlainSecret = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
  $PlainSecret | & $Cli config init --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
} finally {
  if ($Bstr -ne [IntPtr]::Zero) { [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($Bstr) }
  Remove-Variable AppSecret, PlainSecret, Cli -ErrorAction SilentlyContinue
}
```

### 步骤 4：检查配置

macOS Terminal（zsh）：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
"$LARK_CLI_BIN" profile list
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
& $Cli profile list
```

### 步骤 5：授权

如果管理员提供的是 A/B 两组 scope，执行以下两组授权命令；如果管理员另有说明，以管理员提供的 scope 为准。

Scope A：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
SCOPES_A='PASTE_SCOPE_GROUP_A_HERE'
"$LARK_CLI_BIN" auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
SCOPES_B='PASTE_SCOPE_GROUP_B_HERE'
"$LARK_CLI_BIN" auth login --scope "$SCOPES_B"
```

Windows PowerShell Scope A：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$SCOPES_A = 'PASTE_SCOPE_GROUP_A_HERE'
& $Cli auth login --scope $SCOPES_A
```

Windows PowerShell Scope B：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$SCOPES_B = 'PASTE_SCOPE_GROUP_B_HERE'
& $Cli auth login --scope $SCOPES_B
```

### 步骤 6：验证

官方基础验证命令是 `auth status`；这里使用 `--verify` 做更完整检查。

macOS Terminal（zsh）：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
"$LARK_CLI_BIN" auth status --verify
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
& $Cli auth status --verify
```

### 步骤 7：查看当前安装情况

macOS Terminal（zsh）：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
"$LARK_CLI_BIN" profile list
"$LARK_CLI_BIN" auth status --verify
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
& $Cli profile list
& $Cli auth status --verify
```

Agent 需要确认：

- `profile list` 能看到当前飞书配置。
- `appId` 以 `cli_` 开头，且没有占位符或尖括号。
- `auth status --verify` 返回 `verified: true`，用户身份可用。
- 如果用户只连接 1 个飞书，说明安装完成；再询问是否需要执行可选验收案例。

## 场景 2：多飞书员工

### 步骤 1：确认执行方式

```text
如果 Agent 能执行本机命令：让 Agent 直接执行下一步命令。
如果 Agent 不能执行本机命令：Mac 打开 Terminal（终端）；Windows 打开 PowerShell，不要用 Command Prompt。
```

### 步骤 2：安装 CLI（仅未安装时）

只有“启动检查与分流”确认 `npm list -g @larksuite/cli --depth=0` 未安装，且 npm 全局 bin 中也没有 `lark-cli` / `lark-cli.cmd` 文件时，才执行安装。只是 `command not found` / `not recognized` 时先进入“CLI 路径与 PATH 自修复”，不要重复安装。
中国境内员工或 npm/npx 下载失败时，先进入“境内网络与镜像处理”的 npm 镜像步骤，再回到本步骤。

```shell
npm install -g @larksuite/cli
npx -y skills add https://open.feishu.cn --skill -y
```

安装命令成功后，必须立即执行“CLI 路径与 PATH 自修复”中的对应设备命令来验证 CLI。不要只用 `lark-cli --version` 判断安装成败。

### 步骤 3：初始化配置

只输出一份初始化模板。把 `PROFILE_NAME_HERE` 替换为团队约定的 profile 名称，把 `APP_ID_HERE` 替换为对应飞书的 App ID。
profile 名称建议只用小写英文、数字、短横线或下划线，例如 `company_a`。

macOS Terminal（zsh）：

复制下面命令到终端执行后，终端会提示输入 `PROFILE_NAME_HERE App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
read -s "APP_SECRET?PROFILE_NAME_HERE App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | "$LARK_CLI_BIN" config init --name PROFILE_NAME_HERE --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
unset APP_SECRET
```

macOS 示例，假设 profile 名称是 `company_a`，App ID 是 `cli_xxxxxxxxxxxxxxxx`：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
read -s "APP_SECRET?company_a App Secret: "
printf '\n'
printf '%s\n' "$APP_SECRET" | "$LARK_CLI_BIN" config init --name company_a --brand feishu --app-id "cli_xxxxxxxxxxxxxxxx" --app-secret-stdin
unset APP_SECRET
```

Windows PowerShell：

复制下面命令到 PowerShell 执行后，终端会提示输入 `PROFILE_NAME_HERE App Secret`；用户在终端输入或粘贴即可，输入内容不会显示在屏幕上。

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$AppSecret = Read-Host -Prompt "PROFILE_NAME_HERE App Secret" -AsSecureString
$Bstr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($AppSecret)
try {
  $PlainSecret = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($Bstr)
  $PlainSecret | & $Cli config init --name PROFILE_NAME_HERE --brand feishu --app-id "APP_ID_HERE" --app-secret-stdin
} finally {
  if ($Bstr -ne [IntPtr]::Zero) { [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($Bstr) }
  Remove-Variable AppSecret, PlainSecret, Cli -ErrorAction SilentlyContinue
}
```

### 步骤 4：检查配置

macOS Terminal（zsh）：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
"$LARK_CLI_BIN" profile list
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
& $Cli profile list
```

### 步骤 5：授权

- 每个飞书应用的 scope 以管理员提供的信息为准，不要混用其他应用的 scope。

Scope A：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
SCOPES_A='PASTE_SCOPE_GROUP_A_HERE'
"$LARK_CLI_BIN" --profile PROFILE_NAME_HERE auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
SCOPES_B='PASTE_SCOPE_GROUP_B_HERE'
"$LARK_CLI_BIN" --profile PROFILE_NAME_HERE auth login --scope "$SCOPES_B"
```

Windows PowerShell Scope A：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$SCOPES_A = 'PASTE_SCOPE_GROUP_A_HERE'
& $Cli --profile PROFILE_NAME_HERE auth login --scope $SCOPES_A
```

Windows PowerShell Scope B：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$SCOPES_B = 'PASTE_SCOPE_GROUP_B_HERE'
& $Cli --profile PROFILE_NAME_HERE auth login --scope $SCOPES_B
```

### 步骤 6：验证

官方基础验证命令是 `auth status`；这里使用 `--verify` 做更完整检查。

macOS Terminal（zsh）：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
"$LARK_CLI_BIN" --profile PROFILE_NAME_HERE auth status --verify
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
& $Cli --profile PROFILE_NAME_HERE auth status --verify
```

### 步骤 7：查看当前安装情况

每完成一个飞书的授权后，都必须先执行当前安装情况检查。

macOS Terminal（zsh）：

```zsh
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
"$LARK_CLI_BIN" profile list
"$LARK_CLI_BIN" --profile PROFILE_NAME_HERE auth status --verify
```

Windows PowerShell：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
& $Cli profile list
& $Cli --profile PROFILE_NAME_HERE auth status --verify
```

Agent 需要确认：

- `profile list` 能看到刚配置的 profile。
- 当前 profile 的 `appId` 以 `cli_` 开头，且没有占位符或尖括号。
- 当前 profile 的 `auth status --verify` 返回 `verified: true`，用户身份可用。
- 用简短文字告诉用户当前已完成哪个 profile。
- 然后询问用户：继续配置下一个飞书，还是结束安装。
- 如果继续配置下一个飞书，回到本场景步骤 3，要求用户提供下一个 `profile 名称` 和 `App ID`。
- 如果用户说已全部完成，再询问是否需要执行可选验收案例。

## 可选验收案例：创建测试日程

授权验证成功后，只有在用户明确同意时，才执行这个验收案例。先告诉用户：这会真实创建一个半小时日程，标题为 `CLI 验证日程`。

执行规则：

- 使用用户当前本地时区，动态计算“明天”的日期；不要硬编码示例日期。
- 测试时间固定为明天 09:00-09:30。
- 默认只给当前用户自己创建日程，不邀请其他人。
- 执行前先确认已授权日历相关 scope，例如 `calendar:calendar.event:read`、`calendar:calendar.event:create`、`calendar:calendar.free_busy:read` 或覆盖它们的 `calendar:calendar`；如果缺少，先提示补授权，不要把验收失败误判为 CLI 配置失败。
- 先查忙闲；如果 09:00-09:30 有冲突，先告诉用户冲突，不要继续创建。
- 如果忙闲为空，再创建日程，并返回 `event_id`。
- 单飞书命令不带 `--profile`；多飞书命令必须带 `--profile PROFILE_NAME_HERE`。

单飞书命令模板（Mac / Linux / Git Bash）：

```shell
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
if date -v+1d +%Y-%m-%d >/dev/null 2>&1; then EVENT_DATE="$(date -v+1d +%Y-%m-%d)"; else EVENT_DATE="$(date -d tomorrow +%Y-%m-%d)"; fi
TZ_OFFSET="$(date +%z | sed -E 's/^([+-][0-9]{2})([0-9]{2})$/\1:\2/')"
START="${EVENT_DATE}T09:00:00${TZ_OFFSET}"
END="${EVENT_DATE}T09:30:00${TZ_OFFSET}"
"$LARK_CLI_BIN" calendar +freebusy --start "$START" --end "$END" --as user --format json
"$LARK_CLI_BIN" calendar +create --summary "CLI 验证日程" --start "$START" --end "$END" --as user --format json
```

单飞书命令模板（Windows PowerShell）：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$EventDate = (Get-Date).AddDays(1).ToString("yyyy-MM-dd")
$Offset = (Get-Date).ToString("zzz")
$Start = "${EventDate}T09:00:00${Offset}"
$End = "${EventDate}T09:30:00${Offset}"
& $Cli calendar +freebusy --start $Start --end $End --as user --format json
& $Cli calendar +create --summary "CLI 验证日程" --start $Start --end $End --as user --format json
```

多飞书命令模板（Mac / Linux / Git Bash）：

```shell
NPM_GLOBAL_BIN="${NPM_GLOBAL_BIN:-$(npm config get prefix)/bin}"
LARK_CLI_BIN="${LARK_CLI_BIN:-$(command -v lark-cli || true)}"
if [ -z "$LARK_CLI_BIN" ] && [ -x "$NPM_GLOBAL_BIN/lark-cli" ]; then LARK_CLI_BIN="$NPM_GLOBAL_BIN/lark-cli"; fi
if [ -z "$LARK_CLI_BIN" ]; then echo "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1; fi
if date -v+1d +%Y-%m-%d >/dev/null 2>&1; then EVENT_DATE="$(date -v+1d +%Y-%m-%d)"; else EVENT_DATE="$(date -d tomorrow +%Y-%m-%d)"; fi
TZ_OFFSET="$(date +%z | sed -E 's/^([+-][0-9]{2})([0-9]{2})$/\1:\2/')"
START="${EVENT_DATE}T09:00:00${TZ_OFFSET}"
END="${EVENT_DATE}T09:30:00${TZ_OFFSET}"
"$LARK_CLI_BIN" --profile PROFILE_NAME_HERE calendar +freebusy --start "$START" --end "$END" --as user --format json
"$LARK_CLI_BIN" --profile PROFILE_NAME_HERE calendar +create --summary "CLI 验证日程" --start "$START" --end "$END" --as user --format json
```

多飞书命令模板（Windows PowerShell）：

```powershell
$NpmGlobalBin = if ($NpmGlobalBin) { $NpmGlobalBin } else { npm config get prefix }
$LarkCliBin = if ($LarkCliBin) { $LarkCliBin } else { (Get-Command lark-cli.cmd -ErrorAction SilentlyContinue).Source }
if (-not $LarkCliBin -and (Test-Path (Join-Path $NpmGlobalBin "lark-cli.cmd"))) { $LarkCliBin = Join-Path $NpmGlobalBin "lark-cli.cmd" }
if (-not $LarkCliBin) { Write-Error "CLI_NOT_FOUND: 先执行 CLI 路径与 PATH 自修复"; exit 1 }
$Cli = $LarkCliBin
$EventDate = (Get-Date).AddDays(1).ToString("yyyy-MM-dd")
$Offset = (Get-Date).ToString("zzz")
$Start = "${EventDate}T09:00:00${Offset}"
$End = "${EventDate}T09:30:00${Offset}"
& $Cli --profile PROFILE_NAME_HERE calendar +freebusy --start $Start --end $End --as user --format json
& $Cli --profile PROFILE_NAME_HERE calendar +create --summary "CLI 验证日程" --start $Start --end $End --as user --format json
```

## 权限列表转授权命令

当用户拿到管理员发的权限列表、需要转成授权命令时，直接引导用户把权限列表原样贴出来，再按下面规则生成可直接复制执行的 `auth login --scope` 命令。终端类型和单 / 多飞书沿用本次会话已确认的信息，不必重复询问；多飞书使用本场景约定的 `PROFILE_NAME_HERE`。

如果用户还没贴出权限列表，先请用户把管理员给的 scope 列表（JSON 或纯文本均可）发出来，再生成命令。

生成规则：

1. 只生成 `auth login --scope` 相关命令块，不要使用 `--recommend`、`--no-wait`、`--json`；命令块必须内置“CLI 命令前置解析”，Mac 使用 `"$LARK_CLI_BIN"`，Windows PowerShell 使用 `& $Cli`。
2. 如果权限数据包含 `scopes.user` 和 `scopes.tenant`，只使用 `scopes.user`。
3. 如果不是 JSON，按管理员标注为“用户授权”的 scope 处理；不确定时先和用户确认。
4. 去重并保留原 scope 字符串，不翻译、不改名。
5. 如果 scope 超过 120 个，拆成 Scope A/B/C 多组，每组一条命令。
6. 单飞书命令不要带 `--profile`；多飞书命令必须带 `--profile PROFILE_NAME_HERE`。
7. 按用户本次使用的终端只输出一种命令格式。
8. 只输出命令块和必要标题，不要解释。
