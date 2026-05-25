# TOOLS.md

版本:v0.5.2
状态:active
最后更新:2026-05-25
变更：小电学习数据库批改强制入库工具上线（grade_to_db.py）；健康监控新增 learning.sqlite 一致性检查

## 工具使用原则

1. 优先读实际文件和配置,不凭记忆判断系统状态。
2. 先做只读调查,再做配置改动。
3. 修改运行配置前保留备份。
4. 对 Markdown 文档用清晰索引和版本记录维护。
5. 对 JSON 配置用结构化读写,避免手工破坏格式。
6. **写入即实测**:凡在 TOOLS.md 或 Skill Registry 里写「可用」,必须先端到端跑通一次。不测不落笔。

## 当前关键环境

- 新治理入口:`<workspace>`
- OpenClaw 工程:`<openclaw_repo>`
- OpenClaw 数据:`<openclaw_data_dir>`
- 小霆运行区:`<workspace>/02_XIAOTING`
- 小电运行区:`<workspace>/03_XIAODIAN`
- 电力市场系统:`<workspace>/02_XIAOTING/POWER_MARKET_AGENT`
- 飞书文档创建工具:`<workspace>/08_TOOLS/create_feishu_doc.py` - 当前支持已配置文档凭据的 `xiaoshu` / `xiaoting` / `xiaodian`

## 记忆归档系统

灵感来自 OpenHuman (Neocortex + Subconscious Loop)，2026-05-18 上线。

### 记忆压缩
```bash
# 压缩昨天所有 Agent 的记忆
python3 08_TOOLS/memory_archive.py compress

# 压缩指定日期/Agent
python3 08_TOOLS/memory_archive.py compress --date 2026-05-18 --agent xiaoting

# 重建记忆树索引
python3 08_TOOLS/memory_archive.py build-tree

# 查看状态
python3 08_TOOLS/memory_archive.py status
```

### 潜意识扫描
```bash
# 夜间反思扫描（默认近3天）
python3 08_TOOLS/nightly_reflect.py

# 指定日期和范围
python3 08_TOOLS/nightly_reflect.py --date 2026-05-18 --days 7
```

### Daily 占位
```bash
# 检查昨日各Agent MEMORY/daily，空则补占位记录
python3 08_TOOLS/daily_placeholder.py

# 指定日期
python3 08_TOOLS/daily_placeholder.py --date 2026-05-17
```

### 输出位置
- 记忆块：`06_DATA/99_memory/daily/{agent}/`
- 记忆树索引：`06_DATA/99_memory/MEMORY_TREE.md`
- 反思报告：`06_DATA/99_memory/reflections/`
- 主动洞察：`06_DATA/99_memory/insights/`
- Schema：`06_DATA/99_memory/SCHEMA.md`

### 长期记忆检索

多个 Agent 的长期记忆统一进入本地检索索引。默认不需要 Ollama，也不需要在线模型；当前环境没有 `chromadb` 时自动使用 SQLite + 本地哈希向量。

```bash
# 重建索引
python3 08_TOOLS/memory_semantic_index.py index

# 查看索引状态
python3 08_TOOLS/memory_semantic_index.py status

# 跨 Agent 搜索
python3 08_TOOLS/memory_semantic_index.py search "backup drive 备份 掉挂载" -n 5

# 限定某个 Agent
```

索引位置：
- SQLite：`06_DATA/99_memory/memory_index.sqlite`
- ChromaDB 可选集合：`06_DATA/99_memory/chroma_data` / `openclaw_memory`

使用规则：凡涉及“之前说过什么、跨日连续任务、历史故障、用户偏好、agent 旧产出”，先检索长期记忆，再回答。

## 任务账本

跨日任务、跨 Agent 任务、cron 失败、系统建设任务、用户明确交办但不能立即闭环的事项，都进入任务账本。

```bash
# 初始化/生成看板
python3 08_TOOLS/task_ledger.py init

# 新增任务
python3 08_TOOLS/task_ledger.py add --title "任务标题" --owner xiaoshu --source user

# 更新任务
python3 08_TOOLS/task_ledger.py update TASK-YYYYMMDD-001 --status done --output "结果路径" --reported

# 查看任务
python3 08_TOOLS/task_ledger.py list
python3 08_TOOLS/task_ledger.py report

# 生成 A2A 标准消息
python3 08_TOOLS/task_ledger.py a2a-template TASK-YYYYMMDD-001 --to xiaoting
```

