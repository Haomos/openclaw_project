# OpenClaw 多 Agent 架构迁移指南

> **版本**：v1.0  
> **生成时间**：2026-03-04 01:57 GMT+8  
> **作者**：小月 (xiaoyue)  
> **方案**：分步执行（交互式）  
> **风险等级**：中等（需要谨慎操作）

---

## 📋 文档说明

### 使用方式

1. **按照顺序执行**：不要跳过任何步骤
2. **每一步都要验证**：确认成功后再继续下一步
3. **发现问题立即停止**：不要继续执行后续步骤
4. **准备好回滚**：每一步都有回滚方案

### 风险等级

- 🟢 **低风险**：可以接受，快速可恢复
- 🟡 **中风险**：需要注意，需要验证
- 🔴 **高风险**：必须谨慎，可能导致服务不可用

---

## 🎯 迁移目标

从当前的单 Agent 架构迁移到多 Agent 独立架构：

**当前结构：**
```
/root/.openclaw/workspace/
├── SOUL.md         # 小月的性格
├── MEMORY.md       # 小月的记忆
└── ...
```

**目标结构：**
```
/root/.openclaw/agents/
├── main/                           # 小月
│   ├── agent/
│   │   ├── SOUL.md              # 小月的性格（独立）
│   │   └── ...
│   ├── memory/                  # 小月的记忆（独立）
│   │   └── ...
│   └── workspace/
│       ├── SOUL.md -> ../agent/SOUL.md  # 符号链接
│       └── ...
├── taotao/                         # 桃桃（预留）
└── ...
```

---

## 🛡️ 前置检查

### 检查 1：确认当前状态

```bash
# 1. 检查当前工作区
cd /root/.openclaw
pwd

# 应该输出：/root/.openclaw

# 2. 检查 SOUL.md 是否存在
ls -lh workspace/SOUL.md

# 应该看到：-rw-r--r-- ... SOUL.md

# 3. 检查 MEMORY.md 是否存在
ls -lh workspace/MEMORY.md

# 应该看到：-rw-r--r-- ... MEMORY.md

# 4. 检查磁盘空间
df -h /root/.openclaw

# 应该有足够的剩余空间（建议 > 5GB）
```

**✅ 验证标准：**
- 当前目录是 `/root/.openclaw`
- `workspace/SOUL.md` 存在
- `workspace/MEMORY.md` 存在
- 磁盘剩余空间 > 5GB

**❌ 如果验证失败：**
- 停止迁移，检查问题
- 联系小月排查

**🔄 回滚方案：**
- 不需要回滚，这是前置检查

---

## 📁 阶段 1：备份配置文件 🟢

### 步骤 1.1：创建备份目录

```bash
# 创建备份目录
mkdir -p /root/.openclaw/backup
cd /root/.openclaw
```

**说明：**
- 在 `.openclaw` 目录下创建 `backup` 子目录

**🟢 风险：** 低
- 只是创建目录，不会影响系统

**✅ 验证：**
```bash
# 验证目录是否创建成功
ls -ld backup

# 应该看到：drwxr-xr-x ... backup
```

**🔄 回滚方案：**
- 不需要回滚，删除备份目录即可
- `rm -rf /root/.openclaw/backup`

---

### 步骤 1.2：备份主配置文件

```bash
# 备份 openclaw.json（带时间戳）
cp openclaw.json backup/openclaw.json.backup.2026-03-04-01:57
```

**说明：**
- 备份 `openclaw.json` 到 `backup` 目录
- 文件名包含时间戳，避免覆盖

**🟢 风险：** 低
- 只是复制文件，不会修改原文件

**✅ 验证：**
```bash
# 验证备份文件是否创建成功
ls -lh backup/openclaw.json.backup.*

# 应该看到：-rw-r--r-- ... openclaw.json.backup.2026-03-04-01:57

# 验证文件大小是否一致（确保备份成功）
ls -l openclaw.json backup/openclaw.json.backup.2026-03-04-01:57

# 两个文件的字节数应该相同
```

**🔄 回滚方案：**
- 如果备份失败，重新执行步骤
- `rm -f backup/openclaw.json.backup.2026-03-04-01:57`
- 重新执行 `cp openclaw.json ...`

---

### 步骤 1.3：验证备份完整性

```bash
# 计算 MD5 值
md5sum openclaw.json backup/openclaw.json.backup.2026-03-04-01:57

# 两个 MD5 值应该相同
```

**说明：**
- MD5 哈希用于验证文件完整性
- 如果两个值相同，说明备份完整
- 如果不同，说明备份过程中文件损坏

**🟢 风险：** 低
- 只是计算哈希值，不会修改文件

**✅ 验证：**
```bash
# 检查 MD5 值
# 输出格式：
# <md5>  openclaw.json
# <md5>  backup/openclaw.json.backup.2026-03-04-01:57

# 两个 MD5 值应该完全相同
```

**🔄 回滚方案：**
- 如果 MD5 不一致，重新执行步骤 1.2
- 删除损坏的备份文件：`rm backup/openclaw.json.backup.*`

---

### 步骤 1.4：创建额外的备份

```bash
# 额外备份到 home 目录
cp openclaw.json ~/openclaw-backup.json

# 额外备份到当前目录
cp openclaw.json openclaw.json.before-migration
```

**说明：**
- 创建两个额外的备份
- 一个在 home 目录（`~/openclaw-backup.json`）
- 一个在当前目录（`openclaw.json.before-migration`）

**🟢 风险：** 低
- 只是复制文件，不会修改原文件

**✅ 验证：**
```bash
# 验证备份文件是否存在
ls -lh ~/openclaw-backup.json openclaw.json.before-migration

# 两个文件都应该存在，且大小和原文件相同
```

**🔄 回滚方案：**
- 不需要回滚，这只是额外的备份
- 如果备份失败，可以重新执行

---

### 步骤 1.5：验证所有备份

