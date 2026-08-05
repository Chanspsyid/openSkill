# openSkill

公开 Skill 仓库。后续可复用、可分享的 Skill 都放在这里。

## 当前 Skill

- `feishu_installCli_skill`：安装飞书 CLI，并按单飞书 / 多飞书场景完成初始化、scope 授权和验证。

### PM 工作流套件

产品经理工作流 Skill 套件。本地 Markdown 文件是唯一可信源，飞书作为信息输入源和（确认后的）输出渠道；所有对外内容先生成本人草稿审阅，确认定稿后才派生受众版本、才允许发送：

- `pm_workspace_skill`：共享底座。初始化本地工作区，定义信息/决策/待办/需求/缺陷/版本/项目七个维度的目录、文件格式、ID、状态流和重点方向（focus）配置。
- `pm_capture_skill`：信息收集与分流。把飞书（多维表格、文档、飞书项目、飞书任务）和粘贴内容收进 inbox，再分流到七个维度。依赖 `feishu_installCli_skill` 装好的 lark-cli。
- `pm_track_skill`：日常事项管理。增删改查、状态推进、巡检、站会视图、全盘清单（DASHBOARD.md 个人数据库总览）；建议必出四类：优先级排序、风险预警、沟通建议、焦点对齐。
- `pm_brief_skill`：简报生成。按天/周/双周/月/季度聚合数据，先本人草稿，确认后派生老板版（工作量快照 + 核心决策与业务影响 + 给老板的参与点）、团队版、下属版。
- `pm_delegate_skill`：工作量盘点与委派分析。量化工作量给老板看，四象限分析哪些事能分出去，生成招人论证材料。
- `pm_send_skill`：飞书发送。逐条确认接收人和内容后，把定稿简报/材料以飞书文档或消息发出，绝不自动发送。

推荐使用顺序：`feishu_installCli_skill`（一次性）→ `pm_workspace_skill`（一次性）→ 日常 `pm_capture_skill` + `pm_track_skill` → 周期 `pm_brief_skill` + `pm_send_skill` → 按需 `pm_delegate_skill`。

## 仓库结构

```text
openSkill/
├── feishu_installCli_skill/
│   ├── SKILL.md
│   └── agents/
│       └── openai.yaml
├── pm_workspace_skill/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── pm_capture_skill/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── pm_track_skill/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── pm_brief_skill/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── pm_delegate_skill/
│   ├── SKILL.md
│   └── agents/openai.yaml
├── pm_send_skill/
│   ├── SKILL.md
│   └── agents/openai.yaml
└── README.md
```

## 给别人使用

把 GitHub 仓库地址发给对方，并告诉对方执行：

```shell
npx -y skills add https://github.com/Chanspsyid/openSkill -g --skill feishu_installCli_skill --agent '*' -y
```

如果对方使用的工具要求本地安装，也可以先克隆仓库：

```shell
git clone https://github.com/Chanspsyid/openSkill.git
cd openSkill
npx -y skills add . -g --skill feishu_installCli_skill --agent '*' -y
```

安装后，让对方在 Agent 里使用：

```text
使用 $feishu_installCli_skill 输出飞书 CLI 安装和授权命令
```

## 将管理员 scope 转成命令

拿到管理员发的权限列表后，可以把 `feishu_installCli_skill/SKILL.md` 里的“权限列表转命令提示词”复制给 AI，让 AI 生成可直接执行的 `auth login --scope` 命令。

## 注意

- Skill 里的 `APP_ID_HERE`、`PROFILE_NAME_HERE`、`PASTE_SCOPE_GROUP_A_HERE` 等占位符需要按自己的飞书应用替换。
- 如果其他资料里出现用尖括号包住 APP ID 的写法，左右尖括号也是占位符的一部分，替换后不要保留。
- 员工替换命令后，建议先发给 Agent 确认，再执行；不要把 App Secret 发给 Agent。
- App Secret 不要发到聊天、文档或 GitHub。
- 内部标准 scope 清单不要提交到公开仓库。
