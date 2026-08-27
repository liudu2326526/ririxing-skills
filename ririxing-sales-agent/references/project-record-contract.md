# 项目档案与协作契约（销售视角）

## 何时使用

整理孤立的一次性草稿时可不读取本文件。需要读写共享项目档案、向其他 Agent 派发任务、处理客户变更或完成项目交接时使用。

## 1. 项目档案是唯一协作主干

销售 Agent 与项目 Agent 必须针对同一个 `project_id` 读写同一份项目档案。聊天消息用于通知，不能替代档案中的结构化状态。

建议的顶层字段：

|字段|内容|销售 Agent 权限|
|---|---|---|
|`identity`|项目、客户、商机、产品标识|维护销售来源字段|
|`customer_requirement`|客户原话、归一化需求、版本、来源|主责维护|
|`commercial`|数量、目标成本、条款、报价版本与状态|维护草稿；正式值需 Owner 审核|
|`functional_estimates`|各职能 NRE、BOM、周期、测试、试产输入|只引用，不代填|
|`plan`|项目阶段、WBS、里程碑、基线与预测|只读或发起请求|
|`tasks`|跨 Agent 与人工任务卡|可创建和跟踪销售相关卡片|
|`risks_issues`|风险、问题与行动|可登记客户/商务风险|
|`decisions_changes`|决策、变更、影响与批准|创建客户需求变更；不越权批准|
|`artifacts`|文档、报价、报告和版本链接|登记销售产物|
|`audit_events`|谁在何时基于什么输入做了什么|只追加，不覆盖|

每个关键对象至少带：

```yaml
id: "稳定且唯一的标识"
version: 1
status: "draft | needs_input | pending_owner | pending_approval | approved | stale | superseded"
source_refs: []
owner: "岗位或人员"
updated_at: "带时区时间"
updated_by: "Agent 或人员标识"
```

`approved` 只表示指定 Owner 已批准当前版本，不代表其他维度也已批准。新版本不得静默覆盖旧版本；旧版本改为 `superseded` 并保留引用。

## 2. 事实与证据

每项关键输入标注以下类别之一：

- `confirmed`：有客户、合同、系统或有权 Owner 的明确证据。
- `reported`：来自消息或会议纪要，尚未完成正式确认。
- `assumption`：为继续分析而采用的假设。
- `missing`：尚无数据。
- `conflict`：存在至少两个互相冲突的来源。

来源引用应包含 `source_id/title`、时间、页码/段落/消息定位（可得时）和提取人。不要把“历史项目通常如此”标成当前项目事实。

## 3. 任务卡

向项目或职能 Agent 派发的任务使用以下最小结构：

```yaml
task_id: "TASK-..."
project_id: "PRJ-..."
title: "动词开头的可交付任务"
requested_by: "ririxing-sales-agent"
owner: "目标 Agent 或人工 Owner"
goal: "为什么需要这个结果"
input_refs:
  - id: "输入对象"
    version: 1
expected_output: "结构、单位、粒度和格式"
acceptance_criteria: []
dependencies: []
due_at: "带时区时间或 null"
approval_required: false
status: "todo | in_progress | awaiting_input | blocked | done | stale | cancelled"
blocker: null
created_at: "带时区时间"
```

当输入对象版本变化时，不得继续把旧任务结果当成当前结论。将任务或结果标记为 `stale`，引用变更记录，再决定重开还是新建任务。

## 4. 变更传播

至少执行以下影响检查：

|上游变化|默认检查并标脏的下游对象|
|---|---|
|客户功能、性能或接口需求|技术方案、NRE、BOM、测试项、排程、报价|
|外观、尺寸、材料或防护需求|结构/模具评估、BOM、测试、NPI、排程、报价|
|目标数量、交付地区或商务条款|询价、BOM 成本、毛利、报价、交付计划|
|目标日期或交付批次|项目排程、资源计划、NPI、采购、客户跟进计划|

“标脏”不是删除原产物，而是将当前版本状态改为 `stale`，记录原因和影响，再派发重算任务。

## 5. 销售交接完成条件

交接包至少包括：

- 项目/客户/产品身份与当前需求版本。
- 客户目标、关键场景、范围边界和优先级。
- 已确认与待确认的商务条件。
- 当前报价状态及其专业输入版本。
- 已作出的客户承诺与明确未承诺事项。
- 未决问题、风险、任务卡、Owner 和截止时间。
- 原始资料、产物和审计记录的可访问引用。

缺失项可带状态交接，但必须显式说明是否阻断立项。
