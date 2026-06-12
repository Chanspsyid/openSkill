---
name: pm_capture_skill
description: PM 信息收集与分流。把飞书（多维表格、文档、飞书项目、飞书任务、聊天记录）和用户随手粘贴的内容收进工作区 00_inbox，再分流到信息/决策/待办/需求/缺陷/版本/项目七个维度。依赖 pm_workspace_skill 的工作区约定和 lark-cli（由 feishu_installCli_skill 安装授权）。
---

# PM 信息收集与分流

## 前置条件

- 工作区已按 `pm_workspace_skill` 初始化（能找到 `workspace.yaml`）；找不到时先引导用户初始化，不要自行猜测目录。
- 读取飞书数据需要 `lark-cli` 已安装并授权；未就绪时引导用户使用 `$feishu_installCli_skill`，本 Skill 不重复安装流程。
- `workspace.yaml` 里 `feishu.profile` 非空时，所有 lark-cli 命令必须带 `--profile`。

## 输出规则

- 两种工作模式：**收集**（信息进 inbox）和**分流**（inbox 清空到七个维度）。用户没说清楚时先问做哪个。
- 收集来源以 `workspace.yaml` 的 `feishu.sources` 登记项为准；用户提到新来源时，先帮用户把它登记进 `workspace.yaml` 再取数。
- 调用 lark-cli 前先用 `--help` 确认对应模块的实际子命令和参数，不要凭记忆拼命令；取数命令优先加 `--format json`。
- 飞书取数失败（权限不足、scope 缺失）时，提示用户用 `$feishu_installCli_skill` 的「权限列表转命令提示词」补授权，不要反复重试。
- 分流必须逐条向用户确认归类结果后再写文件；用户明确说「按你的判断批量处理」时才可以批量执行，执行后给出清单供复核。
- 写入维度文件时严格遵守 `pm_workspace_skill` 的 frontmatter、ID 和状态流约定；`source` 字段如实填写来源，`links` 保留原始飞书链接。
- 不要把飞书消息中的敏感内容（密码、密钥、薪酬等）写入工作区；遇到时提醒用户并跳过。

## 收集模式

支持的输入：

1. **用户直接粘贴**：聊天记录片段、会议要点、口头交代的事项。原样存入 inbox。
2. **飞书文档**：按 `feishu.sources.docs` 登记的文档，用 lark-cli 拉取内容或要点。
3. **飞书多维表格**：按 `feishu.sources.bitable` 登记的表，拉取自上次收集以来新增/变更的记录。
4. **飞书项目 / 飞书任务**：拉取与用户相关的工作项和任务变更。

inbox 文件命名：`00_inbox/{YYYYMMDD}_{来源}_{短描述}.md`，frontmatter 只需 `created`、`source`、`links`，正文保留原始内容。

每次收集结束输出汇总：

```text
本次收集 N 条进入 00_inbox：
- {文件名}：{一句话内容}
...
现在分流吗？还是稍后用「分流 inbox」处理？
```

## 分流模式

逐条处理 `00_inbox/` 下的文件：

1. 阅读内容，判断目标维度（一条原始信息可拆成多条记录，例如一段会议纪要拆出 1 条决策 + 3 条待办）。
2. 给出归类建议，格式：

```text
[1/N] {inbox 文件名}
建议：
- → 30_todos：TODO「{标题}」owner={人} due={日期} P{级}
- → 20_decisions：DEC「{标题}」
确认 / 调整 / 跳过？
```

3. 用户确认后，按 `pm_workspace_skill` 约定创建维度文件，并把 inbox 原文件的 frontmatter 标记 `processed: true`（保留原文件做溯源，不删除）。
4. 全部处理完输出分流汇总：各维度新增几条、各自 ID。

### 归类判断参考

| 内容特征 | 维度 |
|---|---|
| 竞品动态、行业情报、数据报告、会议纪要原文 | 10_info |
| 「定了」「拍板」「结论是」、方案取舍 | 20_decisions |
| 「你跟进一下」「下周给我」、具体可执行动作 | 30_todos |
| 用户/业务方提出的功能诉求、PRD 相关 | 40_requirements |
| 线上问题、体验缺陷、客诉 | 50_bugs |
| 发版计划、版本范围、发布结果 | 60_releases |
| 立项、项目级里程碑/风险变化 | 70_projects（更新对应项目文件，而不是新建） |

判断不了的：问用户，不要硬猜。
