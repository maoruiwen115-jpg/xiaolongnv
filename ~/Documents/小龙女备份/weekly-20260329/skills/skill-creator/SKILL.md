---
name: skill-creator
description: 如何设计好的 Skill。触发时机：创建新skill之前、修改现有skill时。先读这个文件，理解原则再动手。
---

# skill-creator

如何设计和编写好的 Skill。

---

## 核心原则（来自 3 个 clawhub skill）

### 1. 精神 > 步骤（来自 chindden/skill-creator）

**好的 skill 描述精神，不是写操作手册。**

❌ 太详细（像 feishu-sync 之前那样）：
> "Step 1: 获取token。Step 2: curl POST...Step 3: 解析..."

✅ 描述精神：
> "用 master script 一行命令搞定。如果不可用，用 curl 直接调 API，每次 ≥15秒间隔。"

**原则**：写"做什么、为什么"，不写"按什么顺序点哪个按钮"。

---

### 2. 1 个能力 = 1 个 Skill（来自 chindden/skill-creator）

**每个 skill 只做一件事。**

❌ 不要这样：
> "这个 skill 既生成日报、又同步飞书、又发摘要"

✅ 这样做：
> "auto-updater 只负责调度。daily-journal 只负责生成内容。proactivity 只负责发摘要。"

---

### 3. 搜索前先 Clarify（来自 ivangdavila/find）

**不明确需求就开始写 = 浪费。**

当用户说"帮我做xxx"时：
1. 先问清楚：具体是什么？成功标准是什么？
2. 再开始设计 skill
3. 交付前验证

---

### 4. 每次出错立即记录（来自 pskoett/self-improving-agent）

**不要相信记忆。**

创建任何 skill 后，立即把：
- 已知坑 → 写入 skill 的"注意事项"
- 验证成功的命令 → 写入 skill
- 遇到的新错误 → 记到 `.learnings/ERRORS.md`

---

## Skill 设计模板

```markdown
---
name: skill-name
description: 一句话说这个 skill 是做什么的
---

# skill-name

## 精神（1-3句话）
描述这个 skill 存在的核心理由。

## 触发时机
什么时候用这个 skill？

## 使用方法
- 关键命令（如果有）
- 注意事项（已验证的坑）

## 已知问题
记录下来，避免重复踩坑。
```

---

## 创建新 Skill 的流程

1. **Clarify**：问清楚需求（来自 find）
2. **Design**：想好这个 skill 的"精神"，不要写步骤
3. **Create**：用系统的 `skill-creator` 创建：
   ```bash
   python3 /opt/homebrew/lib/node_modules/openclaw/skills/skill-creator/scripts/init_skill.py [name] --path ~/.openclaw/skills --resources scripts,references
   ```
4. **Document**：写 SKILL.md，遵循"精神 > 步骤"原则
5. **Test**：跑一次，验证能用
6. **Record**：把遇到的问题写入 skill 的"已知问题"

---

## 已有 Skill 的设计原则

| Skill | 设计问题 | 改进方向 |
|-------|---------|---------|
| feishu-sync | 太像操作手册 | 改为"精神"描述，只留 master script |
| daily-journal | 结构太详细 | 精简为"精神 + 必须查什么" |
| auto-updater | OK | 保持 |

---

## 参考

- chindden/skill-creator（clawhub）：Spirit over steps
- ivangdavila/find（clawhub）：Clarify before action
- pskoett/self-improving-agent（clawhub）：Log everything, promote regularly
