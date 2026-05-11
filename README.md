# ECC 本地管理

`~/Desktop/tool_main/` 是 ECC 唯一源码目录，通过软链接 + settings.json 注册到 Claude Code。

## 架构

```
~/Desktop/tool_main/          ← git 管理
├── skills/       → ~/.claude/skills/      (目录级软链接)
├── agents/       → ~/.claude/agents/      (文件级软链接)
├── commands/     → ~/.claude/commands/    (文件级软链接)
├── rules/        → ~/.claude/rules/       (目录级软链接)
├── hooks/        → ~/.claude/settings.json (合并到 hooks 字段)
└── mcp-configs/  → ~/.claude/settings.json (合并到 mcpServers 字段)
```

`CLAUDE_PLUGIN_ROOT=~/Desktop/tool_main` 已写入 settings.json，hook 脚本自动解析到此路径。

## 新增 Skill

```bash
# 1. 在 tool_main 下新建 skill 目录和文件
mkdir -p ~/Desktop/tool_main/skills/my-skill
# 编写 SKILL.md（参考 skills/configure-ecc/SKILL.md 的 frontmatter 格式）

# 2. 创建软链接
cd ~/.claude/skills
[ ! -e "my-skill" ] && ln -s ~/Desktop/tool_main/skills/my-skill my-skill
```

## 修改 Skill

直接编辑 `~/Desktop/tool_main/skills/<skill-name>/SKILL.md`，软链接实时生效。

## 新增 Agent / Command

```bash
# Agent
cd ~/.claude/agents
[ ! -e "my-agent.md" ] && ln -s ~/Desktop/tool_main/agents/my-agent.md my-agent.md

# Command
cd ~/.claude/commands
[ ! -e "my-command.md" ] && ln -s ~/Desktop/tool_main/commands/my-command.md my-command.md
```

## 新增 Rules

```bash
# 新增语言规则目录
mkdir -p ~/Desktop/tool_main/rules/my-lang
cd ~/.claude/rules
[ ! -e "my-lang" ] && ln -s ~/Desktop/tool_main/rules/my-lang my-lang
```

## 修改 Hooks / MCP

编辑 `hooks/hooks.json` 或 `mcp-configs/mcp-servers.json`，然后重新合并到 settings.json：

```bash
python3 << 'EOF'
import json

with open('/home/zya/.claude/settings.json') as f:
    settings = json.load(f)
with open('/home/zya/Desktop/tool_main/hooks/hooks.json') as f:
    hooks = json.load(f)
with open('/home/zya/Desktop/tool_main/mcp-configs/mcp-servers.json') as f:
    mcp = json.load(f)

settings['hooks'] = hooks['hooks']
settings['mcpServers'] = mcp['mcpServers']
settings['env']['CLAUDE_PLUGIN_ROOT'] = '/home/zya/Desktop/tool_main'

with open('/home/zya/.claude/settings.json', 'w') as f:
    json.dump(settings, f, indent=2, ensure_ascii=False)
print('✓ 已同步 settings.json')
EOF
```

## Git 工作流

```bash
cd ~/Desktop/tool_main
git add -A
git commit -m "feat: 描述你的改动"
git push
```
