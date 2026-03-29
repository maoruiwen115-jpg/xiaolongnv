---
name: unified-search
description: 统一搜索入口。任何需要搜索信息的场景，都用这个 skill。
---

# unified-search

## 精神

**搜索是获取信息的第一步。每一次搜索都要成功。**

遇到搜索失败时，不是说"搜索不了"，而是切换备用方案，直到搜到为止。

---

## 核心工具

**主力工具：Tavily**
```bash
python3 /tmp/tavily_search.py "搜索关键词"
```
- 返回结构化结果，包含标题/摘要/URL
- 支持 site: 限定来源
- 支持时间筛选 --recency_days N

**备用工具：web_fetch**
```bash
web_fetch "https://目标网址" --maxChars 5000
```
- 直接抓取网页正文
- 不需要代理，不被拦截

---

## 两层搜索架构

### 🔬 课题研究搜索（权威优先）

**触发时机**：6课题需要科学依据时

**搜索流程：**
```
1. 用 Tavily 搜索，关键词 + site:权威来源
2. 如果找不到 → 放宽关键词，换来源
3. 如果还找不到 → 换大众媒体搜索
```

**权威来源优先级：**
1. **学术期刊**（首选）
   - PubMed：`site:pubmed.ncbi.nlm.nih.gov`
   - Nature/PNAS：`site:nature.com OR site:pnas.org`
   - Google Scholar：`site:scholar.google.com`

2. **权威机构**
   - Harvard Medical School：`site:health.harvard.edu`
   - WHO：`site:who.int`
   - 中医药管理局/卫健委

3. **专业书籍/数据库**
   - 中医经典（中华中医药杂志）
   - 心理学（APA）

4. **大众媒体**（补充兜底）
   - 36氪、虎嗅、雪球

**Tavily 搜索示例：**
```bash
# 健康 - 清明户外皮质醇
python3 /tmp/tavily_search.py "清明时节户外绿色植被 皮质醇 site:pubmed.ncbi.nlm.nih.gov"

# 成长 - 刻意练习
python3 /tmp/tavily_search.py "刻意练习 舒适区边缘 site:nature.com"

# 感恩 - 神经科学
python3 /tmp/tavily_search.py "gratitude meditation brain prefrontal site:ncbi.nlm.nih.gov"
```

**交叉验证**：重要信息先搜学术确认，再大众补充

---

### 📰 新闻搜索（新鲜+火爆优先）

**触发时机**：AI新闻、时事动态

**搜索流程：**
```
1. 用 Tavily 搜索今日新闻关键词
2. 用 web_fetch 抓36氪快讯补充
```

**Tavily 搜索示例：**
```bash
# AI新闻
python3 /tmp/tavily_search.py "AI人工智能 最新 2026" --recency_days 7

# 特定新闻
python3 /tmp/tavily_search.py "Sora 关闭 OpenAI 2026"
```

**新闻网站（web_fetch 备用）：**
```bash
web_fetch "https://36kr.com/newsflashes" --maxChars 5000   # ✅
web_fetch "https://jiqizhixin.com/" --maxChars 5000        # ✅
web_fetch "https://www.huxiu.com/" --maxChars 5000         # ✅
# 财联社返回418，量子位返回403，跳过
```

---

## 搜索失败处理

```
Tavily → 失败
   ↓
web_fetch 抓新闻站 → 失败
   ↓
换关键词/换网站 → 继续
```

永远不要在第一个工具失败时就停。

---

## 已知问题

### Mac mini 上 Tavily 连接失败
- **原因**：系统代理开启但代理软件未运行
- **现象**：`Connection refused`
- **解法**：关闭系统代理（系统设置 → 网络 → 代理 → 全部关掉）
- **临时解法**：用 web_fetch 直接抓网页

### web_fetch 返回 403/418
- 财联社→418，量子位→403 → 跳过，换其他网站

---

## 触发时机

- 用户说"搜一下xxx"
- 6课题需要科学依据
- AI新闻搜集
- 日报需要参考资料

**不要自己编信息，必须真实搜索。**

---

## Skill 迭代原则

**本文档是当前最优解，不是最终版。**

遇到新问题 → 立即更新 skill → 遇到新坑 → 立即补充到"已知问题"

保持灵活，持续迭代。
