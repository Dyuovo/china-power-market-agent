# ACTION_REGISTRY.md

版本：v0.1.1  
状态：active  
最后更新：2026-05-22  
维护者：小枢

## 目的

Skill 说明“会做什么”，Action 说明“怎么被调用”。  
本文件为小枢路由、任务账本、A2A 转交和提交前自审提供统一动作卡。

## Action 字段

每个 Action 至少包含：

- `action_id`：稳定 ID。
- `owner_agent`：负责 Agent。
- `trigger`：何时调用。
- `inputs`：必要输入。
- `outputs`：预期输出。
- `command_or_prompt`：命令、Prompt 或入口文件。
- `permissions`：读写/外发/数据库/网络边界。
- `failure_mode`：失败时怎么报告、是否降级。
- `review_required`：提交前自审要求。
- `ledger_rule`：是否写任务账本。
- `memory_rule`：是否写长期记忆。

## Action 注册表

| action_id | owner_agent | trigger | outputs | ledger_rule | review_required |
|---|---|---|---|---|---|
| `xiaoshu.memory.archive` | xiaoshu | 每日 02:40 或手动修复记忆 | `06_DATA/99_memory/daily/`、`MEMORY_TREE.md` | cron 自动任务不逐条记账；异常时建账 | 是 |
| `xiaoshu.memory.reflect` | xiaoshu | 每日 02:45 | `reflections/`、`insights/` | 异常时建账 | 是 |
| `xiaoshu.memory.index` | xiaoshu | 每日 02:47 或记忆规则更新后 | `memory_index.sqlite` | 异常时建账 | 是 |
| `xiaoshu.health.monitor` | xiaoshu | 每日 03:00 或系统修复后 | 飞书健康报告 | 异常项建账 | 是 |
| `xiaoshu.backup.daily` | xiaoshu | 每日 23:30 或关键系统改动后 | `<backup_mount>/daily_YYYYMMDD/` | 失败时建账 | 是 |
| `xiaoshu.task.ledger` | xiaoshu | 跨日/跨 Agent/阻塞任务 | `task_ledger.sqlite`、`TASK_BOARD.md` | 本身即账本 | 是 |
| `xiaoting.daily.report` | xiaoting | 小霆日报 | Markdown/飞书日报 | cron 自动任务不逐条记账；失败时建账 | 是 |
| `xiaoting.pulse.policy` | xiaoting | 周二政策 Pulse | Markdown/飞书 Pulse | 失败或高风险结论建账 | 是 |
| `xiaoting.pulse.market_signal` | xiaoting | 周五市场信号 Pulse | Markdown/飞书 Pulse | 失败或连续错误建账 | 是 |
| `xiaodian.daily.learning_push` | xiaodian | 工作日早推 | 学习内容/每天一问/飞书文档 | 失败时建账 | 是 |
| `shared.feishu.doc.create` | xiaoshu/xiaoting/xiaodian | 需要给 owner 交付飞书云文档 | 可编辑飞书文档链接 | 授权失败时建账 | 是 |

## 详细动作卡

### xiaoshu.task.ledger

- owner_agent: xiaoshu
- trigger: 用户交办不能立即闭环、跨 Agent 协作、cron 失败、系统修复、长期任务
- inputs: 标题、负责人、来源、优先级、下一步
- outputs: `06_DATA/10_tasks/task_ledger.sqlite`、`06_DATA/10_tasks/TASK_BOARD.md`
- command_or_prompt:

```bash
python3 <workspace>/08_TOOLS/task_ledger.py add --title "..." --owner xiaoshu --source user
python3 <workspace>/08_TOOLS/task_ledger.py update TASK-YYYYMMDD-001 --status done --reported
```

- permissions: 可写 `06_DATA/10_tasks/`
- failure_mode: 若 SQLite 不可写，直接报告并保留手工 Markdown 记录
- review_required: 核对 owner、状态、输出路径、是否已回报
- ledger_rule: 所有非即时任务必须建账
- memory_rule: 完成系统级任务后更新 `MEMORY.md` 或 `MEMORY/STATE.md`

### xiaoshu.memory.index

- owner_agent: xiaoshu
- trigger: 每日 02:47；记忆规则、任务账本、自审规则更新后手动刷新
- inputs: `06_DATA/99_memory/`、各 Agent `MEMORY/`、根级运行文档
- outputs: `06_DATA/99_memory/memory_index.sqlite`
- command_or_prompt:

```bash
python3 <workspace>/08_TOOLS/memory_semantic_index.py index
```

- permissions: 只写记忆索引目录
- failure_mode: ChromaDB 不可用时自动降级 SQLite；SQLite 失败时建任务账本
- review_required: 查看 `status`，确认片段数非异常下降
- ledger_rule: 仅异常建账
- memory_rule: 重大规则变化后刷新索引

### xiaoshu.health.monitor

- owner_agent: xiaoshu
- trigger: 每日 03:00；系统修复后手动运行
- inputs: 数据库、cron、备份、记忆归档、记忆索引、任务账本、关键文件链接、残余会话
- outputs: 控制台健康报告、`00_GOVERNANCE/SYSTEM_DASHBOARD.md`、必要时自动创建任务账本项
- command_or_prompt:

```bash
python3 <workspace>/08_TOOLS/health_monitor.py
```

- permissions: 只读为主；只允许写系统看板和任务账本
- failure_mode: 发现 `❌` 或 `⚠️` 时按健康分区建任务账本，使用 `health_monitor:{section}` 防重复
- review_required: 区分真实故障和已修复待下次验证
- ledger_rule: 异常建账
- memory_rule: 重大修复后更新状态记忆

### shared.feishu.doc.create

- owner_agent: xiaoshu / xiaoting / xiaodian
- trigger: 需要把 Markdown 报告、学习推送、Pulse 或治理文档交付为飞书云文档
- inputs: Markdown 文件路径、文档标题、正确的 `--agent` 值
- outputs: 可编辑的 `https://bytedance.feishu.cn/docx/...` 链接
- command_or_prompt:

```bash
python3 <workspace>/08_TOOLS/create_feishu_doc.py --agent <xiaoshu|xiaoting|xiaodian> <markdown文件路径> "文档标题"
```

- permissions: 可调用飞书 docx/drive API；不得打印 secret、tenant token 或 app secret
- failure_mode: 未输出 `Edit access confirmed`、授权失败或脚本非零退出时，视为交付失败；不得把不可编辑链接回报给 owner
- review_required: 确认可编辑权限、正确 `--agent`、Markdown 源文件路径、最终 `Done!`
- ledger_rule: cron 或跨日任务中授权失败时建账
- memory_rule: 权限规则变更后更新治理记忆

## 与 Skill Registry 的关系

- Skill Registry 管“能力是否存在、归属谁、版本多少”。
- Action Registry 管“这次怎么调用、输入输出是什么、失败怎么办”。
- 新 Skill 进入 active 前，至少要补一张 Action 卡。

## 版本历史

| 版本 | 日期 | 变更 |
|---|---|---|
| v0.1.1 | 2026-05-22 | 新增飞书文档创建 Action，明确可编辑交付硬规则 |
| v0.1.0 | 2026-05-20 | 初始版本：高频 Action 注册和调用字段 |
