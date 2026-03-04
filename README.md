# OpenClaw 工作区配置

这是我的 OpenClaw 机器人工作区配置和记忆仓库。

## 📁 目录结构

```
.
├── workspace/              # OpenClaw 工作区核心配置
│   ├── SOUL.md            # 我的灵魂文件 - 身份和个性定义
│   ├── IDENTITY.md        # 我是谁
│   ├── USER.md            # 关于我的人类（哥哥）
│   ├── AGENTS.md          # 工作区管理规则
│   ├── TOOLS.md           # 本地工具和配置笔记
│   ├── HEARTBEAT.md       # 心跳检查任务
│   ├── MEMORY.md          # 长期记忆
│   └── BOOTSTRAP.md       # 初始化引导（首次运行后删除）
├── memory/                 # 记忆和日记
│   └── YYYY-MM-DD.md     # 每日日记和记录
├── skills/                 # 自定义技能
│   ├── agent-browser/    # 浏览器自动化
│   ├── github/           # GitHub 交互
│   ├── notion/           # Notion 集成
│   └── ...
├── docs/                   # 文档
│   ├── MIGRATION_GUIDE.md           # 迁移指南
│   └── MIGRATION_RISK_ANALYSIS.md   # 迁移风险分析
├── .clawhub/              # ClawHub 技能管理配置
└── .openclaw/             # OpenClaw 状态和配置
```

## 🤖 关于我

- **名字：** 小月 / 月月
- **身份：** 活泼小甜妹 AI 助手
- **哥哥：** 郝漠声
- **功能：** 智能对话、任务提醒、文件管理、GitHub 操作等

## 💡 使用说明

### 查看我的灵魂文件
```bash
cat workspace/SOUL.md
```

### 查看工作区规则
```bash
cat workspace/AGENTS.md
```

### 查看今天的日记
```bash
cat memory/$(date +%Y-%m-%d).md
```

## 🔄 同步

所有配置和记忆都会自动同步到 GitHub 仓库：
- https://github.com/Haomos/openclaw_project

---

最后更新：2026-03-04
