# 出题、修订与试卷处理

[项目首页](../../README.md) · [安装指南](../../docs/SETUP.md) · [已知问题](../../docs/KNOWN_ISSUES.md)

这个目录是一组实验工具，涵盖知识点出题、题目质量检查、Excel 修订、教师反馈比较，以及桌面窗口需要的数据库抽题。文件名中的 `v1` 到 `v4_1` 是并存的实验实现，不能简单把最高编号当作经过验证的统一入口。

## 先按任务选择入口

| 要做的事 | 阅读或使用的入口 | 前置条件 |
| --- | --- | --- |
| 从知识点 Word 生成题目 | `jap_question_generator_v2_1_qwen_based.py` 与 `config.py` | Python 依赖、API 凭据、可解析的知识点文档；先做小规模验证 |
| 用一个模型检查和修改整份 Excel | `jap_question_generator_v3_1_qwen_based.py` | `docs/Generated_paper/shatin/` 中的题目、可用模型 |
| 把整份题目拆成小批次 | `split_excel_paper.py` | 原题 Excel，先手工创建输出目录并备份旧分片 |
| 用多模型投票判断哪些题需要修改 | `jap_question_generator_v3_4_qwen_based.py` 的模式 2 | 已切分的题目、对应实验组模型的 API 权限 |
| 比较程序修改与教师标注 | `extract_bad_questions.py` 和 `jap_question_generator_v3_2_qwen_based.py` | 对齐的原题、修订题和教师反馈 |
| 理解桌面界面的分析与抽题 | `test_paper_generation.py` | MySQL 中已有学生、题目和作答记录 |
| 理解 Word 批改和知识点读取 | `jap_paper_revise.py` | 符合固定格式的试卷、标准答案和学生答案 |

模型调用可能反复执行修订、检查和重试。第一次运行应限定一个知识点或一个小文件，并把输出放在新目录。此处列的是源码入口，不表示已验证全部脚本可以无修改运行。

## 每个文件的用途

### 共享设置和文件处理

| 文件 | 用途与输入输出 | 使用说明 |
| --- | --- | --- |
| [config.py](config.py) | 为 v2_1 提供知识点 Word 路径、输出目录、知识点范围、题型规则、难度分配和提示词模板 | 默认 1 到 8 个知识点，每知识点 15 题、5 类题型；其他版本不会自动采用这里的全部设置 |
| [jap_excel_processor.py](jap_excel_processor.py) | 解析 LLM 文本，写出带教师评价栏的 Excel，将 Word 文件批量转换为 Excel，也把 Excel 读回题目列表 | 共用的格式桥梁；列顺序和题号样式是数据接口的一部分 |
| [jap_paper_revise.py](jap_paper_revise.py) | 清理 Word 中的示例和分隔符、拆题、读取姓名与答案、比较对错、提取知识点和 N4/N5 标签 | 从标准答案判分，不调用 LLM；文件末尾有直接执行的示例，导入也可能处理文件 |
| [test_paper_generation.py](test_paper_generation.py) | 查询学生与试卷、显示错题知识点、按比例从题库抽题、排版 Word | GUI 使用的功能模块；不是测试套件，也不是 LLM 生成器 |
| [__init__.py](__init__.py) | Python 包标记 | 空文件，不需运行 |
| `README.md` | 当前目录的入口、逐文件解释和数据约定 | 阅读文档 |

### 从知识点生成新题的实验版本

| 文件 | 代码中的主要特点 | 输入与产物 |
| --- | --- | --- |
| [jap_question_generator_v1_qwen_based.py](jap_question_generator_v1_qwen_based.py) | 早期知识点出题和整套检查修订流程，规则写在文件内部 | 知识点 Word → 初稿与修订 Word、Excel |
| [jap_question_generator_v2_qwen_based.py](jap_question_generator_v2_qwen_based.py) | 两种语法填空模式，包含难度分配、题干与多解检查、重复检查 | 知识点 Word → 生成与修订材料 |
| [jap_question_generator_v2_1_qwen_based.py](jap_question_generator_v2_1_qwen_based.py) | 将主要生成规则移到 `config.py`；现有配置包含复合动词题型 | 配置指定的知识点 Word → 配置指定的输出目录 |
| [jap_question_generator_v2_2_qwen_based.py](jap_question_generator_v2_2_qwen_based.py) | 两种题型的提示词变体，细化唯一答案、干扰项、选项位置和示例要求 | 知识点 Word → 生成与修订材料 |
| [jap_question_generator_v3_qwen_based.py](jap_question_generator_v3_qwen_based.py) | 提取问题题目并合回试卷的修订流程 | 知识点 Word → 修订试卷；默认路径多带了一层 `JAP_LLM_Platform/`，且有额外 `langchain.prompts` 导入 |
| [jap_question_generator_v4_qwen_based.py](jap_question_generator_v4_qwen_based.py) | 通过 JSON 风格的检查结果提取错误和评论，传递知识点辅助修订 | 知识点 Word → 带检查修订过程的结果文件 |
| [jap_question_generator_v4_1_qwen_based.py](jap_question_generator_v4_1_qwen_based.py) | 在生成流程中调用 `jap_voting_machine.py`，汇总多模型意见后修订 | 知识点 Word → 经过投票环节的修订材料 |