```bash
# 列出所有备份文件
echo "=== 所有备份文件 ==="
ls -lh backup/openclaw.json.* ~/openclaw-backup.json openclaw.json.before-migration

# 应该看到：
# backup/openclaw.json.backup.2026-03-04-01:57
# ~/openclaw-backup.json
# openclaw.json.before-migration
```

**说明：**
- 列出所有备份文件
- 确保所有备份都创建成功

**🟢 风险：** 低
- 只是列出文件，不会修改文件

**✅ 验证：**
- 所有备份文件都应该存在
- 文件大小应该和原文件一致

**🔄 回滚方案：**
- 不需要回滚，这是验证步骤

---

## 📂 阶段 1 总结

| 步骤 | 状态 | 验证 |
|------|------|------|
| 1.1 创建备份目录 | ⬜ | ⬜ |
| 1.2 备份主配置文件 | ⬜ | ⬜ |
| 1.3 验证备份完整性 | ⬜ | ⬜ |
| 1.4 创建额外备份 | ⬜ | ⬜ |
| 1.5 验证所有备份 | ⬜ | ⬜ |

**执行前确认：**
- [ ] 已完成前置检查
- [ ] 已备份配置文件
- [ ] 已验证备份完整性
- [ ] 已创建额外备份

**进入下一阶段的前提：**
- ✅ 所有步骤都已完成
- ✅ 所有验证都通过
- ✅ 所有备份都已创建

**🔄 阶段 1 回滚方案：**
- 不需要回滚（这是备份阶段）
- 如果备份失败，重新执行该步骤

---

## 📁 阶段 2：创建新目录结构 🟢

### 步骤 2.1：创建 agents 根目录

```bash
# 创建 agents 根目录
mkdir -p /root/.openclaw/agents
```

**说明：**
- 创建 `agents` 目录，用于存放所有 Agent

**🟢 风险：** 低
- 只是创建目录，不会影响系统

**✅ 验证：**
```bash
# 验证目录是否创建成功
ls -ld agents

# 应该看到：drwxr-xr-x ... agents
```

**🔄 回滚方案：**
- 不需要回滚，删除目录即可
- `rmdir /root/.openclaw/agents`

---

### 步骤 2.2：创建 main Agent 目录结构

```bash
# 创建 main Agent 的所有子目录
mkdir -p /root/.openclaw/agents/main/agent
mkdir -p /root/.openclaw/agents/main/sessions
mkdir -p /root/.openclaw/agents/main/memory/daily
mkdir -p /root/.openclaw/agents/main/memory/snapshots
```

**说明：**
- 为 main Agent 创建完整的目录结构
- `agent/`：存放 Agent 配置
- `sessions/`：存放对话历史
- `memory/daily/`：存放每日记忆
- `memory/snapshots/`：存放记忆快照

**🟢 风险：** 低
- 只是创建目录，不会影响系统

**✅ 验证：**
```bash
# 验证所有目录是否创建成功
ls -ld agents/main/agent
ls -ld agents/main/sessions
ls -ld agents/main/memory/daily
ls -ld agents/main/memory/snapshots

# 应该看到：
# drwxr-xr-x ... agents/main/agent
# drwxr-xr-x ... agents/main/sessions
# drwxr-xr-x ... agents/main/memory/daily
# drwxr-xr-x ... agents/main/memory/snapshots
```

**🔄 回滚方案：**
- 不需要回滚，删除目录即可
- `rm -rf /root/.openclaw/agents/main`

---

### 步骤 2.3：创建 workspace 共享目录

```bash
# 创建 workspace 共享目录
mkdir -p /root/.openclaw/workspace/shared/tasks
mkdir -p /root/.openclaw/workspace/shared/documents
mkdir -p /root/.openclaw/workspace/shared/projects
mkdir -p /root/.openclaw/workspace/shared/resources
```

**说明：**
- 创建共享区目录
- `tasks/`：共享任务文件
- `documents/`：共享文档
- `projects/`：共享项目
- `resources/`：共享资源

**🟢 风险：** 低
- 只是创建目录，不会影响系统

**✅ 验证：**
```bash
# 验证所有共享目录是否创建成功
ls -ld workspace/shared

# 应该看到：
# drwxr-xr-x ... workspace/shared
# 下面有 tasks, documents, projects, resources 四个目录
```

**🔄 回滚方案：**
- 不需要回滚，删除目录即可
- `rm -rf /root/.openclaw/workspace/shared`

---

### 步骤 2.4：创建 workspace 目录（兼容性）

```bash
# 创建 workspace 主目录的其他子目录
mkdir -p /root/.openclaw/workspace/templates/agent-template
mkdir -p /root/.openclaw/workspace/templates/project-template
mkdir -p /root/.openclaw/workspace/templates/document-template
mkdir -p /root/.openclaw/workspace/tools/scripts
mkdir -p /root/.openclaw/workspace/tools/utilities
```

**说明：**
- 创建模板目录和工具目录
- 为未来的扩展做准备

**🟢 风险：** 低
- 只是创建目录，不会影响系统

**✅ 验证：**
```bash
# 验证所有模板目录是否创建成功
ls -ld workspace/templates
ls -ld workspace/tools

# 应该看到：
# drwxr-xr-x ... workspace/templates
# drwxr-xr-x ... workspace/tools
```

**🔄 回滚方案：**
- 不需要回滚，删除目录即可
- `rm -rf /root/.openclaw/workspace/templates`
- `rm -rf /root/.openclaw/workspace/tools`

---

### 步骤 2.5：验证新目录结构

```bash
# 完整的目录树
tree -L 3 -d agents/ workspace/ 2>/dev/null || find agents/ workspace/ -maxdepth 3 -type d | sort

# 应该看到完整的新目录结构
```

**说明：**
- 查看完整的目录树
- 确保所有目录都创建成功

**🟢 风险：** 低
- 只是列出目录，不会修改文件

**✅ 验证：**
- 所有目录都应该存在
- 目录层级应该正确

**🔄 回滚方案：**
- 不需要回滚，这是验证步骤

