---
name: feishu_installCli_skill
description: 输出飞书/Lark CLI 安装、初始化、scope 授权和验证命令。单飞书员工使用官方式默认 profile，不使用代称；多飞书员工使用 profile 代称，其中 mt=雪球飞书、ht=核桃飞书。授权命令只使用 --scope；内置雪球飞书 mt 的默认 Scope A/B 命令，核桃飞书 ht 的 scope 以核桃邮件为准。
---

# 安装飞书 CLI

## 基本规则

- 飞书CLI官方引导：https://www.feishu.cn/content/article/7623291503305083853
- 飞书CLI最佳实践：https://bytedance.larkoffice.com/wiki/ILuTww7Xcimb6GkhH0mcK2f4nS7
- `mt` = 雪球飞书；`ht` = 核桃飞书。
- 单飞书员工：按官方式默认 profile 配置，不使用 `mt` / `ht` 代称。
- 多飞书员工：每个飞书一个 profile，用 `<PROFILE>` 占位符，实际填 `mt` 或 `ht`。
- 开始前确认本机能使用 `npm` 和 `npx`；如果不能用，先安装 Node.js。
- App Secret 只能在本机终端隐藏输入；不要写进聊天、文档、命令历史或 Skill。
- 授权命令只使用 `auth login --scope`。

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

如果单飞书是雪球飞书，执行以下 A/B 两组授权命令。如果不是雪球飞书，使用管理员提供的 scope。

Scope A：

```zsh
SCOPES_A='admin:app.info:readonly contact:department.base:readonly contact:department.hrbp:readonly contact:department.organize:readonly contact:job_title:readonly contact:user.assign_info:read contact:user.base:readonly contact:user.basic_profile:readonly contact:user.department:readonly contact:user.department_path:readonly contact:user.dotted_line_leader_info.read contact:user.email:readonly contact:user.employee:readonly contact:user.employee_id:readonly contact:user.employee_number:read contact:user.gender:readonly contact:user.id:readonly contact:user.job_family:readonly contact:user.user_geo contact:user:search contact:work_city:readonly calendar:calendar calendar:calendar.acl:create calendar:calendar.acl:delete calendar:calendar.acl:read calendar:calendar.event:create calendar:calendar.event:delete calendar:calendar.event:read calendar:calendar.event:reply calendar:calendar.event:update calendar:calendar.free_busy:read calendar:calendar:create calendar:calendar:delete calendar:calendar:read calendar:calendar:readonly calendar:calendar:subscribe calendar:calendar:update calendar:exchange.bindings:create calendar:exchange.bindings:delete calendar:exchange.bindings:read calendar:room:readonly calendar:settings.caldav:create calendar:settings.workhour:read calendar:time_off:create calendar:time_off:delete base:app:copy base:app:create base:app:read base:app:update base:collaborator:create base:collaborator:delete base:collaborator:read base:dashboard:copy base:dashboard:create base:dashboard:delete base:dashboard:read base:dashboard:update base:field:create base:field:delete base:field:read base:field:update base:field_group:create base:form:create base:form:delete base:form:read base:form:update base:history:read base:record:create base:record:delete base:record:read base:record:retrieve base:record:update base:role:create base:role:delete base:role:read base:role:update base:table:create base:table:delete base:table:read base:table:update base:view:read base:view:write_only base:workflow:create base:workflow:delete base:workflow:read base:workflow:update base:workflow:write base:workspace:list bitable:app:readonly docs:doc:readonly docs:document.comment:create docs:document.comment:delete docs:document.comment:read docs:document.comment:update docs:document.comment:write_only docs:document.content:read docs:document.media:download docs:document.media:upload docs:document.subscription docs:document.subscription:read docs:document:copy docs:document:export docs:document:import docs:event.document_deleted:read docs:event.document_edited:read docs:event.document_opened:read docs:event:subscribe docs:permission.member docs:permission.member:apply docs:permission.member:auth docs:permission.member:create docs:permission.member:delete docs:permission.member:readonly docs:permission.member:retrieve docs:permission.member:transfer docs:permission.setting:read docs:permission.setting:readonly docs:permission.setting:write_only docx:document docx:document.block:convert docx:document:create docx:document:readonly docx:document:write_only drive:drive.metadata:readonly drive:drive.search:readonly drive:drive:readonly drive:drive:version:readonly drive:export:readonly drive:file.like:readonly drive:file.meta.sec_label.read_only drive:file:download drive:file:readonly drive:file:upload drive:file:view_record:readonly sheets:spreadsheet sheets:spreadsheet.meta:read sheets:spreadsheet.meta:write_only sheets:spreadsheet:create sheets:spreadsheet:read sheets:spreadsheet:readonly sheets:spreadsheet:write_only slides:presentation:create slides:presentation:read slides:presentation:update slides:presentation:write_only wiki:member:create wiki:member:retrieve wiki:member:update wiki:node:copy wiki:node:create wiki:node:move wiki:node:read wiki:node:retrieve wiki:node:update wiki:setting:read wiki:space:read wiki:space:retrieve wiki:space:write_only wiki:wiki:readonly space:document.event:read space:document:delete space:document:move space:document:retrieve space:document:shortcut space:folder:create'
lark-cli auth login --scope "$SCOPES_A"
```