这些脚本并非全部使用相同题型。注释可能沿用旧版描述，具体以函数和当前配置为准。提示词要求“唯一答案”“N4/N5”不等于生成内容已满足要求。

### 对已有 Excel 修订和比较

| 文件 | 主要工作 | 默认输入与输出 |
| --- | --- | --- |
| [jap_question_generator_v3_1_qwen_based.py](jap_question_generator_v3_1_qwen_based.py) | 菜单选择单个模型，对整份 Excel 检查、修订，主入口最多 5 轮 | `docs/Generated_paper/shatin/` → `docs/revised_shatin/<模型名>/` |
| [jap_question_generator_v3_2_qwen_based.py](jap_question_generator_v3_2_qwen_based.py) | 对比原题和修订题的题干、选项是否变化，与教师需要修改的题号比较 | 原题、模型结果和教师索引 → `docs/revised_shatin/model_comparison_summary/`；比较本身不调用 LLM |
| [jap_question_generator_v3_3_qwen_based.py](jap_question_generator_v3_3_qwen_based.py) | 将单模型修订用于按题量切分的 Excel，并返回所选模型名供后续合并使用 | `docs/Generated_paper/question_num_significance_test/paper/` → 模型输出目录 |
| [jap_question_generator_v3_4_qwen_based.py](jap_question_generator_v3_4_qwen_based.py) | 支持单模型、一个投票组、全部四个投票组；达到加权阈值的题目再交给修订模型 | 模式 1 读取 `revised_grammar_questions/`；模式 2/3 读取切分目录；结果在模型或 `voting_group_*` 目录 |
| [jap_voting_machine.py](jap_voting_machine.py) | v4_1 使用的多模型检查助手，给题目文本投票并汇总共识错误 | 输入题目文本及模型配置，返回投票和错误信息；不是独立界面 |
| [jap_voting_machine_v3_4.py](jap_voting_machine_v3_4.py) | v3_4 使用的逐题加权投票，配置四组模型和阈值，提供单题修订函数 | 输入题目文本和实验组，返回逐题票数、错误类别及阈值判断 |

两套 voting 模块有不同汇总规则，不能直接互换。v3_4 的四组阈值为 0.51、0.61、0.57、0.48；组名如 High Quality 和 Fast 是配置标签，不是已经证实的质量或速度排名。修订模型在该流程中选用配置权重最高的模型。

### 批处理工具和流程入口

| 文件 | 工作顺序或用途 | 运行前需要知道 |
| --- | --- | --- |
| [extract_bad_questions.py](extract_bad_questions.py) | 将教师反馈中的 `Drop` 和 `Minor changes` 转成问题题号及 0/1 序列 | 读取 `docs/paper_with_feedback/4.11shatin/`，输出到 `question_index/`；索引公式假定每题型 10 题 |
| [comparison_pipeline.py](comparison_pipeline.py) | v3_1 单模型修订 → 提取教师问题标注 → v3_2 比较 | 修订和比较各有模型选择，须选择同一个模型；会调用 API |
| [split_excel_paper.py](split_excel_paper.py) | 询问每份题量，将 `shatin/` 中 Excel 拆成多个小文件 | 输出到 `question_num_significance_test/paper/`；旧目录内容会被清理，首次运行不会自动建这个目录 |
| [combine_excel_paper.py](combine_excel_paper.py) | 按文件名中的分片编号合并修订结果，跳过中间轮次文件 | 会删除所选结果目录里的原有 `.xlsx` 再放入合并文件；先使用实验副本或备份 |
| [split_paper_test_pipeline.py](split_paper_test_pipeline.py) | 切题 → v3_4 修订 → 合并 → v3_2 比较 | 当前适合在准备好人工索引后选择模式 2 的单个投票组；模式 1 输入路径不同，模式 3 返回值不能直接代表一个可合并目录 |

