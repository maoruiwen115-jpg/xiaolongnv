---
name: web-search-ex
description: 区分搜索与读文档，掌握正确网络搜索方法。触发时机：需要上网查信息时。
---

# web-search-ex

## 核心区分（最关键）

| 工具 | 功能 | 使用场景 |
|------|------|---------|
| `openclaw tools search duckduckgo` | **真正的网络搜索** | 搜新闻/搜内容/搜信息 |
| `web_fetch` | 获取已知URL正文 | 读取已知的网页内容 |
| `online`（内置） | **只能读文档** | 读官方文档，**不是搜索** |

**常见错误**：以为 `online` 可以做搜索，实际上它只能读文档。

---

## 正确搜索流程

### Step 1: Clarify（来自 find skill）

在搜索前先问自己：
1. 我到底要搜什么？（具体问题）
2. 成功标准是什么？（找到什么才算成功）
3. 已经知道什么？（避免重复搜同一内容）

### Step 2: 选工具

**主力工具：Tavily（已配置，推荐）**
```bash
python3 /tmp/tavily_search.py "搜索关键词"
# 或直接用 Python:
python3 -c "
from tavily import TavilyClient
client = TavilyClient(api_key='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')
r = client.search('关键词')
for x in r['results']: print(x['title'], x['url'])
"
```

**备用工具：web_fetch 直接抓新闻网站**
```bash
# 直接抓36kr快讯
web_fetch "https://36kr.com/newsflashes"
# 直接抓财联社
web_fetch "https://www.cls.cn/telegraph"
```

### Step 3: 验证结果

找到结果不等于搜对了，问自己：

| 问题 | 检查项 |
|------|--------|
| 来源可信吗？ | 官方文档 > 权威媒体 > 博客 > 论坛 |
| 时效对吗？ | 是最新的，还是过时的？ |
| 匹配吗？ | 有没有偷换概念？ |

### Step 4: Expand（来自 find skill）

没找到时逐步扩展：
1. 换更宽泛的关键词
2. 换同义词/相关词
3. 问"谁会知道这个"
4. 换信息源类型

---

## 使用示例

```bash
# 搜索李笑来《思考的真相》
openclaw tools search duckduck go "李笑来 思考的真相 豆瓣"

# 搜索并指定站点
openclaw tools search duckduck go "site:weixin.qq.com AI新闻"

# 搜索并排除
openclaw tools search duckduck go "AI进展 -广告 -推广"

# 获取搜索到的URL内容
web_fetch "https://example.com" --maxChars 3000
```

---

## 搜索质量自检清单

在交付结果前：

- [ ] 告诉了信息来源吗？
- [ ] 来源是最新的吗？
- [ ] 这个来源可信吗？
- [ ] 是用户要的内容吗，还是我理解错了？

---

## 常见问题

**Q: `online` 可以搜 Google 吗？**
A: 不可以。`online` 只能读文档，不能搜索互联网。

**Q: `web_fetch` 和 `online` 有什么区别？**
A: `web_fetch` 可以抓取任意网页正文，`online` 只能读官方文档。

**Q: 搜索失败怎么办？**
A: 先确认工具是否安装：`openclaw tools install-search`

---

## 参考

- yejinlei/web-search-ex-skill（clawhub）
- ivangdavila/find（clawhub）