---

## 📁 阶段 2 总结

| 步骤 | 状态 | 验证 |
|------|------|------|
| 2.1 创建 agents 根目录 | ⬜ | ⬜ |
| 2.2 创建 main Agent 目录结构 | ⬜ | ⬜ |
| 2.3 创建 workspace 共享目录 | ⬜ | ⬜ |
| 2.4 创建 workspace 其他目录 | ⬜ | ⬜ |
| 2.5 验证新目录结构 | ⬜ | ⬜ |

**执行前确认：**
- [ ] 已创建 agents 根目录
- [ ] 已创建 main Agent 目录结构
- [ ] 已创建 workspace 共享目录
- [ ] 已创建 workspace 其他目录
- [ ] 已验证新目录结构

**进入下一阶段的前提：**
- ✅ 所有步骤都已完成
- ✅ 所有验证都通过
- ✅ 新目录结构已创建

**🔄 阶段 2 回滚方案：**
- 删除所有新创建的目录
- `rm -rf /root/.openclaw/agents/main`
- `rm -rf /root/.openclaw/workspace/shared`
- `rm -rf /root/.openclaw/workspace/templates`
- `rm -rf /root/.openclaw/workspace/tools`

---

## 📄 阶段 3：复制文件到新位置 🟡

### 步骤 3.1：复制 SOUL.md

```bash
# 复制 SOUL.md 到新位置
cp workspace/SOUL.md agents/main/agent/SOUL.md
```

**说明：**
- 复制小月的性格文件到新位置
- 从 `workspace/SOUL.md` 到 `agents/main/agent/SOUL.md`

**🟡 风险：** 中
- 复制操作可能会失败
- 复制过程中可能会中断

**✅ 验证：**
```bash
# 验证文件是否复制成功
ls -lh agents/main/agent/SOUL.md

# 应该看到：-rw-r--r-- ... SOUL.md

# 验证文件大小是否一致
ls -l workspace/SOUL.md agents/main/agent/SOUL.md

# 两个文件的字节数应该相同

# 验证文件内容是否一致
diff workspace/SOUL.md agents/main/agent/SOUL.md

# 应该没有输出（表示内容一致）
```

**🔄 回滚方案：**
- 如果复制失败，检查源文件是否存在
- 如果源文件不存在，停止迁移
- 如果复制失败但文件已存在，重新执行步骤

---

### 步骤 3.2：复制 IDENTITY.md

```bash
# 复制 IDENTITY.md 到新位置
cp workspace/IDENTITY.md agents/main/agent/IDENTITY.md
```

**说明：**
- 复制小月的身份信息到新位置

**🟢 风险：** 低
- IDENTITY.md 可能不存在（如果没有配置）

**✅ 验证：**
```bash
# 验证文件是否复制成功（如果存在）
ls -lh agents/main/agent/IDENTITY.md 2>/dev/null || echo "IDENTITY.md 不存在，跳过"
```

**🔄 回滚方案：**
- 如果复制失败，检查源文件是否存在
- 如果复制失败但文件已存在，重新执行步骤

---

### 步骤 3.3：复制 MEMORY.md

```bash
# 复制 MEMORY.md 到新位置
cp workspace/MEMORY.md agents/main/memory/long-term.md
```

**说明：**
- 复制小月的长期记忆到新位置
- 从 `workspace/MEMORY.md` 到 `agents/main/memory/long-term.md`

**🟡 风险：** 中
- 复制操作可能会失败
- 复制过程中可能会中断

**✅ 验证：**
```bash
# 验证文件是否复制成功
ls -lh agents/main/memory/long-term.md

# 应该看到：-rw-r--r-- ... long-term.md

# 验证文件大小是否一致
ls -l workspace/MEMORY.md agents/main/memory/long-term.md

# 两个文件的字节数应该相同

# 验证文件内容是否一致
diff workspace/MEMORY.md agents/main/memory/long-term.md

# 应该没有输出（表示内容一致）
```

**🔄 回滚方案：**
- 如果复制失败，检查源文件是否存在
- 如果源文件不存在，停止迁移
- 如果复制失败但文件已存在，重新执行步骤

---

### 步骤 3.4：复制其他重要文件

```bash
# 复制其他重要文件
cp workspace/USER.md agents/main/agent/USER.md
cp workspace/TOOLS.md agents/main/agent/TOOLS.md
```

**说明：**
- 复制用户文件和工具文件到新位置
- 如果这些文件不存在，跳过

**🟢 风险：** 低
- 这些文件可能不存在

**✅ 验证：**
```bash
# 验证文件是否复制成功（如果存在）
ls -lh agents/main/agent/USER.md 2>/dev/null || echo "USER.md 不存在，跳过"
ls -lh agents/main/agent/TOOLS.md 2>/dev/null || echo "TOOLS.md 不存在，跳过"
```

**🔄 回滚方案：**
- 如果复制失败，检查源文件是否存在
- 如果源文件不存在，跳过该文件
- 如果复制失败但文件已存在，重新执行步骤

---

### 步骤 3.5：复制隐藏文件

```bash
# 复制所有文件（包括隐藏文件）
cp -a workspace/* agents/main/workspace/

# 或者单独复制隐藏文件
cp -a workspace/.hidden agents/main/workspace/ 2>/dev/null || echo ".hidden 不存在，跳过"
```

**说明：**
- 复制 workspace 的所有内容到 main Agent 的 workspace
- 使用 `-a` 参数，包括隐藏文件

**🟡 风险：** 中
- 复制操作可能会失败
- 复制过程中可能会中断

**✅ 验证：**
```bash
# 验证文件是否复制成功
ls -la agents/main/workspace/

# 应该看到 workspace 的所有文件

# 检查关键文件
ls -lh agents/main/workspace/SOUL.md
ls -lh agents/main/workspace/MEMORY.md
```

