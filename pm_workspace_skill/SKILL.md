---
name: pm_workspace_skill
description: 产品经理工作流的共享底座。定义本地工作区目录结构、Markdown + frontmatter 文件格式、七个维度（信息/决策/待办/需求/缺陷/版本/项目）的模板、ID 规则和状态流转。负责初始化工作区和 workspace.yaml 配置。其他 pm_* Skill 读写数据前必须先按本 Skill 的约定定位工作区。
---

# PM 工作区底座

## 定位

本 Skill 是 `pm_capture_skill`、`pm_track_skill`、`pm_brief_skill` 的共享约定。它只负责两件事：

1. 初始化工作区（首次使用时）。
2. 定义所有数据文件的目录、格式、ID 和状态规则（供其他 Skill 引用）。

核心原则：**本地 Markdown 文件是唯一可信源**。飞书（多维表格、文档、飞书项目、飞书任务）只是信息输入源；所有产出（包括简报）只写本地文件，由用户自己决定是否手动分发。

## 输出规则

- 首次使用时先检查工作区是否存在（找 `workspace.yaml`），不存在则走「初始化流程」。
- 初始化是分步引导：先确认工作区路径和基本信息，用户确认后再创建目录和配置。
- 严禁未经用户确认就在用户磁盘上创建目录。
- 工作区路径优先级：用户明确指定 > `~/pm_workspace`（默认建议）。
- 初始化完成后建议用户把工作区初始化为 git 仓库（`git init`），用于版本追溯和简报取数；用户拒绝也可以正常使用。
- 不要把任何飞书 App Secret、token 写进工作区文件。

## 初始化流程

### 步骤 1：确认基本信息

首次回复模板：

```text
我来帮你初始化 PM 工作区。先确认几件事：
1. 工作区放在哪里？默认建议 ~/pm_workspace，也可以指定其他路径。
2. 你的名字（用于简报署名和待办 owner）？
3. 你的老板称呼、团队名称、下属名单（可以先留空，之后在 workspace.yaml 里补）。
4. 当前在管的项目列表（可以先填 1~2 个主要项目）。
```

### 步骤 2：创建目录结构

```text
pm_workspace/
├── workspace.yaml          # 工作区配置（人员、项目、飞书数据源、周期锚点）
├── 00_inbox/               # 收集箱：未分流的原始信息
├── 10_info/                # 信息收集：已分流的情报、会议纪要、竞品信息
├── 20_decisions/           # 决策记录
├── 30_todos/               # 待办
├── 40_requirements/        # 需求
├── 50_bugs/                # 缺陷
├── 60_releases/            # 版本
├── 70_projects/            # 项目（每个项目一个文件，含里程碑和风险）
└── 90_briefs/              # 简报产出
    ├── daily/
    ├── weekly/
    ├── biweekly/
    ├── monthly/
    └── quarterly/
```

### 步骤 3：生成 workspace.yaml

按用户回答填充，留空项保留注释：

```yaml
owner: ""                # 你的名字
boss: ""                 # 老板称呼
team: []                 # 团队成员
reports: []              # 下属名单
projects: []             # 在管项目，与 70_projects/ 文件一一对应
timezone: Asia/Shanghai
week_start: monday
biweekly_anchor: ""      # 双周报锚点周一，如 2026-01-05；第一次生成双周报时确定
feishu:
  enabled: true
  profile: ""            # 多飞书时填 lark-cli profile 名称；单飞书留空
  sources:               # pm_capture_skill 的取数来源，按需登记
    bitable: []          # 多维表格：{name, app_token, table_id, 用途}
    docs: []             # 飞书文档：{name, url, 用途}
    project_spaces: []   # 飞书项目空间：{name, 用途}
```

### 步骤 4：验证

列出创建的目录树给用户确认，提示后续用法：

```text
工作区已就绪。后续：
- 收集信息和分流：使用 $pm_capture_skill
- 管理待办/需求/缺陷/版本/项目：使用 $pm_track_skill
- 生成日报/周报/双周报/月报/季报：使用 $pm_brief_skill
```

## 数据文件约定（其他 Skill 必须遵守）

### 文件格式

每条记录一个 Markdown 文件，文件名 `{ID}_{短标题}.md`，frontmatter 必须包含：

```yaml
---
id: REQ-20260612-001
type: requirement        # info | decision | todo | requirement | bug | release | project
title: 标题
status: draft            # 见各维度状态流
priority: P1             # P0 | P1 | P2 | P3
owner: ""                # 负责人
project: ""              # 所属项目，对应 workspace.yaml projects
created: 2026-06-12
updated: 2026-06-12      # 每次修改必须更新
due: ""                  # 截止日期，可空
source: manual           # feishu_bitable | feishu_doc | feishu_project | feishu_task | chat | manual
links: []                # 相关飞书链接、文件路径
tags: []
---
```

frontmatter 之下是正文，结构由各维度模板决定。

### ID 规则

`{前缀}-{YYYYMMDD}-{当日三位序号}`，前缀与目录对应：

| 维度 | 前缀 | 目录 |
|---|---|---|
| 信息 | INF | 10_info |
| 决策 | DEC | 20_decisions |
| 待办 | TODO | 30_todos |
| 需求 | REQ | 40_requirements |
| 缺陷 | BUG | 50_bugs |
| 版本 | REL | 60_releases |
| 项目 | PRJ | 70_projects |

分配新 ID 前必须先用 Glob 查当日已有最大序号，避免重复。

### 状态流

- info：`unprocessed → processed → archived`
- decision：`proposed → decided → revisited`（决策正文必须含：背景、备选方案、最终决定、决策人、复盘日期）
- todo：`open → in_progress → blocked → done`，终态另有 `cancelled`
- requirement：`draft → reviewing → approved → in_dev → testing → released`，终态另有 `rejected`
- bug：`open → confirmed → fixing → verifying → closed`，终态另有 `wontfix`
- release：`planning → developing → testing → released`
- project：`active → on_hold → done`

状态只能沿流转方向推进或回退一步；跨级变更时 Skill 必须向用户确认。

### 修改纪律

- 任何 Skill 修改记录文件时，必须同步更新 frontmatter 的 `updated` 字段——简报取数依赖它。
- 状态变为终态（done/released/closed 等）的文件不移动、不删除，留在原目录由简报统计。
- 工作区是 git 仓库时，建议每次工作会话结束做一次 commit，message 概括本次变更。
