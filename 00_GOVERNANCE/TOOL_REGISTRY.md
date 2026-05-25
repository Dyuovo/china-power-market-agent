# TOOL_REGISTRY.md — 统一工具注册表

版本：v1.0  
状态：active  
创建：2026-05-25  
维护者：小枢  

## 说明

本注册表列出所有可调用工具的元信息。每个工具声明 `id`、`command`、`params`、`agents`（哪些 Agent 可调用）、`failure_mode`（失败时如何处理）。

各 Agent 的 TOOLS.md 引用本注册表，不各自维护路径。

---

## 工具清单

```yaml
tools:
  # ======================== 飞书 ========================
  - id: feishu.doc.create
    description: 创建飞书云文档（Markdown → 飞书 Doc）
    command: python3 08_TOOLS/create_feishu_doc.py
    params: {agent, file, title}
    agents: [xiaoshu, xiaoting, xiaodian]
    failure: 授权失败回报；不可编辑链接不得返回
    review: 是

  # ======================== 文档处理 ========================
  - id: doc.markitdown
    description: PDF/DOCX/PPTX/XLSX/HTML → Markdown 转换
    command: python3 08_TOOLS/markitdown_tool.py input.file -o output.md
    params: {input_file, output_file, stdout}
    agents: [xiaoshu, xiaoting, xiaodian]
    failure: 退化为纯文本读取
    review: 否

  - id: doc.ocr
    description: 图片文字识别（Tesseract）
    command: node 08_TOOLS/image_ocr.js <image_path> [lang]
    params: {image_path, lang}
    agents: [xiaoshu, xiaoting, xiaodian]
    failure: 报告无法识别
    review: 否

  # ======================== 记忆管理 ========================
  - id: memory.archive
    description: 压缩昨日 Agent 记忆为结构化记忆块
    command: python3 08_TOOLS/memory_archive.py compress
    params: {date, agent}
    agents: [xiaoshu]
    failure: 记录错误日志，不崩溃
    review: 否

  - id: memory.auto_extract
    description: 从会话存档自动提取事件，补全 MEMORY/daily（P0-2）
    command: python3 08_TOOLS/memory_auto_extract.py
    params: {agent, date, dry_run}
    agents: [xiaoshu]
    failure: 记录日志，保持占位符
    review: 否

  - id: memory.placeholder
    description: 检查昨日 daily，空则补占位记录（自动联动 auto_extract）
    command: python3 08_TOOLS/daily_placeholder.py
    params: {date}
    agents: [xiaoshu]
    failure: 记录日志
    review: 否

  - id: memory.reflect
    description: 扫描近 3 天记忆块，生成反思报告和主动洞察
    command: python3 08_TOOLS/nightly_reflect.py
    params: {date, days}
    agents: [xiaoshu]
    failure: 记录日志
    review: 否

  - id: memory.index
    description: 刷新长期记忆语义检索索引
    command: python3 08_TOOLS/memory_semantic_index.py index
    params: {}
    agents: [xiaoshu]
    failure: 保留旧索引
    review: 否

  - id: memory.search
    description: 跨 Agent 长期记忆语义检索
    command: python3 08_TOOLS/memory_semantic_index.py search "<query>" -n 5
    params: {query, agent, n}
    failure: 返回空结果，提示手动检索
    review: 否

  # ======================== 任务管理 ========================
  - id: task.ledger
    description: 任务账本 CRUD
    command: python3 08_TOOLS/task_ledger.py <action>
    params: {action, title, owner, source, status, output}
    failure: 创建 TASK_ASSIGN 失败时回报
    review: 是（done/cancel 状态变更时）

  - id: task.review
    description: 任务账本周/月复盘
    command: python3 08_TOOLS/task_review.py <weekly|monthly>
    params: {period}
    agents: [xiaoshu]
    failure: 记录日志
    review: 是

  # ======================== 健康监控 ========================
  - id: health.monitor
    description: 系统健康检查，刷新看板，自动建账
    params: {no_ledger}
    agents: [xiaoshu]
    failure: 异常项建任务账本（health_monitor:{section} 防重复）
    review: 是（外发时）

  - id: health.db_manager
    params: {action, date, days}
    failure: 记录日志
    review: 否

  # ======================== 备份 ========================
  - id: backup.daily
    description: 每日增量备份到备份盘
    command: bash 08_TOOLS/backup_daily.sh
    params: {}
    agents: [xiaoshu]
    failure: 记录错误日志，跳过不崩溃
    review: 否

  - id: backup.weekly
    description: 每周全量快照到备份盘
    command: bash 08_TOOLS/backup_weekly.sh
    params: {}
    agents: [xiaoshu]
    failure: 记录错误日志
    review: 否

  # ======================== 数据同步 ========================
  - id: data.sync
    description: 跨环境数据同步
    command: python3 08_TOOLS/sync_data.py
    params: {}
    agents: [xiaoshu]
    failure: 记录日志
    review: 否

  - id: data.migrate
    description: 数据迁移（旧结构 → 新结构）
    command: python3 08_TOOLS/migrate_data.py
    params: {source, target}
    agents: [xiaoshu]
    failure: 保留回滚路径
    review: 是

  # ======================== 学习固化 ========================
  - id: learning.grade_to_db
    description: 批改结果强制写入学习数据库（小电）
    command: python3 03_XIAODIAN/RUNTIME/power_market_learning_agent/tools/grade_to_db.py
    params: {task_id, task_title, task_tags, report_date, json}
    agents: [xiaodian]
    failure: 批改视为未完成；健康监控次日检测
    review: 是（每次运行后确认 attempts 增加）
```

---

## 调用规则

- **Agent 调用自身工具**：直接按 command 执行
- **跨 Agent 调用**：通过 TASK_ASSIGN 分派
- **工具路径变更**：只改本注册表，各 Agent TOOLS.md 不独立维护路径
- **新增工具**：必须登记到此注册表 + 写入即实测

## 与 ACTION_REGISTRY 的关系

`ACTION_REGISTRY.md` 记录「怎么调度」——trigger、permissions、review_required  
`TOOL_REGISTRY.md` 记录「怎么运行」——command、params、agents、failure_mode

两者互补，不重复。

## 版本历史

| 版本 | 日期 | 变更 |
|---|---|---|
| v1.0 | 2026-05-25 | 初始版本，登记 18 个工具 |