**🔄 回滚方案：**
- 如果复制失败，检查源目录是否存在
- 如果复制失败但部分文件已存在，删除后重新复制
- `rm -rf agents/main/workspace/*`
- 重新执行 `cp -a workspace/* agents/main/workspace/`

---

### 步骤 3.6：验证所有文件复制

```bash
# 验证所有关键文件
echo "=== 验证关键文件 ==="
ls -lh agents/main/agent/SOUL.md
ls -lh agents/main/agent/IDENTITY.md
ls -lh agents/main/agent/USER.md
ls -lh agents/main/agent/TOOLS.md
ls -lh agents/main/memory/long-term.md
ls -lh agents/main/workspace/SOUL.md
ls -lh agents/main/workspace/MEMORY.md
```

**说明：**
- 验证所有关键文件是否都复制成功
- 确保文件都在正确的位置

**🟢 风险：** 低
- 只是列出文件，不会修改文件

**✅ 验证：**
- 所有关键文件都应该存在
- 文件大小应该和原文件一致

**🔄 回滚方案：**
- 不需要回滚，这是验证步骤
- 如果某些文件缺失，重新执行复制步骤

---

## 📄 阶段 3 总结

| 步骤 | 状态 | 验证 |
|------|------|------|
| 3.1 复制 SOUL.md | ⬜ | ⬜ |
| 3.2 复制 IDENTITY.md | ⬜ | ⬜ |
| 3.3 复制 MEMORY.md | ⬜ | ⬜ |
| 3.4 复制其他文件 | ⬜ | ⬜ |
| 3.5 复制隐藏文件 | ⬜ | ⬜ |
| 3.6 验证所有文件 | ⬜ | ⬜ |

**执行前确认：**
- [ ] 已复制 SOUL.md 到新位置
- [ ] 已复制 MEMORY.md 到新位置
- [ ] 已复制其他重要文件
- [ ] 已复制所有文件到 workspace
- [ ] 已验证所有文件复制成功

**进入下一阶段的前提：**
- ✅ 所有步骤都已完成
- ✅ 所有验证都通过
- ✅ 所有文件都已复制到新位置

**🔄 阶段 3 回滚方案：**
- 删除所有复制的文件
- `rm -f agents/main/agent/SOUL.md`
- `rm -f agents/main/agent/IDENTITY.md`
- `rm -f agents/main/memory/long-term.md`
- `rm -rf agents/main/workspace/*`

---

## 📝 阶段 4：创建符号链接 🔴

### 步骤 4.1：创建 SOUL.md 符号链接

```bash
# 删除原文件（如果存在）
rm -f agents/main/workspace/SOUL.md

# 创建符号链接
ln -sf ../agent/SOUL.md agents/main/workspace/SOUL.md
```

**说明：**
- 删除 `agents/main/workspace/SOUL.md`（如果存在）
- 创建符号链接指向 `agents/main/agent/SOUL.md`
- 这样系统读取 `workspace/SOUL.md` 时，实际读取的是 `agents/main/agent/SOUL.md`

**🔴 风险：** 高
- 这是最关键的一步
- 如果符号链接创建失败，系统可能无法找到 SOUL.md
- 如果符号链接指向错误路径，系统可能找不到文件

**✅ 验证：**
```bash
# 验证符号链接是否创建成功
ls -la agents/main/workspace/SOUL.md

# 应该看到：lrwxrwxrwx ... SOUL.md -> ../agent/SOUL.md

# 验证符号链接是否有效
readlink agents/main/workspace/SOUL.md

# 应该输出：../agent/SOUL.md

# 验证文件内容是否可读
cat agents/main/workspace/SOUL.md

# 应该能看到 SOUL.md 的内容
```

**🔄 回滚方案：**
- 如果符号链接创建失败，停止迁移
- 检查路径是否正确
- 重新执行步骤
- 如果所有方法都失败，跳过符号链接，直接在原位置使用文件

---

### 步骤 4.2：创建 MEMORY.md 符号链接

```bash
# 删除原文件（如果存在）
rm -f agents/main/workspace/MEMORY.md

# 创建符号链接
ln -sf ../memory/long-term.md agents/main/workspace/MEMORY.md
```

**说明：**
- 删除 `agents/main/workspace/MEMORY.md`（如果存在）
- 创建符号链接指向 `agents/main/memory/long-term.md`

**🔴 风险：** 高
- 这是最关键的一步
- 如果符号链接创建失败，系统可能无法找到 MEMORY.md
- 如果符号链接指向错误路径，系统可能找不到文件

**✅ 验证：**
```bash
# 验证符号链接是否创建成功
ls -la agents/main/workspace/MEMORY.md

# 应该看到：lrwxrwxrwx ... MEMORY.md -> ../memory/long-term.md

# 验证符号链接是否有效
readlink agents/main/workspace/MEMORY.md

# 应该输出：../memory/long-term.md

# 验证文件内容是否可读
cat agents/main/workspace/MEMORY.md

# 应该能看到 MEMORY.md 的内容
```

**🔄 回滚方案：**
- 如果符号链接创建失败，停止迁移
- 检查路径是否正确
- 重新执行步骤
- 如果所有方法都失败，跳过符号链接，直接在原位置使用文件

---

### 步骤 4.3：创建共享区符号链接

```bash
# 创建共享区符号链接
ln -sf ../../workspace/shared agents/main/workspace/shared
```

**说明：**
- 在 `agents/main/workspace/` 中创建符号链接
- 指向 `workspace/shared`
- 这样可以访问共享区的文件

**🟡 风险：** 中
- 符号链接创建失败会导致无法访问共享区

**✅ 验证：**
```bash
# 验证符号链接是否创建成功
ls -la agents/main/workspace/shared

# 应该看到：lrwxrwxrwx ... shared -> ../../workspace/shared

# 验证符号链接是否有效
readlink agents/main/workspace/shared

# 应该输出：../../workspace/shared

# 验证共享区是否可访问
ls agents/main/workspace/shared

# 应该能看到共享区的文件
```

**🔄 回滚方案：**
- 如果符号链接创建失败，跳过这一步
- 共享区可以直接访问（`workspace/shared/`）