输出位置：
- SQLite：`06_DATA/10_tasks/task_ledger.sqlite`
- 看板：`06_DATA/10_tasks/TASK_BOARD.md`
- 周/月复盘：`06_DATA/10_tasks/reviews/`

复盘命令：

```bash
python3 08_TOOLS/task_review.py weekly
python3 08_TOOLS/task_review.py monthly
```

治理规则：
- 提交前自审：`00_GOVERNANCE/PRE_SUBMIT_REVIEW.md`
- A2A 任务协议：`00_GOVERNANCE/AGENT_MSG_SPEC.md`
- Action 注册：`00_GOVERNANCE/ACTION_REGISTRY.md`
- 任务复盘：`00_GOVERNANCE/TASK_LEDGER_REVIEW.md`

## 系统看板与健康自动建账

系统健康监控会刷新统一看板，并把 warning/error 自动写入任务账本。

```bash
# 运行健康检查，刷新 SYSTEM_DASHBOARD.md，并自动建账
python3 08_TOOLS/health_monitor.py

# 只看健康，不写任务账本
python3 08_TOOLS/health_monitor.py --no-ledger
```

输出位置：
- 看板：`00_GOVERNANCE/SYSTEM_DASHBOARD.md`
- 自动任务：`06_DATA/10_tasks/task_ledger.sqlite`

防重复规则：同一健康分区只保留一个未闭环异常任务，例如 `health_monitor:file_links`。

## 禁止动作

- 不泄露密钥。
- 不自动交易。
- 不未经确认删除重要数据。
- 不把配置、凭据、会话缓存纳入公开知识库。

## 飞书云文档

### 实际可用工具:Python 脚本

飞书文档创建工具是 `08_TOOLS/create_feishu_doc.py`,已验证可用(2026-05-15)。当前支持已配置文档凭据的 `xiaoshu` / `xiaoting` / `xiaodian` 三个账号:

```bash
# 小枢
python3 <workspace>/08_TOOLS/create_feishu_doc.py --agent xiaoshu <markdown文件路径> [文档标题]

# 小霆
python3 <workspace>/08_TOOLS/create_feishu_doc.py --agent xiaoting <markdown文件路径> [文档标题]

# 小电
python3 <workspace>/08_TOOLS/create_feishu_doc.py --agent xiaodian <markdown文件路径> [文档标题]
```

### 使用步骤

1. 先把要推的内容写成完整 Markdown 文件，保存到本 Agent 的输出目录
2. 调用脚本时务必带 `--agent` 参数，文档会创建在对应 Agent 名下
3. 脚本会自动从 `<feishu_editor_openids>`、`<feishu_agent_owner_openid>` 和 `AI_SYSTEM/openclaw.json` 的飞书 allowlist 收集 owner 的 open_id，并授予 `full_access`
4. 只有脚本输出 `Edit access confirmed` 且最终 `Done!` 时，才算文档交付成功
5. 脚本返回 `doc_id` 和飞书文档链接（`https://bytedance.feishu.cn/docx/...`）
6. 把链接返回给用户

### 凭据

每个 Agent 有独立的飞书应用凭据（`<feishu_app_id>/<feishu_app_secret>` / `<feishu_xiaoshu_*>` / `<feishu_xiaodian_*>`），Gateway 环境均已配置。

### 限制

- 不支持表格、图片、富文本内联格式
- 自动剥离 Markdown 文件开头的 YAML front matter
- 代码块会按逐行文本写入
- 大文档分批写入(每批 50 block),有 429 限流重试
- 提供给 owner 的飞书文档必须可编辑；如果授权失败或脚本非零退出，不得把链接作为成功结果返回

### 禁止

- 不要用 exec curl 手搓飞书 API
- 不要读取或打印 `<feishu_credentials>` secret、`tenant_access_token`、`app_secret`
- 不要把 feishu_doc 当作 OpenClaw 内置工具--它不存在,用上面的 Python 脚本

