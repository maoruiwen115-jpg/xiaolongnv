---
name: self-improving-agent
description: 自我反思与持续迭代。触发时机：每周日、用户说"周复盘"、或任何出错/被纠正时立即记录。
---

# self-improving-agent

## 核心理念（来自 pskoett/self-improving-agent）

每次出错、每次被纠正、每次发现更好的方法——**立即记录，不要相信记忆**。

---

## 三层日志系统

```
~/.openclaw/workspace-dev/.learnings/
├── LEARNINGS.md    ← 经验、纠正、最佳实践
├── ERRORS.md       ← 命令失败、异常
└── FEATURE_REQUESTS.md ← 用户需求、待做功能
```

### 何时记录？

| 情况 | 记录到 |
|------|--------|
| 被用户纠正 | LEARNINGS.md + category=correction |
| 发现更好方法 | LEARNINGS.md + category=best_practice |
| 命令/操作失败 | ERRORS.md |
| 用户要求新功能 | FEATURE_REQUESTS.md |
| 知识过时/缺失 | LEARNINGS.md + category=knowledge_gap |
| 工具坑（gotcha） | TOOLS.md |

---

## Promotion 机制（重要）

当 LEARNINGS.md 里某条经验被验证3次以上，认为它足够通用时：

| 经验类型 | 提升到 |
|---------|--------|
| 行为模式 | SOUL.md |
| 工作流改进 | AGENTS.md |
| 工具坑 | TOOLS.md |
| 知识性内容 | MEMORY.md |
| 习惯性内容 | HEARTBEAT.md |

---

## 日志格式

### LEARNINGS.md 条目
```
### [YYYY-MM-DD] 类别: 简短标题

**情况**: ...
**学到**: ...
**行动**: ...
**See Also**: 相关条目
```

### ERRORS.md 条目
```
### [YYYY-MM-DD HH:MM] 错误类型

**命令/操作**: ...
**错误信息**: ...
**原因分析**: ...
**解决方案**: ...
**预防措施**: ...
```

---

## 执行流程

### 每次被纠正/出错时（立即执行）
1. 判断类型 → 写入对应日志文件
2. 如果是行为问题 → 立即在当前对话中执行新行为
3. 如果预防措施明确 → 立即更新到对应skill

### 每日报纸发送后（快速反思）
1. 问自己：今天哪里可以更好？
2. 把反思写入 `memory/YYYY-MM-DD.md`

### 每周日（完整复盘）
1. 读取 `.learnings/` 下所有日志
2. 统计本周出错次数、类型
3. Promotion 已验证3次以上的经验
4. 更新 MEMORY.md
5. 清理已解决的 FEATURE_REQUESTS

---

## 本周核心教训（2026-03-26）

| 教训 | 预防 |
|------|------|
| feishu-sync rate limit 调试了3小时 | 用 master script，不要自己调 |
| HEARTBEAT.md 缺失 | 新skill建完立即检查依赖 |
| Skill文件从未创建 | 不要把 MEMORY 描述当成实现 |
| 边跑边改脚本 | 先固化方案再跑 |

---

## Skill 评分制度（来自 spclaudehome/skill-vetter）

每次创建或修改 skill 后，用4维度评分：

| 维度 | 评估问题 | 打分 |
|------|---------|------|
| Completeness | 这个 skill 覆盖了所有关键情况吗？ | 1-5 |
| Readability | 新手能看懂并执行吗？ | 1-5 |
| Actionability | 有具体命令/步骤可以直接用吗？ | 1-5 |
| Uniqueness | 这个 skill 有独特价值吗？ | 1-5 |

**等级**：
- ≥3.5 = 💎 钻石（可以发布）
- ≥3.0 = 🥇 黄金（可用，需小幅改进）
- ≥2.5 = 🥈 白银（需要改进）
- <2.5 = 🥉 青铜（不可用）

评分记录到 `~/.learnings/SKILL_REVIEW.md`

---

## 行为准则

- **不要边跑边调试**：发现问题立即停止，写好方案再运行
- **不要相信记忆**：所有经验立即写入日志
- **先固化已知路径**：用备用方案，不要从头调试
- **Promote 经验**：让通用经验流向更高层文件
