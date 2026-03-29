---
name: memory-updater
description: 更新小龙女的记忆上下文。触发时机：每天早7:40 auto-updater 调用，或用户要求读取今日context。
---

# memory-updater

每天早上运行，为当日报表准备上下文信息。

## 输入
- 今天日期（Asia/Shanghai 时区）
- 长沙天气（使用 weather skill）
- 昨日 memory/YYYY-MM-DD.md
- MEMORY.md 中的用户常量

## 输出
生成 `memory/today-context.md`，包含：

```markdown
# 今日 Context | YYYY-MM-DD

## 基本信息
- 日期：YYYY年MM月DD日 星期X
- 天气：[天气情况]
- 节气：[节气名（如果快到了）]

## 昨日回顾（YYYY-MM-DD）
- 6课题各一句话总结
- 关键事件（日报发出/飞书同步等）

## 用户常量（从 MEMORY.md）
- 家庭：洋洋（妻）、麦兜（大儿）、当当（小女，2岁+）
- 所在地：长沙
- 作息：22:00睡/6:00起
- 6课题：健康/成长/能量/事业/关系/感恩

## 今日选题参考
- 最近AI新闻热点：[根据近期新闻]
- 待研究课题方向：[查6课题数据库]
- 待推荐人物：[查传记数据库]
```

## 执行步骤

1. 获取日期：`date "+%Y年%m月%d日 %A"`（Asia/Shanghai）
2. 获取长沙天气：`curl "wttr.in/Changsha?format=%l:+%c+%t+(feels+like+%f),+%w+wind,+%h+humidity"`
3. 读昨日 memory 文件（如存在）
4. 读 MEMORY.md 提取用户常量
5. 合并写入 `memory/today-context.md`

## 注意事项

- 如果昨日文件不存在，跳过"昨日回顾"
- 如果天气获取失败，写"天气待查"
- 判断节气：清明4月4日，谷雨4月20日