## 运行模型流程

API 和 Python 环境配置见[安装指南](../../docs/SETUP.md)。以下命令从仓库根目录运行，先核对输入输出路径并保护原始材料。

```sh
python "Source Code/Paper_Generator/jap_question_generator_v2_1_qwen_based.py"
python "Source Code/Paper_Generator/jap_question_generator_v3_1_qwen_based.py"
```

这两个命令是不同任务的入口，不要求连续执行。前者从知识点出题，后者读取 `shatin/` 已有 Excel，前者的输出不会自动成为后者输入。

需要进行分片投票实验时，先在新的实验副本中准备目录，确认手工反馈索引存在。

```powershell
New-Item -ItemType Directory -Force "docs/Generated_paper/question_num_significance_test/paper"
python "Source Code/Paper_Generator/extract_bad_questions.py"
python "Source Code/Paper_Generator/split_paper_test_pipeline.py"
```

输入正整数指定每个分片的题量，在修订菜单选择 `2`，再选择一个实验组。该流程会调用多个模型、修改输出文件并清理部分中间文件，不适合用原始材料目录做首次试验。若只想理解流程，直接阅读已有 Excel 与日志即可。

## 题目数据约定

### Word 知识点与学生答卷

知识点生成器用规则匹配文档中的编号条目；v2_1 的规则来自 `config.test_grammar_format`，其他版本在自身文件内定义。替换输入文档前应先核对条目是否能被选中。

学生答案解析器只读取以 `問題　` 开头的段落，并按全角 `：` 提取答案。姓名使用 `Name:` 段落，学号提取规则依赖 Windows 反斜杠后的 10 位数字。试卷中的知识点依赖 `-Knowledge Points:` 标记。任意 Word 格式、截图、PDF 或手写答卷不会自动符合这些约定。

### Excel 题目表

`store_questions_to_excel()` 写出以下六列，前四列用于题目内容，后两列供人工评价。

| 列名 | 内容 |
| --- | --- |
| `Question Index` | 题号，解析过程中会结合题型构造 `Qx: もんだいy` |
| `Content` | 题干 |
| `Options` | 合在一个单元格中的四个选项 |
| `Answer` | 标准答案 |
| `Suggestions` | 教师对是否采用或修改的标记 |
| `Modifications (if any)` | 具体修改建议 |

读取逻辑依赖列位置及选项编号，随意调换列或改变题号会影响结果。修订输出常只保留前四列，不能假定教师评价会自动随结果保留。

**审核标签尚未统一。** 写表下拉选项是 `High_Q, Low_Q, Drop, Minor changes`；教师问题提取脚本只保留 `Drop, Minor changes, OK`，再标记前两者；数据库导入却接受 `High_Q, Minor changes, Low_Q`。流程接通前应统一这些含义和允许入库的状态。

## 比较结果的准确含义

v3_2 主要记录题干或四个选项是否改变，仅答案列改变不会被当前条件识别。教师标记需要修改的题与模型修改的题分别形成 0/1 序列，再统计两者相同的位置比例。

因此 `Correct Rate` 表示修改决策与教师标注的一致率，包含“双方都认为不用修改”的题；它不评价修订后的语言质量、标准答案或学生学习效果。题量与题号须一致，固定每题型 10 题的索引假设也必须成立，才能解释该统计。

## 接手时应优先检查

- API 失败与“没有检测到错误”必须区分。部分现有检查函数捕获异常后返回空列表，可能被误读为通过。
- 保存了最终文件可能只是达到最大轮数或没有产生变化，不能仅凭 `revised` 文件名认定审核合格。
- 重跑可能覆盖日志、重复追加比较记录或清理中间文件，应为每轮实验指定独立目录。
- 当前结果分散在文件系统中，还没有统一接入教师审核和数据库发布流程。
- LLM、harness 的格式检查和人工审核分别提供不同证据，发布题目前应保留来源与最终人工判断。

更详细的问题和后续验收见[已知问题](../../docs/KNOWN_ISSUES.md)与[路线图](../../docs/ROADMAP.md)。