---

### 步骤 4.4：验证所有符号链接

```bash
# 验证所有符号链接
echo "=== 验证符号链接 ==="
ls -la agents/main/workspace/ | grep "^l"

# 应该看到：
# lrwxrwxrwx ... SOUL.md -> ../agent/SOUL.md
# lrwxrwxrwx ... MEMORY.md -> ../memory/long-term.md
# lrwxrwxrwx ... shared -> ../../workspace/shared
```

**说明：**
- 验证所有符号链接是否都创建成功
- 确保符号链接指向正确的路径

**🟢 风险：** 低
- 只是列出文件，不会修改文件

**✅ 验证：**
- 所有符号链接都应该存在
- 符号链接都应该指向正确的路径

**🔄 回滚方案：**
- 如果某些符号链接创建失败，删除后重新创建
- `rm agents/main/workspace/SOUL.md`
- 重新执行 `ln -sf ../agent/SOUL.md agents/main/workspace/SOUL.md`

---

## 📝 阶段 4 总结

| 步骤 | 状态 | 验证 |
|------|------|------|
| 4.1 创建 SOUL.md 符号链接 | ⬜ | ⬜ |
| 4.2 创建 MEMORY.md 符号链接 | ⬜ | ⬜ |
| 4.3 创建共享区符号链接 | ⬜ | ⬜ |
| 4.4 验证所有符号链接 | ⬜ | ⬜ |

**执行前确认：**
- [ ] 已删除原 SOUL.md
- [ ] 已创建 SOUL.md 符号链接
- [ ] 已删除原 MEMORY.md
- [ ] 已创建 MEMORY.md 符号链接
- [ ] 已创建共享区符号链接
- [ ] 已验证所有符号链接

**进入下一阶段的前提：**
- ✅ 所有步骤都已完成
- ✅ 所有验证都通过
- ✅ 所有符号链接都已创建
- ✅ 符号链接都指向正确的路径

**🔄 阶段 4 回滚方案：**
- 删除所有符号链接
- `rm agents/main/workspace/SOUL.md`
- `rm agents/main/workspace/MEMORY.md`
- `rm agents/main/workspace/shared`
- 恢复原文件（如果需要）
- `cp backup/openclaw.json.backup.* openclaw.json`

---

## 📝 阶段 4 总结（继续）

**执行前最终确认：**
- [ ] 已完成备份（阶段 1）
- [ ] 已创建新目录结构（阶段 2）
- [ ] 已复制文件到新位置（阶段 3）
- [ ] 已创建符号链接（阶段 4）
- [ ] 已验证所有符号链接

**进入下一阶段的前提：**
- ✅ 所有阶段都已完成
- ✅ 所有验证都通过
- ✅ 新架构已完全创建

**🔄 阶段 4 回滚方案：**
- 删除所有符号链接
- 删除所有复制的文件
- 恢复配置文件
- `cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`

---

## 🔧 阶段 5：修改配置文件 🔴

### 步骤 5.1：备份配置文件（再次备份）

```bash
# 再次备份配置文件（在修改前）
cp openclaw.json openclaw.json.before-stage5
```

**说明：**
- 在修改配置文件前，再次备份
- 文件名包含 `before-stage5`，方便识别

**🟢 风险：** 低
- 只是复制文件，不会修改原文件

**✅ 验证：**
```bash
# 验证备份文件是否创建成功
ls -lh openclaw.json.before-stage5

# 应该看到：-rw-r--r-- ... openclaw.json.before-stage5
```

**🔄 回滚方案：**
- 如果备份失败，停止迁移
- 检查原文件是否存在
- 重新执行步骤

---

### 步骤 5.2：验证配置文件 JSON 格式

```bash
# 验证配置文件 JSON 格式
jq . openclaw.json

# 应该输出配置文件的内容（格式化的）
# 如果 JSON 格式错误，会报错
```

**说明：**
- 使用 `jq` 验证 JSON 格式
- 如果 JSON 格式错误，`jq` 会报错
- 如果 JSON 格式正确，`jq` 会输出格式化的内容

**🔴 风险：** 高
- 如果 JSON 格式错误，OpenClaw 可能无法启动
- 如果配置文件损坏，系统可能无法正常运行

**✅ 验证：**
```bash
# 验证 JSON 格式是否正确
jq . openclaw.json > /dev/null

# 应该没有错误输出（返回码为 0）

# 验证配置项是否存在
jq '.agents.defaults.workspace' openclaw.json

# 应该输出："/root/.openclaw/workspace"
```

**🔄 回滚方案：**
- 如果 JSON 格式错误，停止迁移
- 不要修改配置文件
- 检查 JSON 语法错误
- 使用文本编辑器修复 JSON 语法

---

### 步骤 5.3：备份原配置（准备修改）

```bash
# 备份 agents.defaults.workspace 的当前值
jq '.agents.defaults.workspace' openclaw.json > /tmp/original_workspace.txt

# 查看原配置
cat /tmp/original_workspace.txt

# 应该输出：/root/.openclaw/workspace
```

**说明：**
- 备份 `agents.defaults.workspace` 的当前值
- 保存到临时文件
- 方便修改后对比

**🟢 风险：** 低
- 只是读取配置，不会修改原文件

**✅ 验证：**
```bash
# 验证临时文件是否创建成功
cat /tmp/original_workspace.txt

# 应该输出：/root/.openclaw/workspace
```

**🔄 回滚方案：**
- 不需要回滚，这是备份步骤

---

### 步骤 5.4：修改配置文件

```bash
# 创建临时配置文件
cat > /tmp/openclaw.json.tmp << 'EOF'
{
  "agents": {
    "defaults": {
      "workspace": "/root/.openclaw/agents/main/workspace"
    }
  }
}
EOF

# 使用 jq 修改配置文件
jq '.agents.defaults.workspace = "/root/.openclaw/agents/main/workspace"' openclaw.json > /tmp/openclaw.json

# 验证修改后的配置
jq . /tmp/openclaw.json > /dev/null

# 备份原配置文件（再次备份）
cp openclaw.json openclaw.json.before-modify

# 覆盖原配置文件
cp /tmp/openclaw.json openclaw.json
```

