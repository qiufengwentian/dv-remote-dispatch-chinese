# Domain 文档

工程类技能在探索本代码库时，应如何消费本仓库的领域文档。

## 探索前先读

- 仓库根目录的 **`CONTEXT.md`**；或
- 若根目录存在 **`CONTEXT-MAP.md`**：它指向各自的 `CONTEXT.md`（每个上下文一份）。阅读与主题相关的每一份。
- **`docs/adr/`**：阅读涉及你即将改动区域的 ADR。在多上下文仓库中，还要检查 `src/<context>/docs/adr/` 里与上下文相关的决策。

若这些文件不存在，**静默继续**。不要提示它们的缺失，也不要主动建议创建。`/domain-modeling` 技能（经 `/grill-with-docs` 与 `/improve-codebase-architecture` 触达）会在术语或决策真正落定时懒创建这些文件。

## 文件结构

单上下文仓库（大多数仓库）：

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `CONTEXT-MAP.md`）：

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← 系统级决策
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← 上下文内决策
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

## 使用词汇表的词汇

当你的输出命名某个领域概念（issue 标题、重构提案、假设、测试名）时，使用 `CONTEXT.md` 中定义的术语，不要漂移到词汇表明确避免的同义词。

若你需要的概念尚不在词汇表中，这是一个信号：要么你在发明项目并不使用的语言（请重新考虑），要么存在真实缺口（记下来交给 `/domain-modeling`）。

## 标记 ADR 冲突

若你的输出与既有 ADR 矛盾，显式指出，而非静默覆盖：

> _与 ADR-0007（事件溯源订单）矛盾，但值得重新讨论，因为……_