## 图片 OCR

DeepSeek V4 Pro 无视觉能力，收到图片时用 OCR 提取文字：

```bash
node <workspace>/08_TOOLS/image_ocr.js <图片路径> [语言]
# 或
bash <workspace>/08_TOOLS/image_ocr.sh <图片路径> [语言]
```

默认语言 `chi_sim+eng`（简体中文+英文）。详见 `05_SHARED_SKILLS/image_ocr_skill.md`。

## MarkItDown 文档转换

Microsoft MarkItDown，将 PDF、DOCX、PPTX、XLSX、HTML、EPUB 等格式转为 Markdown。2026-05-18 已安装（`markitdown[all]` v0.1.5）。

### 使用方式

```bash
# 文件转换
python3 <workspace>/08_TOOLS/markitdown_tool.py input.pdf -o output.md

# 输出到 stdout
python3 <workspace>/08_TOOLS/markitdown_tool.py input.docx --stdout

# 管道输入
cat input.html | python3 <workspace>/08_TOOLS/markitdown_tool.py -o output.md
```

### 支持格式

PDF、DOCX、PPTX、XLSX、HTML、图片（OCR）、音频（需 ffmpeg）、EPUB、ZIP、CSV、JSON、XML 等。

### 已知限制

- 音频转换需要 ffmpeg（当前未装），不影响文档格式
- 包装器脚本自动处理 venv 路径和 fallback（无 markitdown 时退化为纯文本读取）

## ChromaDB 向量数据库

电力市场知识库语义检索（961 条文档，集合 `power_market_kb`）：

```python
import chromadb
client = chromadb.PersistentClient(path="<workspace>/02_XIAOTING/POWER_MARKET_AGENT/knowledge_base/chroma_data")
col = client.get_collection("power_market_kb")
results = col.query(query_texts=["查询词"], n_results=5)
```

当前 `pm-knowledge-chroma` Docker 容器运行中。长期记忆检索默认不依赖此 ChromaDB；如本地 Python 环境未安装 `chromadb`，先使用 `memory_semantic_index.py` 的 SQLite 检索。
需要直接访问 `power_market_kb` 时，先确认当前运行环境具备 `chromadb` 包；WSL 环境如遇只读错误，先 `cp -r chroma_data /tmp/` 再访问。

## 小电学习数据库 — 批改强制入库（2026-05-25 上线）

`learning.sqlite` 记录了所有学习任务、题目、答题记录和错题。**每生成一份 V3 批改版，必须同步写入数据库。**

### 工具位置

```bash
03_XIAODIAN/RUNTIME/power_market_learning_agent/tools/grade_to_db.py
```

### 用法

```bash
python3 03_XIAODIAN/RUNTIME/power_market_learning_agent/tools/grade_to_db.py \
  --task-id "push-YYYY-MM-DD" \
  --task-title "主题描述" \
  --task-tags '["tag1","tag2"]' \
  --report-date "YYYY-MM-DD" \
  --json '[{"question_id":"q-id","stem":"题","type":"concept","knowledge_tag":"tag","difficulty":2,"reference_answer":"答","user_answer":"主人答","score":4.0,"max_score":5,"strength":"亮点","gap":"待加强","mistake_type":null}]'
```

### 自动处理

- 插入/更新 `learning_tasks`、`questions`、`attempts`
- 得分 <90% 或明确有 gap → 自动创建 `mistakes`
- 更新对应 `knowledge_tag` 的 `review_queue`

### 规则

- **不跑这一步 = 批改没有入库 = 任务未完成**
- 运行后检查输出确认 `attempts` 行数增加
- 可用 knowledge_tag：`spot_price_basics` `congestion_management` `medium_long_term` `deviation_assessment` `market_structure` `green_power_direct_connection` `policy` `business_model_canvas` `ai_pm` `rag`

### 数据库位置

| 数据库 | 路径 | 用途 |
|---|---|---|

# 单日汇总

# 7日周报

# 数据库校验

# 备份（保留7份）
```

### 写入数据

- 小枢负责校验、备份、汇总
