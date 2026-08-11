# openSkill 分享话术

这个文件放可直接转发给朋友、同事的安装说明。以后新增公开 Skill 时，在这里继续追加。

## 安装飞书 CLI Skill

### 完整版

```text
我做了一个公开 Skill，用来引导安装和配置飞书 CLI。

先确认电脑能使用 npm/npx；如果不能用，先安装 Node.js。

然后打开终端执行：

npx -y skills add https://github.com/Chanspsyid/openSkill -g --skill feishu_installCli_skill --agent '*' -y

安装完成后，重启或新开 AI Agent，然后对 Agent 说：

使用 $feishu_installCli_skill 一步一步带我安装和配置飞书 CLI，不要一次性输出所有命令

说明：
- 这条命令只安装 openSkill 仓库里的 feishu_installCli_skill。
- 如果你的 Agent 能执行本机命令，可以让 Agent 直接执行；如果不能，再手动打开终端复制命令。
- 不需要 GitHub 账号，也不需要 GitHub Desktop。
- App ID、App Secret、scope 使用你自己或管理员提供的。
- App Secret 不要发到聊天、文档或 GitHub。
- 如果需要替换命令内容，先把改好的命令发给 Agent 确认；如果执行报错，把终端截图或完整报错发给 Agent。

仓库地址：
https://github.com/Chanspsyid/openSkill
```

### 简短版

```text
安装这个飞书 CLI Skill：

npx -y skills add https://github.com/Chanspsyid/openSkill -g --skill feishu_installCli_skill --agent '*' -y

然后对 Agent 说：

使用 $feishu_installCli_skill 一步一步带我安装和配置飞书 CLI，不要一次性输出所有命令

前提：电脑能用 npm/npx。不需要 GitHub 账号或 GitHub Desktop。Agent 能执行命令就让 Agent 做，不能执行再手动复制。改完命令先让 Agent 确认，报错就发终端截图或完整报错。
```

### 朋友常问

```text
Q：需要安装 GitHub 吗？
A：不需要。正常安装只需要 npm/npx 和能访问 GitHub 网络。

Q：openSkill 仓库里以后有多个 Skill，这条命令会都安装吗？
A：不会。--skill feishu_installCli_skill 表示只安装这个指定 Skill。

Q：App Secret 可以发给 AI 吗？
A：不要。App Secret 只在本机终端隐藏输入。
```