Scope B：

```zsh
SCOPES_B='aily:data_asset:read aily:file:read aily:knowledge:read aily:message:read aily:run:read aily:session:read aily:skill:read approval:approval:readonly approval:instance:read approval:instance:write approval:task:read approval:task:write attendance:task:readonly baike:entity:readonly board:whiteboard:node:create board:whiteboard:node:delete board:whiteboard:node:read board:whiteboard:node:update cardkit:card:read cardkit:template:read collab_plugins:collab_plugins.relation.change:read component:selector component:user_profile document_ai:health_certificate:recognize document_ai:vehicle_invoice:recognize helpdesk:all:readonly im:chat.access_event.bot_p2p_chat:read im:chat.announcement:read im:chat.chat_pins:read im:chat.chat_pins:write_only im:chat.collab_plugins:read im:chat.members:read im:chat.members:write_only im:chat.moderation:read im:chat.tabs:read im:chat:create_by_user im:chat:read im:chat:readonly im:chat:update im:feed.flag:read im:feed.flag:write im:message im:message.group_msg:get_as_user im:message.p2p_msg:get_as_user im:message.pins:read im:message.pins:write_only im:message.reactions:read im:message.reactions:write_only im:message.send_as_user im:message.urgent.status:write im:message:readonly im:message:recall im:resource mail:event mail:public_mailbox:readonly mail:user_mailbox.event.mail_address:read mail:user_mailbox.folder:read mail:user_mailbox.folder:write mail:user_mailbox.mail_contact.mail_address:read mail:user_mailbox.mail_contact.phone:read mail:user_mailbox.mail_contact:read mail:user_mailbox.mail_contact:write mail:user_mailbox.message.address:read mail:user_mailbox.message.body:read mail:user_mailbox.message.subject:read mail:user_mailbox.message:modify mail:user_mailbox.message:readonly mail:user_mailbox.message:send mail:user_mailbox.rule:read mail:user_mailbox.rule:write mail:user_mailbox:readonly minutes:minutes.artifacts:read minutes:minutes.basic:read minutes:minutes.media:export minutes:minutes.search:read minutes:minutes.statistics:read minutes:minutes.transcript:export minutes:minutes.upload:write minutes:minutes:readonly offline_access okr:okr.content:readonly okr:okr.content:writeonly okr:okr.period:readonly okr:okr.progress.file:upload okr:okr.progress:delete okr:okr.progress:readonly okr:okr.progress:writeonly okr:okr.review:readonly okr:okr.setting:read okr:okr:readonly report:task:readonly search:app search:department:read search:docs:read search:memory_graph_tool_call:read search:message serviceaccount:approval:approvals:read spark:app.table.record:read spark:app.table:read spark:directory.user.id_convert:read task:attachment:read task:attachment:write task:comment task:comment:read task:comment:readonly task:comment:write task:custom_field:read task:custom_field:write task:section:read task:section:write task:task task:task:read task:task:readonly task:task:write task:task:writeonly task:tasklist:read task:tasklist:write tenant:tenant:readonly vc:meeting.meetingevent:read vc:meeting.meetingid:read vc:meeting.search:read vc:note:read vc:record vc:record:readonly vc:reserve vc:reserve:readonly vc:room:readonly'
lark-cli auth login --scope "$SCOPES_B"
```

