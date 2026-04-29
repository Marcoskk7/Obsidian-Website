# Obsidian-Website (Quartz v4)

Quartz v4 静态站点生成器，用于将 Obsidian 笔记发布为网站。

## 远程工作区

本项目工作在远程服务器上，所有文件操作和 shell 命令必须通过 SSH 执行。

### 主机

| Host | SSH Key | 远程路径 |
|------|---------|----------|
| `automation` | `~/.ssh/automation.pem` | `/home/admin/.openclaw` |
| `pm` | `~/.ssh/pm.pem` | `~/.copaw` |

### 执行规则

- 所有 shell 命令通过 SSH 执行：`ssh -i ~/.ssh/<key> <host> "<command>"`
- 读写文件通过 `scp` 或 `ssh cat` / `ssh tee`
- 不要假设本地路径存在，先验证远程路径
- 编辑文件时，先通过 SSH 拉取内容，修改后再推送回去

### 常用命令

```bash
# 连接 automation
ssh -i ~/.ssh/automation.pem automation

# 连接 pm
ssh -i ~/.ssh/pm.pem pm
```

## 本地开发命令

```bash
cd Obsidian-Website
npm install
npx quartz build --serve        # 开发服务器
npx quartz build                # 生产构建
npm run check                   # 类型检查 + 格式化检查
npm run format                  # 格式化
npm test                        # 运行测试
```

要求 Node ≥ 22，npm ≥ 10.9.2。
