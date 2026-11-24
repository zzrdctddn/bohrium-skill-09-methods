---
name: reproducible-methods-writer
description: 把散落的实验记录、配置、日志、代码与数据说明转成"写得全、写得准、做得着、说得住"的方法与实验章节成稿，并保证每个数值与条件都能回溯到来源。当用户要求"写方法章""写实验设置""补参数表""说明数据集划分""写对照实验""补可复现清单""标注限制与待确认项"时使用。输出含六要素抽取表、参数溯源表、可复现清单、对照可比性表、限制—失效场景对齐表与自检结果。
version: 1.0.0
license: MIT
---

# 方法与实验章节写作（Reproducibility-Driven Methods Writing）

> 面向真实科研材料，把"写方法实验"从填空式模板变成**以可复现性为验收标准的工程流程**。核心不是把段落写顺，而是让每个参数、每个条件、每个数值都能回答："这从哪来？别人照着能复做吗？不确定处标出来了吗？"

## 1. 用途与适用边界

**解决什么**：基础 Agent 写方法实验的常见失败——只写主干漏掉关键步骤与重复次数、为段落顺畅补出不存在的数值、参数与配置文件/日志对不上、数据集版本与划分含混导致无法复现、把"与基线比较"写成无依据断言、限制写成"未来可进一步研究"式套话。本 Skill 用"六要素抽取 + 参数溯源 + 可复现清单 + 缺口追问"消除这些失败。

**触发场景**：起草/改写方法章与实验章；把实验记录整理成有序步骤；生成带来源的参数表；说明数据集版本、规模、划分与预处理；写可比对照实验；补复现清单（代码版本、环境、种子、命令、数据获取）；把限制落到具体失效场景。

**不该用**：材料为空或只有结论没有任何实验记录时（应报告缺口写 `partial`/`blocked`，不凭领域常识补参数）；用户只要语言润色而不要结构核对时（转用学术语言润色 Skill）；要求"编一组合理的超参数"时（拒绝编造，转为追问作者确认）。

**输入**：`/input/case.json`（含 `case_id`、任务说明、实验材料/配置/日志/代码/数据说明、受众与文体约束）与 `task.md`。**输出**：`/app/output/result.json`（字段见第 4 节）+ 可编辑的 `methods_experiments.md`。

## 2. 核心方法（RDMW）

本 Skill 的方法名为 **RDMW = Reproducibility-Driven Methods Writing（可复现性驱动的方法实验写作）**，四个不可跳过的机制，对应题面四类关键问题：

1. **写得全 — 六要素抽取（Six-Element Extraction）**：任何方法实验章必须覆盖六类要素——① 方法步骤（每步输入/操作/输出/判断条件，不把多步压成一句）② 实验设置（训练/验证/测试分开，硬件、软件版本、随机种子、重复次数）③ 参数（名称/取值/单位/作用对象/来源）④ 数据集与划分（名称、版本、规模、预处理、划分比例与随机方式、子集用途）⑤ 对照实验（基线对象及版本、是否同预算、评价指标与统计方式）⑥ 复现与限制。缺一类即标 `missing_element`，不静默跳过。
2. **写得准 — 参数溯源表（Parameter Provenance Table）**：每个参数记录 `name / value / unit / scope / source`（配置文件、日志、代码或作者确认）/ `source_location`；正文数值必须与溯源表逐项对齐；源里没有的数值一律不写，禁止为段落顺畅补数。
3. **做得着 — 可复现清单（Reproducibility Manifest）**：给出复现所需的全部条件——代码版本/commit、环境与依赖及版本、随机种子、完整运行命令、数据获取方式、硬件；清单条目必须来自材料，缺项写入 `human_review_items` 而非留空或臆造。
4. **说得住 — 缺口追问与限制落地（Gap Interrogation & Limitation Anchoring）**：材料缺口与不确定处显式标 `uncertainty` 并转成具体追问清单交作者确认；每条限制必须绑定一个**具体失效场景与适用范围**，禁止"仍有提升空间"式套话。

另贯穿铁律：**抽取与核对分离**（先抽取成稿，再以审阅者身份独立核对，两遍不可合并）；**不编造事实与 benchmark**（没有的参数、数值、版本、种子不补；无法判断写 `uncertainty`，运行不了写 `blocked`）。

