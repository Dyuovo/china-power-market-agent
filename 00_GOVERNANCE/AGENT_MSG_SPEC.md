# AGENT_MSG_SPEC.md - Agent 间消息格式规范

版本：v0.3  
状态：active  
最后更新：2026-05-20  
维护者：小枢

## 目的

跨 Agent 任务必须绑定任务账本，避免“交出去了但没人记账”。

## 当前通道

OpenClaw A2A 已启用：

```text
tools.agentToAgent.enabled = true
tools.sessions.visibility = all
```

当前常用 sessionKey：

| Agent | sessionKey |
|---|---|
| 小枢 | `agent:main:feishu:direct:<owner_open_id>` |
| 小霆 | `agent:xiaoting:feishu:direct:<xiaoting_open_id>` |
| 小电 | `agent:xiaodian:feishu:direct:<xiaodian_open_id>` |
| 小衡 | `agent:xiaoheng:feishu:direct:<xiaoheng_open_id>` |

## 任务账本联动

所有跨 Agent 的执行任务必须先创建或引用任务账本：

```bash
python3 <workspace>/08_TOOLS/task_ledger.py add --title "任务标题" --owner xiaoting --source agent
python3 <workspace>/08_TOOLS/task_ledger.py a2a-template TASK-YYYYMMDD-001 --to xiaoting
```

任务完成、阻塞、取消时，发送方或接收方必须更新账本状态。

## 标准消息类型

从 v0.3 起，协作任务优先使用以下四种类型：

| 类型 | 用途 | 必须更新账本 |
|---|---|---|
| `TASK_ASSIGN` | 小枢或执行 Agent 分派任务 | 是 |
| `TASK_ACK` | 接收方确认收到并说明是否接受 | 是 |
| `TASK_DONE` | 任务完成，给出结果位置和摘要 | 是 |
| `TASK_BLOCKED` | 任务阻塞，给出原因和下一步 | 是 |

普通通知仍可使用 `[业务通知]`、`[知识速查]`、`[状态同步]`，但只要需要后续行动，就升级为 `TASK_ASSIGN`。

## 标准消息字段

每条 A2A 消息建议包含：

```text
类型: [TASK_ASSIGN] | [TASK_ACK] | [TASK_DONE] | [TASK_BLOCKED]
任务ID: TASK-YYYYMMDD-NNN
标题: 一句话任务名
状态: open | doing | blocked | done | canceled
优先级: low | normal | high | urgent
期望输出: 文件路径 / 飞书消息 / 数据库写入 / 简短回复
截止: YYYY-MM-DD HH:MM / 无
上下文: 相关文件、记忆检索结果、用户原话摘要
风险: low | medium | high
回报要求: 完成后 TASK_DONE；阻塞时 TASK_BLOCKED
data: {...}  ← 可选 JSON，结构化传递关键参数
```

### data 字段（结构化参数）

`TASK_ASSIGN` 和 `TASK_DONE` 消息**应包含** `data` JSON 字段，让接收方无需从自然语言解析参数：

```json
{
  "province": "江苏",
  "report_type": "policy_pulse",
  "template": "prompts/policy_pulse.md",
  "data_source": "近7天政策抓取",
  "output_dir": "outputs/policy_pulse/",
  "deadline": "2026-05-20T18:00+08:00"
}
```

规则：
- Markdown 字段给人读，`data` 给 Agent 机读
- `data` 可选，但 `TASK_ASSIGN` 强烈建议包含
- 字段名用 snake_case，值可嵌套
- 常用字段：`province`、`report_type`、`template`、`data_source`、`output_dir`、`deadline`、`task_id_list`

## 标准示例

### TASK_ASSIGN

```text
类型: [TASK_ASSIGN]
任务ID: TASK-20260520-001
来源: 小枢
目标: 小霆
标题: 生成江苏政策 Pulse
状态: open
优先级: high
期望输出: 保存 Markdown 并飞书发送
截止: 2026-05-20 18:00
上下文: prompts/policy_pulse.md；近 7 天政策抓取数据
风险: medium
回报要求: 完成后 TASK_DONE；阻塞时 TASK_BLOCKED
data: {"province":"江苏","report_type":"policy_pulse","template":"prompts/policy_pulse.md","data_source":"近7天政策抓取"}
```

### TASK_ACK

```text
类型: [TASK_ACK]
来源: 小霆
目标: 小枢
任务ID: TASK-20260520-001
状态: doing
结果: 已收到，预计 18:00 前完成。
```

### TASK_DONE

```text
类型: [TASK_DONE]
来源: 小霆
目标: 小枢
任务ID: TASK-20260520-001
状态: done
结果: 已完成江苏政策 Pulse。
产出: <workspace>/02_XIAOTING/...
风险: 已标注数据源和发布日期。
data: {"output_file":"outputs/policy_pulse_20260520.md","feishu_url":"https://bytedance.feishu.cn/docx/...","key_findings":["发现1","发现2"]}
```

### TASK_BLOCKED

```text
类型: [TASK_BLOCKED]
来源: 小霆
目标: 小枢
任务ID: TASK-20260514-001
状态: blocked
阻塞原因: 数据源 403，无法获取官方公告。
下一步: 需要小枢确认是否改用手动上传文件。
```

## 普通通知类型

不要求后续行动的消息可以继续使用旧类型：

```text
类型: [状态同步]
来源: 小霆
目标: 小枢, 小电
内容: 今日江苏日报已完成，已更新 STATE.md。
期望: 仅通知
```

## 频率限制

- 业务通知：每天不超过 3 条。
- 知识速查：每天不超过 2 条。
- TASK_ASSIGN：按用户触发或小枢拆解触发。
- 状态同步：优先放进日报或夜间汇总，不刷屏。

## 验证记录

2026-05-14 21:14，测试 `A2A_TEST_20260514_211426`：

- 小枢使用 `sessions_send` 向小霆、小电发送测试消息。
- 小霆回复 `A2A_ACK_XIAOTING A2A_TEST_20260514_211426`。
- 小电回复 `A2A_ACK_XIAODIAN A2A_TEST_20260514_211426`。
- 结论：A2A 已打通。

## 版本历史

| 版本 | 日期 | 变更 |
|---|---|---|
| v0.1 | 2026-05-13 | 初始版本，定义四种消息类型和格式 |
| v0.2 | 2026-05-14 | 记录实际 sessionKey、回执格式、频率限制和 A2A 实测结果 |
| v0.4 | 2026-05-25 | P0-1：A2A 消息增加结构化 `data` JSON 字段，Markdown 给人 + JSON 给 Agent |