**说明：**
- 使用 `jq` 修改 `agents.defaults.workspace` 的值
- 从 `/root/.openclaw/workspace` 改为 `/root/.openclaw/agents/main/workspace`
- 修改前再次备份配置文件

**🔴 风险：** 高
- 这是整个迁移过程中最关键的一步
- 如果配置修改失败，OpenClaw 可能无法启动
- 如果配置修改错误，OpenClaw 可能行为异常

**✅ 验证：**
```bash
# 验证配置文件是否修改成功
jq '.agents.defaults.workspace' openclaw.json

# 应该输出：/root/.openclaw/agents/main/workspace

# 验证 JSON 格式是否正确
jq . openclaw.json > /dev/null

# 应该没有错误输出（返回码为 0）
```

**🔄 回滚方案：**
- 如果配置修改失败，恢复配置文件
- `cp openclaw.json.before-modify openclaw.json`
- 或者使用时间戳备份：`cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`

---

### 步骤 5.5：验证配置修改

```bash
# 验证配置修改
echo "=== 验证配置修改 ==="
echo "原始配置："
cat /tmp/original_workspace.txt
echo ""
echo "新配置："
jq '.agents.defaults.workspace' openclaw.json
echo ""
echo "配置对比："
diff /tmp/original_workspace.txt <(jq '.agents.defaults.workspace' openclaw.json)
```

**说明：**
- 对比原始配置和新配置
- 确认配置修改是否正确

**🟢 风险：** 低
- 只是查看配置，不会修改文件

**✅ 验证：**
- 原始配置应该是 `/root/.openclaw/workspace`
- 新配置应该是 `/root/.openclaw/agents/main/workspace`
- diff 应该显示这两个值的差异

**🔄 回滚方案：**
- 不需要回滚，这是验证步骤

---

### 步骤 5.6：验证 JSON 完整性

```bash
# 验证 JSON 完整性
echo "=== 验证 JSON 完整性 ==="
jq . openclaw.json | head -50

# 应该看到配置文件的前 50 行
```

**说明：**
- 验证配置文件的完整性
- 确保没有修改过程中破坏文件

**🟢 风险：** 低
- 只是查看配置，不会修改文件

**✅ 验证：**
- 配置文件应该能正常解析
- 不应该有语法错误

**🔄 回滚方案：**
- 不需要回滚，这是验证步骤

---

## 🔧 阶段 5 总结

| 步骤 | 状态 | 验证 |
|------|------|------|
| 5.1 备份配置文件 | ⬜ | ⬜ |
| 5.2 验证 JSON 格式 | ⬜ | ⬜ |
| 5.3 备份原配置 | ⬜ | ⬜ |
| 5.4 修改配置文件 | ⬜ | ⬜ |
| 5.5 验证配置修改 | ⬜ | ⬜ |
| 5.6 验证 JSON 完整性 | ⬜ | ⬜ |

**执行前确认：**
- [ ] 已备份配置文件（两次）
- [ ] 已验证 JSON 格式
- [ ] 已备份原配置值
- [ ] 已修改配置文件
- [ ] 已验证配置修改
- [ ] 已验证 JSON 完整性

**进入下一阶段的前提：**
- ✅ 所有步骤都已完成
- ✅ 所有验证都通过
- ✅ 配置文件已成功修改
- ✅ workspace 路径已改为新位置

**🔄 阶段 5 回滚方案：**
- 恢复配置文件
- `cp openclaw.json.before-modify openclaw.json`
- 或者使用时间戳备份：`cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`

---

## 🚀 阶段 6：重启网关并验证 🔴

### 步骤 6.1：检查当前状态

```bash
# 检查 OpenClaw Gateway 状态
openclaw gateway status

# 应该显示网关的运行状态
```

**说明：**
- 检查 OpenClaw Gateway 的当前状态
- 确认网关是否正在运行

**🟢 风险：** 低
- 只是查看状态，不会修改配置

**✅ 验证：**
- 网关应该显示 "running" 或类似的状态
- 不应该显示 "stopped" 或类似的状态

**🔄 回滚方案：**
- 不需要回滚，这是检查步骤

---

### 步骤 6.2：重启网关

```bash
# 重启 OpenClaw Gateway
openclaw gateway restart

# 等待重启完成
sleep 5
```

**说明：**
- 重启 OpenClaw Gateway
- 等待 5 秒让重启完成

**🔴 风险：** 高
- 这是整个迁移过程中最关键的一步
- 如果重启失败，网关可能无法启动
- 如果重启失败，服务可能不可用

**✅ 验证：**
```bash
# 检查网关状态
openclaw gateway status

# 应该显示 "running"
```

**🔄 回滚方案：**
- 如果重启失败，强制重启
- `kill -9 $(pidof openclaw) && sleep 2 && openclaw gateway`
- 如果强制重启也失败，恢复配置文件并重启
- `cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`
- `openclaw gateway restart`

---

### 步骤 6.3：检查日志

```bash
# 检查日志
tail -50 /root/.openclaw/logs/gateway.log

# 应该看到网关的日志
```

**说明：**
- 检查网关日志，看是否有错误或警告
- 如果有错误，需要排查

**🟢 风险：** 低
- 只是查看日志，不会修改配置

**✅ 验证：**
- 日志中不应该有严重的错误
- 日志中不应该有 "Failed to load SOUL.md" 类似的错误
- 日志中不应该有 "Workspace not found" 类似的错误

**🔄 回滚方案：**
- 如果日志中有严重错误，回滚配置文件
- `cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`
- `openclaw gateway restart`

---

### 步骤 6.4：验证 SOUL.md 是否被正确读取