## 3. 工作流（SOP）

严格按顺序执行，每步都有检查点；检查点不过则回退该步或转入异常处理。

**Step 0 · 输入解析与边界确认**
- 读 `case.json` 与 `task.md`，提取：目标体裁/受众/语言、需要补写的要素、材料清单、明确禁止事项（如禁止联网、禁止补数）。
- 检查点：明确六要素中哪些有材料、哪些缺；缺口登记 `uncertainties`，不凭领域标签补造。

**Step 1 · 材料盘点与快照（Inventory）**
- 清点并保存材料快照：实验记录、配置文件、日志、代码、数据说明、表格；为每条材料标注路径/版本/页码或行号，建立可回溯索引。
- 检查点：材料逐项可定位；无来源的数值/参数在盘点阶段就标记，不进入成稿。

**Step 2 · 六要素抽取**
- 按 RDMW 六要素抽取：步骤序列（多步操作拆分，记录判断条件）、训练/验证/测试设置、参数候选、数据集与划分、对照对象、复现条件与限制候选。
- 检查点：六要素齐全或明确标 `missing_element`；步骤不把多步压成一句；训练/评测条件不串台。

**Step 3 · 参数溯源表构建**
- 把每个参数写入溯源表（名称/取值/单位/作用对象/来源/来源位置/状态）；正文出现的每个数值逐项对齐溯源表；来源冲突（配置 vs 日志 vs 代码）并列展示并标 `conflict`，交人工裁定。
- 检查点：正文数值 100% 在溯源表有据；无凭空补数；冲突项已标注而非择一隐藏。

**Step 4 · 条件一致性校验**
- 对照配置/日志/代码，校验数据集版本与划分、训练与评测条件、对照预算是否一致；检查"与基线比较"是否同预算、同指标、同统计方式。
- 检查点：条件串台已纠正并写 `change_log`；不可比对照标 `not_comparable` 并说明，不写成断言。

**Step 5 · 可复现清单编制**
- 汇总复现所需条件：代码版本/commit、依赖及版本、种子、完整命令、数据获取、硬件、运行环境；材料没有的项标 `missing` 并进 `human_review_items`。
- 检查点：清单可支撑他人复做；无臆造的版本号、种子或命令。

**Step 6 · 缺口追问与限制落地**
- 把所有不确定项转成具体追问清单；把限制改写为"在 X 条件/场景失效 + 适用范围"的句式。
- 检查点：每条限制有具体失效场景；无套话；追问项全部进 `human_review_items`。

**Step 7 · 独立核对与输出**
- 以审阅者身份逐项走 `references/checklist.md`；生成 `methods_experiments.md` 与 `result.json`；所有改动写 `change_log`，未通过项写清原因与 `status`。
- 检查点：自检表全过或每项未过有 `change_log`；`status` 与完成情况一致；示例文本明确为示意。

## 4. 输出契约（result.json）

```json
{
  "case_id": "<原样带回>",
  "status": "success | partial | blocked",
  "note": "示意文本请标注，非真实案例",
  "artifact": {
    "methods_text": "<方法章可编辑正文>",
    "experiments_text": "<实验章可编辑正文>",
    "language": "zh|en",
    "audience": "<目标受众>"
  },
  "step_sequence": [
    {"step_id":"S1","input":"...","operation":"...","output":"...","decision_condition":"...","source_location":"日志L12-20"}
  ],
  "parameter_provenance_table": [
    {"name":"learning_rate","value":"1e-4","unit":"-","scope":"训练","source":"config.yaml","source_location":"L7","status":"sourced|conflict|unsourced"}
  ],
  "dataset_split": {
    "name":"...","version":"...","scale":"...","preprocessing":"...","split_ratio":"...","random_method":"...","subset_usage":"...","source_location":"..."
  },
  "experiment_setup": {
    "train":"...","validation":"...","test":"...","hardware":"...","software":"...","seeds":["..."],"repeats":"...","source_location":"..."
  },
  "baseline_comparison": [
    {"baseline":"...","version":"...","budget_matched":true,"metrics":["..."],"statistical_method":"...","comparable":true,"basis":"..."}
  ],
  "reproducibility_manifest": {
    "code_version":"...","environment":"...","commands":["..."],"data_access":"...","seeds":["..."],"hardware":"..."
  },
  "limitations": [
    {"limitation":"...","failure_scenario":"...","scope":"...","source_location":"..."}
  ],
  "fact_source_map": [{"item":"<数值/参数>", "value":"...", "source_location":"..."}],
  "uncertainties": ["..."],
  "human_review_items": ["..."],
  "change_log": [{"step":4,"action":"纠正训练/评测条件串台","reason":"配置与日志不一致"}],
  "self_check": {
    "six_elements_complete": true,
    "numbers_sourced": true,
    "conditions_consistent": true,
    "reproducible": true,
    "limitations_anchored": true,
    "no_fabrication": true
  }
}
```

