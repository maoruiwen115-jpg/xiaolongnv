---
name: obsidian
description: 写入Obsidian笔记文件。触发时机：日报生成后写入本地Obsidian知识库。已知关键问题：write工具创建的文件Obsidian可能不索引，用exec shell echo创建文件Obsidian正常显示。
---

# obsidian

将内容写入 Obsidian 本地笔记库。

## 关键发现（2026-03-25验证，2026-03-26再次确认）

- ❌ `write` 工具创建的文件 → Obsidian 可能不索引（文件存在但不显示）
- ✅ `exec` shell `echo` 创建的文件 → Obsidian 正常显示
- **强制要求**：用 exec 工具写文件，不用 write 工具

## 日报存放路径（固定格式）

```
/Users/maoruiwen/Documents/小龙女/小龙女/每日记录/YYYY/M月/
YYYY-MM-DD-日报-关键词.md
```

**文件名规范**：
- 日期必须是 `YYYY-MM-DD` 格式
- 关键词控制在5字以内
- 标题中包含日期和关键词，便于搜索

## 执行步骤

1. 接收 markdown 内容（完整日报）
2. 确定文件路径（根据日期）
3. **必须用 exec 写入**：

```bash
# 先创建目录
mkdir -p "/Users/maoruiwen/Documents/小龙女/小龙女/每日记录/2026/3月"

# 用cat heredoc写入（exec方式，Obsidian兼容）
cat > "/Users/maoruiwen/Documents/小龙女/小龙女/每日记录/2026/3月/2026-03-26-日报-关键词.md" << 'EOF'
[日报内容]
EOF
```

4. **验证文件存在且Obsidian可见**：
```bash
ls -la "/Users/maoruiwen/Documents/小龙女/小龙女/每日记录/2026/3月/"
# 确认文件大小 > 0
```

## heredoc 注意事项

- 使用单引号 `'EOF'` 作为分隔符，避免 `$` 变量展开
- 如果内容包含单引号，先写入临时文件再复制

## 完整命令模板

```bash
FILE="/Users/maoruiwen/Documents/小龙女/小龙女/每日记录/2026/3月/YYYY-MM-DD-日报-关键词.md"
mkdir -p "$(dirname "$FILE")"
cat > "$FILE" << 'JOURNALEOF'
[这里粘贴完整日报内容]
JOURNALEOF
echo "Done. Size: $(wc -c < "$FILE") bytes"
```

## 验证清单

- [ ] 文件已创建（`ls -la` 确认）
- [ ] 文件大小 > 1000 bytes（确认内容完整）
- [ ] 打开 Obsidian 确认文件出现在库中

## 小龙女主动优化空间

每次写入后，快速检查：
- 文件名是否规范？
- Obsidian里能否搜到这个文件？
- 有没有乱码或内容截断？

发现问题立即重写，不要将就。