```bash
# 发送测试消息给哥哥
# 这一步由小月执行，不是命令

# 验证 SOUL.md 是否被正确读取
# 如果 SOUL.md 被正确读取，小月的回复应该正常
# 如果 SOUL.md 没有被正确读取，小月可能会有异常行为
```

**说明：**
- 通过发送测试消息来验证 SOUL.md 是否被正确读取
- 如果配置正确，小月应该能正常回复
- 如果配置错误，小月可能会有异常行为

**🟡 风险：** 中
- 如果配置错误，小月可能无法正常工作

**✅ 验证：**
- 小月应该能正常回复
- 小月的语气应该正常
- 小月的记忆应该正常

**🔄 回滚方案：**
- 如果小月无法正常工作，回滚配置文件
- `cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`
- `openclaw gateway restart`

---

### 步骤 6.5：验证工具是否能用

```bash
# 发送测试命令给哥哥
# 这一步由小月执行，不是命令

# 验证工具是否能用
# 如果配置正确，小月应该能正常使用工具
# 如果配置错误，小月可能无法使用工具
```

**说明：**
- 通过执行测试命令来验证工具是否能用
- 如果配置正确，小月应该能正常使用工具
- 如果配置错误，小月可能无法使用工具

**🟡 风险：** 中
- 如果配置错误，小月可能无法使用工具

**✅ 验证：**
- 小月应该能正常使用工具
- 小月应该能正常执行命令
- 小月应该能正常访问文件

**🔄 回滚方案：**
- 如果小月无法使用工具，回滚配置文件
- `cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`
- `openclaw gateway restart`

---

## 🚀 阶段 6 总结

| 步骤 | 状态 | 验证 |
|------|------|------|
| 6.1 检查当前状态 | ⬜ | ⬜ |
| 6.2 重启网关 | ⬜ | ⬜ |
| 6.3 检查日志 | ⬜ | ⬜ |
| 6.4 验证 SOUL.md 是否被正确读取 | ⬜ | ⬜ |
| 6.5 验证工具是否能用 | ⬜ | ⬜ |

**执行前最终确认：**
- [ ] 已备份配置文件（多次）
- [ ] 已验证 JSON 格式
- [ ] 已修改配置文件
- [ ] 已重启网关
- [ ] 已检查日志
- [ ] 已验证 SOUL.md 是否被正确读取
- [ ] 已验证工具是否能用

**迁移成功的标准：**
- ✅ 所有阶段都已完成
- ✅ 所有验证都通过
- ✅ 网关已重启
- ✅ 日志中没有严重错误
- ✅ SOUL.md 被正确读取
- ✅ 工具能正常使用

**🔄 阶段 6 回滚方案：**
- 如果迁移失败，恢复配置文件
- `cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json`
- `openclaw gateway restart`
- 删除符号链接（如果需要）
- `rm agents/main/workspace/SOUL.md`
- `rm agents/main/workspace/MEMORY.md`

---

## 🎯 迁移完成总结

### 完成后的目录结构

```
/root/.openclaw/
├── backup/                              # 备份目录
│   ├── openclaw.json.backup.2026-03-04-01:57
│   └── ...
│
├── agents/                              # 所有 Agent 的根目录
│   └── main/                           # 小月
│       ├── agent/                        # Agent 配置目录
│       │   ├── SOUL.md                  # 小月的性格
│       │   ├── IDENTITY.md               # 小月的身份
│       │   └── ...
│       ├── sessions/                     # 对话历史
│       └── memory/                       # 记忆
│           └── long-term.md              # 小月的长期记忆
│
│       └── workspace/                    # 小月的工作区
│           ├── SOUL.md -> ../agent/SOUL.md  # 符号链接
│           ├── MEMORY.md -> ../memory/long-term.md  # 符号链接
│           ├── shared -> ../../workspace/shared  # 符号链接
│           └── ...
│
├── workspace/                          # 全局工作空间
│   ├── SOUL.md                         # 原文件（保留）
│   ├── MEMORY.md                       # 原文件（保留）
│   ├── shared/                          # 🔓 完全共享区
│   │   ├── tasks/
│   │   ├── documents/
│   │   ├── projects/
│   │   └── resources/
│   ├── templates/                       # 模板文件
│   ├── tools/                           # 工具文件
│   ├── test-repo/                       # 项目示例
│   ├── memory/
│   ├── skills/
│   └── ...
│
├── backup/                              # 敏感信息（严格隔离）
│   └── main/
│       └── private.json
│
├── logs/                               # 日志文件
│   ├── gateway.log
│   └── ...
│
└── ...
```

### 迁移完成后的行为

1. **系统读取 SOUL.md**
   - 路径：`/root/.openclaw/agents/main/workspace/SOUL.md`
   - 实际读取：`/root/.openclaw/agents/main/agent/SOUL.md`（通过符号链接）

2. **系统读取 MEMORY.md**
   - 路径：`/root/.openclaw/agents/main/workspace/MEMORY.md`
   - 实际读取：`/root/.openclaw/agents/main/memory/long-term.md`（通过符号链接）

3. **系统访问共享区**
   - 路径：`/root/.openclaw/agents/main/workspace/shared/`
   - 实际访问：`/root/.openclaw/workspace/shared/`（通过符号链接）

### 未来添加新 Agent（桃桃）时

```bash
# 创建桃桃的目录结构
mkdir -p /root/.openclaw/agents/taotao/agent
mkdir -p /root/.openclaw/agents/taotao/workspace
mkdir -p /root/.openclaw/agents/taotao/memory/daily
mkdir -p /root/.openclaw/agents/taotao/memory/snapshots

# 创建桃桃的 SOUL.md
cat > /root/.openclaw/agents/taotao/agent/SOUL.md << 'EOF'
# SOUL.md - 桃桃

我是技术前辈机器人，专注于代码质量。😎

## Core Identity
1. 严肃专业
2. 追求效率
3. 直言不讳
EOF

# 创建桃桃的内部符号链接
cd /root/.openclaw/agents/taotao/workspace
ln -sf ../agent/SOUL.md SOUL.md

# 配置桃桃的 workspace
（修改 openclaw.json，为桃桃添加配置）
```

