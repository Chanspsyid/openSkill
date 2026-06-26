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

> 前置依赖：对方电脑需先装好 **Node.js**（提供 `npm` / `npx`）和 **git**。下面的安装命令会用 `git clone` 拉取本仓库，没有 git 会直接报错。
> - 验证：`node -v` 和 `git --version` 都能正常输出即可。
> - macOS 首次运行 `git` 会弹出"安装命令行工具"，按提示装即可；Windows 默认不带 git，需单独安装 [Git for Windows](https://git-scm.com/download/win)。

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