Windows PowerShell：使用同一份 scope 字符串，把 `SCOPES_A='...'` 改成 `$SCOPES_A = '...'`，命令改成 `lark-cli auth login --scope $SCOPES_A`。

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

只输出一份初始化模板。把 `<PROFILE>` 替换为 `mt` 或 `ht`，把 `<APP_ID>` 替换为对应飞书的 App ID。

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

- 核桃飞书 `ht`：scope 以核桃邮件为准，不要套用雪球飞书的 scope。

雪球飞书 `mt` Scope A：

```zsh
SCOPES_A='admin:app.info:readonly contact:department.base:readonly contact:department.hrbp:readonly contact:department.organize:readonly contact:job_title:readonly contact:user.assign_info:read contact:user.base:readonly contact:user.basic_profile:readonly contact:user.department:readonly contact:user.department_path:readonly contact:user.dotted_line_leader_info.read contact:user.email:readonly contact:user.employee:readonly contact:user.employee_id:readonly contact:user.employee_number:read contact:user.gender:readonly contact:user.id:readonly contact:user.job_family:readonly contact:user.user_geo contact:user:search contact:work_city:readonly calendar:calendar calendar:calendar.acl:create calendar:calendar.acl:delete calendar:calendar.acl:read calendar:calendar.event:create calendar:calendar.event:delete calendar:calendar.event:read calendar:calendar.event:reply calendar:calendar.event:update calendar:calendar.free_busy:read calendar:calendar:create calendar:calendar:delete calendar:calendar:read calendar:calendar:readonly calendar:calendar:subscribe calendar:calendar:update calendar:exchange.bindings:create calendar:exchange.bindings:delete calendar:exchange.bindings:read calendar:room:readonly calendar:settings.caldav:create calendar:settings.workhour:read calendar:time_off:create calendar:time_off:delete base:app:copy base:app:create base:app:read base:app:update base:collaborator:create base:collaborator:delete base:collaborator:read base:dashboard:copy base:dashboard:create base:dashboard:delete base:dashboard:read base:dashboard:update base:field:create base:field:delete base:field:read base:field:update base:field_group:create base:form:create base:form:delete base:form:read base:form:update base:history:read base:record:create base:record:delete base:record:read base:record:retrieve base:record:update base:role:create base:role:delete base:role:read base:role:update base:table:create base:table:delete base:table:read base:table:update base:view:read base:view:write_only base:workflow:create base:workflow:delete base:workflow:read base:workflow:update base:workflow:write base:workspace:list bitable:app:readonly docs:doc:readonly docs:document.comment:create docs:document.comment:delete docs:document.comment:read docs:document.comment:update docs:document.comment:write_only docs:document.content:read docs:document.media:download docs:document.media:upload docs:document.subscription docs:document.subscription:read docs:document:copy docs:document:export docs:document:import docs:event.document_deleted:read docs:event.document_edited:read docs:event.document_opened:read docs:event:subscribe docs:permission.member docs:permission.member:apply docs:permission.member:auth docs:permission.member:create docs:permission.member:delete docs:permission.member:readonly docs:permission.member:retrieve docs:permission.member:transfer docs:permission.setting:read docs:permission.setting:readonly docs:permission.setting:write_only docx:document docx:document.block:convert docx:document:create docx:document:readonly docx:document:write_only drive:drive.metadata:readonly drive:drive.search:readonly drive:drive:readonly drive:drive:version:readonly drive:export:readonly drive:file.like:readonly drive:file.meta.sec_label.read_only drive:file:download drive:file:readonly drive:file:upload drive:file:view_record:readonly sheets:spreadsheet sheets:spreadsheet.meta:read sheets:spreadsheet.meta:write_only sheets:spreadsheet:create sheets:spreadsheet:read sheets:spreadsheet:readonly sheets:spreadsheet:write_only slides:presentation:create slides:presentation:read slides:presentation:update slides:presentation:write_only wiki:member:create wiki:member:retrieve wiki:member:update wiki:node:copy wiki:node:create wiki:node:move wiki:node:read wiki:node:retrieve wiki:node:update wiki:setting:read wiki:space:read wiki:space:retrieve wiki:space:write_only wiki:wiki:readonly space:document.event:read space:document:delete space:document:move space:document:retrieve space:document:shortcut space:folder:create'
lark-cli --profile mt auth login --scope "$SCOPES_A"
```