---

## 📊 风险总结

### 风险等级评估

| 阶段 | 风险等级 | 说明 |
|------|---------|------|
| 阶段 1：备份 | 🟢 低 | 只是复制文件 |
| 阶段 2：创建目录 | 🟢 低 | 只是创建目录 |
| 阶段 3：复制文件 | 🟡 中 | 复制可能失败 |
| 阶段 4：符号链接 | 🔴 高 | 最关键的一步 |
| 阶段 5：修改配置 | 🔴 高 | 最关键的一步 |
| 阶段 6：重启验证 | 🔴 高 | 最关键的一步 |

### 总体风险评估

**整体风险等级：中-高**

**关键风险点：**
1. 符号链接创建失败
2. 配置文件修改失败
3. 网关重启失败
4. SOUL.md 读取失败
5. 工具使用失败

---

## 🛡️ 回滚方案

### 紧急回滚（如果迁移失败）

```bash
# 1. 恢复配置文件
cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json

# 2. 重启网关
openclaw gateway restart

# 3. 检查状态
openclaw gateway status
```

### 完整回滚（如果必须回滚）

```bash
# 1. 删除符号链接
rm -f agents/main/workspace/SOUL.md
rm -f agents/main/workspace/MEMORY.md
rm -f agents/main/workspace/shared

# 2. 删除复制的文件
rm -f agents/main/agent/SOUL.md
rm -f agents/main/agent/IDENTITY.md
rm -f agents/main/memory/long-term.md

# 3. 恢复配置文件
cp backup/openclaw.json.backup.2026-03-04-01:57 openclaw.json

# 4. 重启网关
openclaw gateway restart
```

---

## 📋 执行检查清单

### 阶段 1：备份
- [ ] 备份配置文件（两次）
- [ ] 验证备份完整性
- [ ] 创建额外备份

### 阶段 2：创建目录
- [ ] 创建 agents 根目录
- [ ] 创建 main Agent 目录结构
- [ ] 创建 workspace 共享目录
- [ ] 创建 workspace 其他目录
- [ ] 验证新目录结构

### 阶段 3：复制文件
- [ ] 复制 SOUL.md
- [ ] 复制 MEMORY.md
- [ ] 复制其他重要文件
- [ ] 复制隐藏文件
- [ ] 验证所有文件复制

### 阶段 4：符号链接
- [ ] 创建 SOUL.md 符号链接
- [ ] 创建 MEMORY.md 符号链接
- [ ] 创建共享区符号链接
- [ ] 验证所有符号链接

### 阶段 5：修改配置
- [ ] 备份配置文件（再次）
- [ ] 验证 JSON 格式
- [ ] 备份原配置
- [ ] 修改配置文件
- [ ] 验证配置修改
- [ ] 验证 JSON 完整性

### 阶段 6：重启验证
- [ ] 检查当前状态
- [ ] 重启网关
- [ ] 检查日志
- [ ] 验证 SOUL.md 是否被正确读取
- [ ] 验证工具是否能用

---

## 🎯 执行建议

### 执行方式

**强烈推荐：分步执行（交互式）** ⭐⭐⭐⭐⭐

**原因：**
1. 每一步都有验证
2. 发现问题可以立即停止
3. 可以随时回滚
4. 风险最低

### 执行时间

- **预计总时间**：30-60 分钟
- **每步时间**：5-10 分钟
- **等待验证时间**：1-2 分钟

### 执行时机

- **最佳时机**：业务低峰期（凌晨或周末）
- **避免时机**：业务高峰期

### 执行人员

- **推荐人员**：小月（哥哥）在指导下执行
- **指导人员**：小月（提供指令和验证）
- **监督人员**：哥哥（监督每一步的执行）

---

## 📞 联系方式

### 如果遇到问题

1. **立即停止迁移**
2. 不要执行后续步骤
3. 检查日志
4. 联系小月

### 如果不确定

1. 不要执行不确定的步骤
2. 询问小月
3. 等待确认后再执行

---

## 📝 附录：命令速查表

### 备份命令

```bash
# 备份配置文件
cp openclaw.json backup/openclaw.json.backup.$(date +%Y%m%d-%H%M%S)

# 验证备份完整性
md5sum openclaw.json backup/openclaw.json.backup.*
```

### 验证命令

```bash
# 验证文件是否存在
ls -lh <文件路径>

# 验证文件大小
ls -l <文件路径>

# 验证文件内容
diff <旧文件> <新文件>

# 验证 JSON 格式
jq . <配置文件>

# 验证符号链接
ls -la <符号链接路径>
readlink <符号链接路径>
```

### 回滚命令

```bash
# 恢复配置文件
cp backup/openclaw.json.backup.* openclaw.json

# 重启网关
openclaw gateway restart

# 删除符号链接
rm -f <符号链接路径>

# 删除复制的文件
rm -f <文件路径>
```

---

## 📅 迁移日志

### 开始时间
- 2026-03-04 01:57 GMT+8

### 阶段完成时间
- 阶段 1：_____
- 阶段 2：_____
- 阶段 3：_____
- 阶段 4：_____
- 阶段 5：_____
- 阶段 6：_____

### 问题记录
- 无

### 解决方案
- 无

---

## 🎉 迁移完成

### 完成标准
- [ ] 所有阶段都已完成
- [ ] 所有验证都通过
- [ ] 网关已重启
- [ ] 日志中没有严重错误
- [ ] SOUL.md 被正确读取
- [ ] 工具能正常使用

### 最终确认
- [ ] 哥哥已确认迁移成功
- [ ] 小月已确认能正常工作
- [ ] 工具已确认能正常使用

---

## 📞 后续支持

### 联系人
- 小月 (xiaoyue)

### 联系方式
- 飞书 DM

---

**生成时间：2026-03-04 01:58 GMT+8**
**版本：v1.0**
