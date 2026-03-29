---
name: feishu-sync
description: 将 Obsidian 日报同步到飞书文档（完整 markdown 格式保留）。触发时机：日报生成后同步到飞书"每日日报·AI学习资料库"。
---

# feishu-sync

将 Obsidian 日报同步到飞书"每日日报·AI学习资料库"文件夹，**完整保留 markdown 格式**。

---

## ⚠️ 核心原则（2026-03-27 验证通过）

**feishu_doc 工具只能在飞书渠道消息触发时可用**。主 agent 需要用 **Python API 方式**写入。

---

## 方法一：feishu_doc 工具（仅飞书消息渠道可用）

```
action: create_doc  → 创建文档
action: write       → 写入 markdown（飞书自动转换格式）
action: update_public → 设置公开权限
```

---

## 方法二：Python API（通用方法，主 agent 直接用这个）

### 完整流程

**Step 1**: 读取 Obsidian markdown 文件

路径格式：`/Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档/YYYY/M月/YYYY-MM-DD-日报-关键词.md`

**Step 2**: 用 convert API 把 markdown 转换成 Feishu blocks

```
POST https://open.feishu.cn/open-apis/docx/v1/documents/blocks/convert
Body: {"content_type": "markdown", "content": "<完整markdown>"}
返回: {"data": {"blocks": [...], "first_level_block_ids": [...]}}
```

**Step 3**: 创建新文档

```
POST https://open.feishu.cn/open-apis/docx/v1/documents
Body: {"title": "YYYY-MM-DD 日报 | 关键词1 · 关键词2 · 关键词3", "folder_token": "BJFRf2lOQlZLQSdFsx1cL4ynnQd"}
```

**Step 4**: 逐个插入 blocks（关键！批量插入会400错误）

```
POST https://open.feishu.cn/open-apis/docx/v1/documents/{doc_token}/blocks/{doc_token}/children
Body: {"children": [block], "index": -1}
```

- **batch_size=1 成功率最高**，但每批 10 个也可以（部分会失败）
- 每次请求间隔 **1.2-1.5 秒**
- 按 `first_level_block_ids` 顺序插入

**Step 5**: 设置公开权限

```
PATCH https://open.feishu.cn/open-apis/drive/v1/permissions/{doc_token}/public?type=docx
Body: {"link_share_entity": "anyone_readable", "security_entity": "anyone_can_view"}
```

---

## 完整可运行脚本模板

