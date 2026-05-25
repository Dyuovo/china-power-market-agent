# OUTPUT_SCHEMAS — 输出格式校验模板

版本：v1.0  
创建：2026-05-25  
状态：活跃  

## 说明

此目录下的 JSON Schema 定义关键输出类型的必填字段和结构。
小枢收到 TASK_DONE 后可自动校验字段完整性。

各 Schema 分两段：
- `required`：必须出现的关键字段
- `recommended`：建议包含但非强制

---

## 1. 电力市场日报 (xiaoting.daily.report)

```yaml
output_type: daily_report
agent: xiaoting
required:
  - date: YYYY-MM-DD
  - province: 省份名
  - market_summary:
      - da_avg_price: 日前均价（元/MWh）
      - rt_avg_price: 实时均价（可选）
      - peak_valley_spread: 峰谷价差
  - key_signals: [至少1条市场信号]
  - risk_flags: [至少1条风险提示]
  - data_source: 数据来源
recommended:
  - weather_summary: 天气概况
  - load_peak: 负荷峰值
  - new_energy_output: 新能源出力
  - near_term_outlook: 近3日展望
  - feishu_url: 飞书文档链接（如已推送）
```

## 2. 电力市场周报 (xiaoting.weekly.report)

```yaml
output_type: weekly_report
agent: xiaoting
required:
  - week: WNN
  - date_range: [start, end]
  - province: 省份名
  - weekly_summary:
      - avg_da_price: 周均日前价
      - price_trend: 涨/跌/平
  - key_event: 本周最重要市场事件
  - next_week_outlook: 下周展望
recommended:
  - price_chart_summary: 价格走势描述
  - policy_update: 政策动态
  - risk_matrix: 风险矩阵
  - feishu_url: 飞书文档链接
```

## 3. 学习推送 (xiaodian.learning.push)

```yaml
output_type: learning_push
agent: xiaodian
required:
  - date: YYYY-MM-DD
  - version: V1 | V2 | V3
  - daily_question:
      - question: 题目
      - reference_answer: 参考答案（V1 可不填）
  - exercises: [至少1道练习题]
  - knowledge_tags: [至少1个知识标签]
recommended:
  - owner_answer: 主人答案（V2/V3 必填）
  - scores: [逐题得分] （V3 必填）
  - strengths: [亮点] （V3 必填）
  - gaps: [待加强] （V3 必填）
  - feishu_url: 飞书文档链接
  - db_synced: 是否已写入 learning.sqlite
```

## 4. 政策 Pulse (xiaoting.pulse.policy)

```yaml
output_type: policy_pulse
agent: xiaoting
required:
  - date: YYYY-MM-DD
  - policy_title: 政策文件名
  - issuing_body: 发文单位
  - publish_date: 发布日期
  - key_points: [至少3条要点]
  - market_impact: 对市场的可能影响
recommended:
  - full_text_link: 原文链接
  - related_policies: [关联政策]
  - stakeholder_analysis: 利益方分析
```


```yaml
output_type: health_report
required:
  - date: YYYY-MM-DD
  - sections:
      - body: 体重/体脂/肌肉（如有）
      - sleep: 睡眠时长/质量（如有）
      - training: 训练内容（如有）
      - diet: 饮食概况（如有）
  - trend_note: 趋势判断（升/降/稳）
recommended:
  - risk_flags: [健康风险提示]
  - next_day_plan: 次日计划
```

---

## 校验脚本

```bash
# 校验单个输出
python3 00_GOVERNANCE/OUTPUT_SCHEMAS/validate.py --schema daily_report --file path/to/report.md

# 自动校验 TASK_DONE 产出
python3 00_GOVERNANCE/OUTPUT_SCHEMAS/validate.py --task-id TASK-YYYYMMDD-NNN
```

---

## 版本历史

| 版本 | 日期 | 变更 |
|---|---|---|
| v1.0 | 2026-05-25 | 初始版本，定义 5 种输出类型 schema |