雪球飞书 `mt` Scope B：

```zsh
SCOPES_B='aily:data_asset:read aily:file:read aily:knowledge:read aily:message:read aily:run:read aily:session:read aily:skill:read approval:approval:readonly approval:instance:read approval:instance:write approval:task:read approval:task:write attendance:task:readonly baike:entity:readonly board:whiteboard:node:create board:whiteboard:node:delete board:whiteboard:node:read board:whiteboard:node:update cardkit:card:read cardkit:template:read collab_plugins:collab_plugins.relation.change:read component:selector component:user_profile document_ai:health_certificate:recognize document_ai:vehicle_invoice:recognize helpdesk:all:readonly im:chat.access_event.bot_p2p_chat:read im:chat.announcement:read im:chat.chat_pins:read im:chat.chat_pins:write_only im:chat.collab_plugins:read im:chat.members:read im:chat.members:write_only im:chat.moderation:read im:chat.tabs:read im:chat:create_by_user im:chat:read im:chat:readonly im:chat:update im:feed.flag:read im:feed.flag:write im:message im:message.group_msg:get_as_user im:message.p2p_msg:get_as_user im:message.pins:read im:message.pins:write_only im:message.reactions:read im:message.reactions:write_only im:message.send_as_user im:message.urgent.status:write im:message:readonly im:message:recall im:resource mail:event mail:public_mailbox:readonly mail:user_mailbox.event.mail_address:read mail:user_mailbox.folder:read mail:user_mailbox.folder:write mail:user_mailbox.mail_contact.mail_address:read mail:user_mailbox.mail_contact.phone:read mail:user_mailbox.mail_contact:read mail:user_mailbox.mail_contact:write mail:user_mailbox.message.address:read mail:user_mailbox.message.body:read mail:user_mailbox.message.subject:read mail:user_mailbox.message:modify mail:user_mailbox.message:readonly mail:user_mailbox.message:send mail:user_mailbox.rule:read mail:user_mailbox.rule:write mail:user_mailbox:readonly minutes:minutes.artifacts:read minutes:minutes.basic:read minutes:minutes.media:export minutes:minutes.search:read minutes:minutes.statistics:read minutes:minutes.transcript:export minutes:minutes.upload:write minutes:minutes:readonly offline_access okr:okr.content:readonly okr:okr.content:writeonly okr:okr.period:readonly okr:okr.progress.file:upload okr:okr.progress:delete okr:okr.progress:readonly okr:okr.progress:writeonly okr:okr.review:readonly okr:okr.setting:read okr:okr:readonly report:task:readonly search:app search:department:read search:docs:read search:memory_graph_tool_call:read search:message serviceaccount:approval:approvals:read spark:app.table.record:read spark:app.table:read spark:directory.user.id_convert:read task:attachment:read task:attachment:write task:comment task:comment:read task:comment:readonly task:comment:write task:custom_field:read task:custom_field:write task:section:read task:section:write task:task task:task:read task:task:readonly task:task:write task:task:writeonly task:tasklist:read task:tasklist:write tenant:tenant:readonly vc:meeting.meetingevent:read vc:meeting.meetingid:read vc:meeting.search:read vc:note:read vc:record vc:record:readonly vc:reserve vc:reserve:readonly vc:room:readonly'
lark-cli --profile mt auth login --scope "$SCOPES_B"
```

Windows PowerShell：使用同一份 scope 字符串，把 `SCOPES_A='...'` 改成 `$SCOPES_A = '...'`，命令改成 `lark-cli --profile mt auth login --scope $SCOPES_A`。

### 步骤 6：验证

官方基础验证命令是 `auth status`；这里使用 `--verify` 做更完整检查。

```shell
lark-cli --profile mt auth status --verify
lark-cli --profile ht auth status --verify
```