字段原则：`status` 诚实反映完成情况；`partial`/`blocked` 必须写清 `uncertainties`；`unsourced`/`missing` 不等于"没有"，需进人工确认；不得编造参数、版本号、种子、命令或 benchmark 分数；示例一律标注为示意。

## 5. 质量自检清单

执行前完整读取 `references/checklist.md`，逐项打钩。关键硬项：

- [ ] 六要素（步骤/设置/参数/数据/对照/复现限制）齐全，缺项有 `missing_element`
- [ ] 步骤拆到单步，含输入/操作/输出/判断条件
- [ ] 每个数值/参数可回溯到配置/日志/代码/作者确认
- [ ] 数据集版本、划分比例、随机方式、子集用途明确
- [ ] 训练/验证/测试条件不串台，对照同预算可比
- [ ] 可复现清单含代码版本、环境、种子、命令、数据获取
- [ ] 每条限制落到具体失效场景；缺口转为追问清单
- [ ] 无材料外新增数值/参数/版本；`status` 诚实；改动有 `change_log`

## 6. 异常处理与边界

| 情况 | 处理 |
|---|---|
| 材料只有结论无记录 | `status=blocked`，列缺口，不凭领域常识补参数 |
| 参数来源冲突（配置/日志/代码不一致） | 并列展示标 `conflict`，进 `human_review_items`，不擅自择一 |
| 参数无来源 | 标 `unsourced`，成稿保留占位并追问，不补数 |
| 数据集版本/划分不明 | 标 `uncertainty`，写清影响，交作者确认 |
| 对照不同预算/不可比 | 标 `not_comparable`，降级为描述性对照，不写成有效性断言 |
| 复现命令/环境缺失 | 清单标 `missing`，进人工确认，不臆造版本号/种子 |
| 权限不足读不到配置/代码 | 说明受阻材料，`status=partial`，给出"补充 X 可继续" |
| 要求补造"合理的数值" | 拒绝，转成追问，说明会损害可复现性与正确性 |

## 7. 最小调用示例（示意，非真实案例）

**输入**（`case.json` 片段）：
```json
{"case_id":"demo-009","task":"根据实验记录与配置写方法章和实验章","params":{"language":"zh","audience":"CS 顶会读者"},"materials":["notes.md","config.yaml","train.log","code/","data_split.json"]}
```
**调用**：加载本 Skill 后，用户说"用 reproducible-methods-writer 补这篇的方法和实验部分"。

**示意输出**（节选）：
```json
{"case_id":"demo-009","status":"success","artifact":{"language":"zh"},"parameter_provenance_table":[{"name":"learning_rate","value":"1e-4","source":"config.yaml","source_location":"L7","status":"sourced"}],"reproducibility_manifest":{"code_version":"<待作者确认>","commands":["python train.py --config config.yaml"]},"self_check":{"six_elements_complete":true,"numbers_sourced":true}}
```
> 上述为演示接口用的虚构示例，不代表任何真实案例结果；`<待作者确认>` 体现不编造。

## 8. 资源清单

- `references/method.md` —— RDMW 四机制细则与反例库
- `references/checklist.md` —— 覆盖四类关键问题的逐项自检清单
- `references/schema.json` —— 输出字段字典（JSON Schema）
- `references/element_rules.md` —— 六要素定义、参数来源优先级、对照可比性规则
- `references/reproducibility_rules.md` —— 溯源字段字典、缺口追问话术、限制落地模式
- `assets/methods_experiments_template.md` —— 可编辑方法实验章模板
- `examples/` —— 最小调用示例（标注为示例，非真实案例）
