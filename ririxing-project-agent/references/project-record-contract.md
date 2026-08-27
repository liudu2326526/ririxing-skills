# 项目档案与协作契约（项目视角）

## 何时使用

生成孤立的一次性说明时可不读取本文件。需要立项、排程、跨 Agent 协作、更新状态、处理变更或归档时使用。

## 1. 共享项目档案

项目 Agent 以 `project_id` 对应的共享项目档案为状态主干。聊天、提醒和会议纪要只作为来源或通知，不能替代结构化状态。

建议的顶层字段：

|字段|内容|项目 Agent 权限|
|---|---|---|
|`identity`|项目、客户、商机、产品标识|核对并维护项目字段|
|`customer_requirement`|客户原话、归一化需求、版本、来源|只读；变更退回销售/编排层确认|
|`commercial`|报价、条款、数量和商务状态|只读；不得批准|
|`functional_estimates`|各职能 NRE、BOM、工期、测试、试产输入|汇总已审核结果，不代填|
|`plan`|阶段、WBS、依赖、资源、里程碑、基线和预测|主责维护；基线需批准|
|`tasks`|跨 Agent 与人工任务卡|主责协调|
|`risks_issues`|风险、问题、行动和升级|主责维护|
|`decisions_changes`|决策、变更、影响和批准|主责登记；不得越权批准|
|`artifacts`|计划、报告、交付物和版本链接|登记项目产物|
|`audit_events`|谁在何时基于什么输入做了什么|只追加，不覆盖|

关键对象至少带：

```yaml
id: "稳定且唯一的标识"
version: 1
status: "draft | needs_input | pending_owner | pending_approval | approved | stale | superseded"
source_refs: []
owner: "岗位或人员"
updated_at: "带时区时间"
updated_by: "Agent 或人员标识"
```

已批准版本发生变更时保留旧版本，不直接覆盖。新版本在重新批准前不得继承 `approved` 状态。

## 2. 输入可信度

使用 `confirmed`、`reported`、`assumption`、`missing`、`conflict` 标注关键输入。项目计划中的每个重要工期、资源或日期都要能追溯到 Owner、合同/系统、历史基准或明确假设。

历史项目数据可以作为估算参考，但必须说明适用条件和差异，不能冒充当前项目 Owner 的承诺。

## 3. 任务卡与状态

```yaml
task_id: "TASK-..."
project_id: "PRJ-..."
title: "动词开头的可交付任务"
requested_by: "ririxing-project-agent"
owner: "目标 Agent 或人工 Owner"
goal: "为什么需要这个结果"
input_refs:
  - id: "输入对象"
    version: 1
expected_output: "结构、单位、粒度和格式"
acceptance_criteria: []
dependencies: []
planned_start: null
due_at: null
approval_required: false
status: "todo | in_progress | awaiting_input | blocked | done | stale | cancelled"
blocker: null
updated_at: "带时区时间"
```

状态判断：

- `done`：验收条件满足且交付物已登记，不等于仅回复“完成”。
- `awaiting_input`：Owner 已行动但等待外部输入；注明等待对象和期望时间。
- `blocked`：无法继续，注明阻塞原因、影响和升级动作。
- 逾期是日期比较结果，不应作为替代状态；在原状态之外单独标记。

任务输入版本变化时，将旧结果标记为 `stale`，建立变更引用并重评任务。

## 4. 计划与基线对象

每个计划任务建议包含：

```yaml
wbs_id: "WBS-..."
name: "可验收工作包"
owner: "岗位或人员"
duration: "数值加单位或区间"
duration_source: "Owner/历史基准/假设"
predecessors: []
resource_constraints: []
calendar_ref: null
target_dates: {start: null, finish: null}
forecast_dates: {start: null, finish: null}
baseline_dates: {start: null, finish: null}
deliverable: "可验证产物"
acceptance_criteria: []
```

只有获批计划才填写 `baseline_dates`，并同时记录 `baseline_version`、批准人和批准时间。日期计算要说明工作日历、节假日、并行假设、缓冲和关键依赖。

## 5. 风险、问题、决策与变更

四类对象不要混用：

- **风险**：尚未发生的不确定事件；记录概率、影响、触发器、Owner、缓解与应急动作。
- **问题**：已经发生且影响项目的事件；记录影响、Owner、处置和升级时限。
- **决策**：需有权人员选择的事项；记录选项、依据、决策人、截止时间和结果。
- **变更**：对已确认需求或已批准基线的修改；记录来源、影响、方案、审批和传播结果。

最小变更传播规则：

|上游变化|默认检查并标脏的下游对象|
|---|---|
|客户需求|技术方案、BOM/NRE、测试项、NPI、排程、报价|
|技术方案或关键器件|BOM/NRE、结构/软件、测试、采购、NPI、排程|
|数量、供应或资源|成本、采购、试产、排程、报价|
|里程碑或交付日期|依赖任务、资源计划、客户跟进、日报/周报|

原产物保留并改为 `stale`；重算完成和获批后再生成新版本。

## 6. 僵局升级

发生以下任一情况时转人工：

- 多个 Owner 的结论冲突且超出项目 Agent 权限。
- 关键决策超过截止时间并影响关键路径。
- 同一阻塞在约定周期内无 Owner 或无下一动作。
- 变更会影响正式报价、合同、客户承诺或已批准基线。

升级包包含事实时间线、冲突观点及来源、量化影响、可选方案、建议决策人和最晚决策时间。
