---
name: auto-updater
description: 每日早晨自动触发器。触发时机：每天早上7:40（OpenClaw cron）或用户说"跑日报飞轮"。协调6个环节按两步执行。
---

# auto-updater

## 触发方式
- **自动触发**：OpenClaw 内置 cron（`openclaw cron`），每天 7:40 Asia/Shanghai，已配置
- **用户触发**：说"跑日报飞轮"，立即执行完整流程

## ⚠️ 重要：两步流程（2026-03-27 重大更新）

**第一步（cron 自动）** → 完成后暂停，等你确认
**第二步（你说"确认"后）** → 继续剩余步骤

这样做是为了：让你先检查 Obsidian 的日报内容，确认没问题后再生成飞书文档和公众号文章，避免内容返工。

---

## 第一步：cron 自动触发（7:40，每天）

### ① memory-updater
```bash
# 生成今日 context
mkdir -p ~/.openclaw/workspace-dev/memory
# 写入 today-context.md（见 memory-updater skill）
```

### ② ai-news（必须先搜索，详见 ai-news skill）

**新闻优先级**：36氪快讯 > 量子位/机器之心 > 财联社 > 虎嗅
**交叉验证**：重要结论先搜学术来源（PubMed/PNAS/Nature），再用大众媒体补充

```bash
# 主搜索
python3 /tmp/tavily_search.py "AI 人工智能 最新新闻 $(date +%Y年%m月%d日)"

# 如果失败，换维度继续搜，不放弃
python3 /tmp/tavily_search.py "OpenAI 大模型 最新 2026"
python3 /tmp/tavily_search.py "人形机器人 具身智能 2026"
```

### ③ daily-journal
```bash
# 1. 查人物传记数据库（必须！）
cat /Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/人物传记存档/人物传记索引.md

# 2. 查6课题数据库
ls /Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/6课题数据库/

# 3. 生成日报（调用 daily-journal skill）
```

### ④ obsidian
```bash
# 正确路径（来自 file-manager skill）：
# DB_DAILY = /Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档
cat > "/Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档/YYYY/M月/YYYY-MM-DD-日报-关键词.md" << 'EOF'
[日报内容]
EOF

# 验证
ls -la /Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档/YYYY/M月/YYYY-MM-DD-日报-关键词.md
```

### ✅ 完成后报告
发送消息给你：
> "✅ 日报已生成，内容在 /Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档/YYYY/M月/YYYY-MM-DD-日报-关键词.md。请确认内容，没问题后说"确认"，我会立即生成飞书文档和公众号文章。"

**停下来，等你说"确认"。**

---

## 第二步：你确认后执行

### ⑤ feishu-sync（Python API 方法，详见 feishu-sync skill）

**核心方法**（2026-03-27 验证通过）：
1. 读取 Obsidian markdown 文件
2. 用 convert API 把 markdown 转换成 Feishu blocks
3. 创建新文档
4. 逐个插入 blocks（batch_size=1 最稳定）
5. 设置公开权限

**执行命令**：
```bash
# 生成并运行脚本
python3 /tmp/feishu_sync.py
```

**脚本内容模板**（由主agent生成到 /tmp/feishu_sync.py）：
```python
#!/usr/bin/env python3
import urllib.request, json, time
APP_ID = 'cli_a93862a9a6f8dbd4'
APP_SECRET = 'RTyTroX4ZUQv18gySo8ZbbCrn5AunbTs'
# ... (见 feishu-sync skill)
```

**验证**：
- 链接可访问 ✅
- blocks 数量 160+ ✅（317 blocks 也成功过）
- 顺序正确 ✅

### ⑥ article-writer
```bash
# 用 article-writer skill 生成公众号文章
# 见 ~/.openclaw/skills/article-writer/SKILL.md
```

### ⑦ proactivity
发送微信群摘要（收到飞书链接后）。

---

## 检查清单

| 步骤 | 验证方法 |
|------|---------|
| ① context | `ls ~/.openclaw/workspace-dev/memory/today-context.md` |
| ② ai-news | 3条新闻已搜集（必须用 Tavily） |
| ③ journal | 文件存在，wc -c > 5000 |
| ④ obsidian | 路径正确：`1-数据库/日报存档/YYYY/M月/` |
| ⑤ feishu | 链接可访问，参考文献链接可点击 |
| ⑥ article | 文章已生成，wc -c > 3000 |
| ⑦ proactivity | 摘要已发送 |

## 错误处理

- **飞书 rate limit（1770001）**：等60秒重试
- **任何环节失败**：停止，告知用户具体环节
- **不要在流程中调试**：脚本写好后直接跑，不要边跑边改
- **参考文献链接不可点击**：用 update_block 逐条修复

## cron 配置状态（已验证正确）

```bash
# 每天 7:40 Asia/Shanghai
# sessionTarget: current（运行在主会话，可以交互）
# delivery: announce → feishu → ou_5452f63885a1adf9b130d947121df41d
openclaw cron list  # 查看状态
```