```python
#!/usr/bin/env python3
import urllib.request, json, time

APP_ID = 'cli_a93862a9a6f8dbd4'
APP_SECRET = 'RTyTroX4ZUQv18gySo8ZbbCrn5AunbTs'
FOLDER = 'BJFRf2lOQlZLQSdFsx1cL4ynnQd'

def get_token():
    url = 'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal'
    data = json.dumps({'app_id': APP_ID, 'app_secret': APP_SECRET}).encode()
    resp = urllib.request.urlopen(urllib.request.Request(url, data=data, headers={'Content-Type': 'application/json'}), timeout=10)
    return json.loads(resp.read()).get('tenant_access_token', '')

def main():
    token = get_token()
    headers = {'Authorization': 'Bearer ' + token, 'Content-Type': 'application/json'}
    
    # 1. 读取 markdown 文件
    filepath = '/Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档/YYYY/M月/文件名.md'
    with open(filepath, 'r', encoding='utf-8') as f:
        md_content = f.read()
    
    # 2. Convert markdown → blocks
    convert_url = 'https://open.feishu.cn/open-apis/docx/v1/documents/blocks/convert'
    payload = json.dumps({'content_type': 'markdown', 'content': md_content}, ensure_ascii=False).encode('utf-8')
    req = urllib.request.Request(convert_url, data=payload, headers=headers, method='POST')
    resp = urllib.request.urlopen(req, timeout=60)
    result = json.loads(resp.read())
    blocks = result.get('data', {}).get('blocks', [])
    first_ids = result.get('data', {}).get('first_level_block_ids', [])
    print(f'Converted: {len(blocks)} blocks, {len(first_ids)} top-level')
    
    # 3. 创建文档
    create_url = 'https://open.feishu.cn/open-apis/docx/v1/documents'
    create_data = json.dumps({'title': '标题', 'folder_token': FOLDER}).encode()
    create_req = urllib.request.Request(create_url, data=create_data, headers=headers)
    create_resp = urllib.request.urlopen(create_req, timeout=10)
    doc_token = json.loads(create_resp.read()).get('data', {}).get('document', {}).get('document_id', '')
    
    # 4. 按 first_level_block_ids 顺序逐个插入
    block_map = {b['block_id']: b for b in blocks}
    top_blocks = [block_map[fid] for fid in first_ids if fid in block_map]
    
    insert_url = f'https://open.feishu.cn/open-apis/docx/v1/documents/{doc_token}/blocks/{doc_token}/children'
    ok, fail = 0, 0
    for i, block in enumerate(top_blocks):
        pl = json.dumps({'children': [block]}, ensure_ascii=False).encode('utf-8')
        rq = urllib.request.Request(insert_url, data=pl, headers=headers, method='POST')
        try:
            rp = urllib.request.urlopen(rq, timeout=20)
            rs = json.loads(rp.read())
            if rs.get('code') == 0: ok += 1
            else: fail += 1
        except: fail += 1
        if (i+1) % 50 == 0: print(f'Progress: {i+1}/{len(top_blocks)}')
        time.sleep(1.2)
    
    # 5. 设置公开
    purl = f'https://open.feishu.cn/open-apis/drive/v1/permissions/{doc_token}/public?type=docx'
    pdata = json.dumps({'link_share_entity': 'anyone_readable', 'security_entity': 'anyone_can_view'}).encode()
    pr = urllib.request.Request(purl, data=pdata, method='PATCH', headers={'Authorization': 'Bearer ' + token, 'Content-Type': 'application/json'})
    urllib.request.urlopen(pr, timeout=10)
    
    print(f'Done: {ok} OK, {fail} failed')
    print(f'Link: https://pcnjhs1bymqa.feishu.cn/docx/{doc_token}')

if __name__ == '__main__':
    main()
```

---

## 已知限制与解决方案

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 批量插入（batch_size>1）大量 400 | API 对部分 block type 有限制 | batch_size=1 或 10，逐个插入 |
| type 31（Table）插入失败 | Feishu API 不支持 | 过滤掉 table blocks（内容丢失） |
| type 32（TableCell）插入失败 | 同上 | 过滤掉 |
| 整批失败但单个成功 | 批里有不支持的 block | 用逐个插入绕过 |
| `import_tasks` API 返回 `point is required` | 需要 user_access_token | 用 convert API 代替 |

---

## 格式保留情况

| 格式 | 状态 |
|------|------|
| 标题（# ## ###） | ✅ 完美 |
| 粗体（**bold**） | ✅ 完美 |
| 链接（[文字](URL)） | ✅ 完美 |
| 列表（- bullet） | ✅ 完美 |
| 有序列表（1.） | ✅ 完美 |
| 引用（> quote） | ✅ 完美 |
| 分隔线（---） | ✅ 完美 |
| emoji | ✅ 完美 |
| 表格 | ❌ 转为普通文本（API限制） |

---

## 配置（固定不变）

| 配置项 | 值 |
|--------|-----|
| folder_token | `BJFRf2lOQlZLQSdFsx1cL4ynnQd` |
| 文档链接前缀 | `https://pcnjhs1bymqa.feishu.cn/docx/` |
| 日报文件路径 | `/Users/maoruiwen/Documents/小龙女/小龙女/1-数据库/日报存档/`（嵌套路径，vault里有vault） |

---

## 标题格式

`YYYY-MM-DD 日报 | 关键词1 · 关键词2 · 关键词3`

示例：`2026-03-27 日报 | Sora关闭 · 可灵活了 · 王阳明传`

---

## Skill 迭代原则

**遇到新问题 → 立即更新 skill → 保持灵活，持续迭代。**
