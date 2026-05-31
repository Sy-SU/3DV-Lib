# Retrieval Queue

待检索论文通过以下 Obsidian Base 视图管理：

- [[Retrieval Queue.base]]

## 选择规则

每批最多处理 5 篇论文。

优先级顺序：

1. `priority: high`
2. `priority: medium`
3. `priority: low`

同一优先级下，状态顺序：

1. `needs-review`
2. `reading`
3. `skimmed`
4. `unread`
5. `reference-only`

跳过：

```yaml
summary_level: full
```

## 执行规则

1. 每篇论文只维护一个主笔记。
2. 优先使用本地 PDF。
3. 只读到摘要时，不允许生成完整总结。
4. 每批结束后更新 YAML Properties。
5. 每批结束后更新 `Retrieval Log.md`。
6. 日志中的论文名称使用普通文本，不使用 Wikilink。
7. 每批结束后立即停止，等待人工检查。
