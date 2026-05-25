# DATABASE_CONVENTION.md

版本：v0.1.0  
状态：active  
最后更新：2026-05-14  
制定者：小枢  
适用范围：所有 Agent（小霆、小电、小枢）

## 目的

统一 AI_SYSTEM 内所有 SQLite 数据库的命名、路径、注册和更新规范，防止数据库碎片化。

## 命名规范

| 规则 | 说明 |
|------|------|
| 文件后缀 | 统一使用 `.db`（新库），已有 `.sqlite` 暂不强制改名 |
| 命名格式 | `{领域}_{用途}.db`，如 `ai_pm_knowledge.db`、`power_market_learning.db` |
| 禁止 | 不使用中文文件名、空格、特殊字符 |

## 路径规范

```
{Agent工作区}/RUNTIME/{项目目录}/knowledge_base/{数据库名}.db
```

示例：

- ✅ `03_XIAODIAN/RUNTIME/AI_PM_KNOWLEDGE_ARCHIVE/knowledge_base/ai_pm_knowledge.db`
- ✅ `03_XIAODIAN/RUNTIME/power_market_learning_agent/knowledge_base/learning.sqlite`（已有，暂不改名）
- ❌ `03_XIAODIAN/RUNTIME/AI_PM_KNOWLEDGE_ARCHIVE/ai_pm_knowledge.db`（不应放在归档根目录）

## 注册要求

**每个新建数据库必须登记到：**

1. `06_DATA/00_registry/DATA_SOURCE_REGISTRY.md` — 数据源注册表
2. 该数据库的 Schema 文档 — 放在 `knowledge_base/` 同级目录的 `README.md` 或专用文档

登记内容必须包含：

- 数据库路径
- 建库脚本路径
- 表结构摘要
- 更新方式（全量重建 / 增量追加 / 手动）
- 负责人

## 建库前置审批

**所有 Agent 新建数据库前，必须先和小枢（治理层）沟通确认：**

1. 数据库是否和已有数据库功能重叠？
2. 路径和命名是否符合本规范？
3. Schema 设计是否合理？
4. 更新策略是什么？

未经小枢审批的数据库视为非正式资产，不被纳入数据治理范围。

## 更新策略

| 策略 | 适用场景 | 要求 |
|------|----------|------|
| 全量重建 | 数据源每次完整覆盖（如知识星球归档重抓） | 保留旧版本备份，脚本可复现 |
| 增量追加 | 持续增长的数据（如学习记录、新帖子） | 必须保证幂等（重复运行不产生重复数据） |
| 手动 | 偶尔更新 | 更新后必须在 DATA_UPDATE_LOG.md 登记 |

## FTS 索引维护

- 使用 FTS5 的数据库，FTS 索引需与主表保持同步。
- 全量重建模式：build 脚本末尾统一重建 FTS。
- 增量模式：每次 INSERT/UPDATE/DELETE 后同步更新 FTS 表。
- 定期运行 `INSERT INTO posts_fts(posts_fts) VALUES('rebuild')` 优化索引。

## 备份策略

- 每次全量重建前，自动备份旧库为 `{库名}.bak.{日期}.db`
- 增量更新的库，每周手动备份一次

## 当前已注册数据库

| 数据库 | 位置 | 建库脚本 | 更新方式 | 负责人 |
|--------|------|----------|----------|--------|
| ai_pm_knowledge.db | `03_XIAODIAN/RUNTIME/AI_PM_KNOWLEDGE_ARCHIVE/knowledge_base/` | `build_db.py` | 全量重建 | 小电 |
| learning.sqlite | `03_XIAODIAN/RUNTIME/power_market_learning_agent/knowledge_base/` | 学习系统自动创建 | 增量追加 | 小电 |
