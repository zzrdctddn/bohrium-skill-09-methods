# 方法与实验章节写作 Skill（reproducible-methods-writer）

一个把散落实验材料转成"可核验、可复做"方法实验章的 Agent Skill。核心：六要素抽取 + 参数溯源表 + 可复现清单 + 缺口追问与限制落地 + 抽取核对分离。

## 目录结构
- `SKILL.md` —— Skill 本体（含 YAML frontmatter、RDMW 工作流、输出契约）
- `references/` —— 方法细则、自检清单、字段字典、要素规则、复现规则
  - `method.md` —— RDMW 四机制细则与反例库
  - `checklist.md` —— 覆盖"写得全/写得准/做得着/说得住"的分组自检清单
  - `schema.json` —— result.json 的 JSON Schema（draft-07）
  - `element_rules.md` —— 六要素定义、参数来源优先级、数据集与对照可比性规则
  - `reproducibility_rules.md` —— 溯源字段字典、缺口追问话术、限制落地模式
- `assets/methods_experiments_template.md` —— 可编辑方法实验章模板
- `examples/` —— 最小调用示例（已标注为示例，非真实案例）
- `GITHUB_URL.txt` —— 提交前填 GitHub 公开链接

## 使用方式
1. 将本目录作为 Skill 安装到 Agent（平台约定路径 `/skills/weekly-skill`）。
2. 输入 `/input/case.json`（实验材料、配置、日志、代码、数据说明、任务说明）与 `task.md`，自然语言下指令如"补方法章和实验章"。
3. 按 SKILL.md 的 SOP（Step 0~7）执行，输出到 `/app/output/result.json` 与 `methods_experiments.md`。

## 依赖
仅需文本处理能力；无外部运行时依赖。六要素定义、溯源规则、追问话术内置于 SKILL.md 与 `references/`。读取配置/日志/代码为材料解析，不需要执行实验；对无法读取的材料诚实标 `blocked`。

## 适用与限制
- 适用：CS/AI/数据等学科的方法与实验章起草、参数表生成、数据集划分说明、对照实验描述、复现清单与限制条件撰写。
- 限制：材料缺失、配置冲突、参数无来源、版本不明时会标 `partial`/`blocked` 并追问，不编造参数、版本号、种子、命令或 benchmark 分数；不替代作者提供只有本人知道的实验细节。

## 与评分维度的对应
- 实际效果(40)：SOP Step 2~6 覆盖步骤/设置/参数/数据/对照/复现与限制六类要素，溯源表与清单提供逐条回溯（见第 3、4 节）。
- 使用增益(25)：结构化抽取与校验减少翻配置、追问和返工；增益对照由平台白板给出，本作品不自带分数（见第 2、7 节）。
- 创新增益(15)：参数溯源、条件一致性校验、缺口追问与限制落地为新增能力，优于填空模板（见第 2 节与 `references/method.md`）。
- 可靠体验(10)：异常处理覆盖配置冲突、参数无源、版本不明、权限不足等情况（见第 6 节）。
- 生态贡献(10)：要素规则、复现规则、模板、示例与反例库可供复用（见第 8 节与 `references/`）。

## Runtime contract
平台加载 Skill 到 `/skills/weekly-skill`，输入在 `/input/case.json` 与 `task.md`，输出到 `/app/output/result.json`（含全部输出契约字段与 `case_id`）。资源最多 50 个 UTF-8 文本文件、单文件 ≤500KB、总计 ≤2MB、无软链。执行预算 64 次模型请求 / 900 秒 / 每次 16384 token。
