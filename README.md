# openSkill

公开 Skill 仓库。后续可复用、可分享的 Skill 都放在这里。

## 当前 Skill

- `feishu_installCli_skill`：安装飞书 CLI，并按单飞书 / 多飞书场景完成初始化、scope 授权和验证。

## 仓库结构

```text
openSkill/
├── feishu_installCli_skill/
│   ├── SKILL.md
│   └── agents/
│       └── openai.yaml
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

## 注意

- Skill 里的 `<APP_ID>`、`App Secret`、`scope` 需要按自己的飞书应用替换。
- App Secret 不要发到聊天、文档或 GitHub。
- 内部标准 scope 清单不要提交到公开仓库。
