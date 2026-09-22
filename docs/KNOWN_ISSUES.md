# 已知问题与交接边界

[项目首页](../README.md) · [运行准备](SETUP.md) · [后续计划](ROADMAP.md)

以下结论来自代码快照 `f4bd5b4` 的静态阅读，不是对历史数据库或线上服务的检查报告。本次改动只补充文档，未修复这些代码、执行数据库脚本或验证模型 API。问题按对交接的影响列出，便于接手者先完成小规模可靠流程。

## 数据库与记录

| 问题 | 源码位置 | 影响与建议 |
| --- | --- | --- |
| 导入模块即可删除三张表 | [db_question_students_results.py](../Source%20Code/SQL_Database/db_question_students_results.py) 顶层连接和 `try` 块；[insert_db.py](../Source%20Code/SQL_Database/insert_db.py) 的导入 | 拆分连接、SQL 定义与显式初始化；不能在已有教学库上直接运行导入 |
| 初始化结束即关闭共享连接 | 建表模块 `finally`，导入脚本使用其 `db` | 导入脚本虽然尝试重连，之前的删表已经发生；应独立管理连接生命周期 |
| 对错含义互相冲突 | [jap_paper_revise.py](../Source%20Code/Paper_Generator/jap_paper_revise.py) 的 `return_revised_result`、GUI 分析、`select_mistake_query` | 前者 1 为错、0 为对，GUI 沿用；查询错题 SQL 却筛 0。先确定约定并核对历史数据 |
| 新导入会删除该学生旧结果 | `insert_db.py` 的 `process_student_results` | `DELETE FROM exam_results WHERE student_id = ...` 没有作业限定，不能当成长时学习记录 |
| 学生更新参数数量不符 | `insert_or_update_student` | SQL 有两个占位符，但传入三个参数，已有学生更新会出错 |
| “插入或更新”实际是普通插入 | `insert_or_update_question` 与 `insert_questions_query` | 唯一题号重复时可能失败；需要真正的重复处理策略 |
| 数据库参数分散且名称大小写混用 | 界面、建表、Excel 导入、查询及抽题文件 | 统一 host、port、database 和账号配置，不能假定只修改一个文件即可 |
| 样例邮箱由学号拼接 | `process_student_results` | 未验证实际邮箱，不适合当作真实联系信息 |
| Word 报告同名覆盖 | `insert_db.py` 的 `main` | 多学生循环共用 `docs/new_db_test.docx`，不能据此认定保存了每次报告 |

## 题库与格式

| 问题 | 源码位置 | 影响与建议 |
| --- | --- | --- |
| Excel 导入仍使用旧词汇题格式 | [Excel2Db.py](../Source%20Code/SQL_Database/Excel2Db.py) 顶层文件循环 | 只有包含 `Vocabulary` 的文件名会初始化部分变量，现有语法题不一定兼容；级别也从文件名前两字符取得 |
| 入库筛选并非仅高质量题 | 同文件第五列判断 | `Low_Q` 与 `Minor changes` 也进入导入条件，应增加明确的审核发布规则 |
| 文件被移动不等于全部成功入库 | 同文件 `shutil.move` | 增加成功数、失败数、事务与导入日志，再决定归档 |
| 人工评价标签不统一 | [jap_excel_processor.py](../Source%20Code/Paper_Generator/jap_excel_processor.py)、[extract_bad_questions.py](../Source%20Code/Paper_Generator/extract_bad_questions.py)、Excel 导入 | 写表提供 High_Q/Low_Q，提取脚本却保留 OK 等标签，可能漏读反馈 |
| 学生 Word 格式限制较强 | `jap_paper_revise.py` 的答案、姓名和学号提取 | 依赖特定字符及 Windows 路径样式，没有通用文档上传解析 |
| 导入解析模块也会生成处理文件 | `jap_paper_revise.py` 最后几行 | 示例调用不在主程序保护内；应将演示代码与可复用函数分离 |
| 分片首次输出目录不存在 | [split_excel_paper.py](../Source%20Code/Paper_Generator/split_excel_paper.py) | 仅清理已有目录，没有创建缺失目录；运行前需要准备并检查输出路径 |
| 合并会删除原目录中的 Excel | [combine_excel_paper.py](../Source%20Code/Paper_Generator/combine_excel_paper.py) 的合并尾部 | 当前会删除所选目录所有 `.xlsx`，包括中间结果，先使用副本或备份 |

## 模型实验与评价

| 问题 | 代码依据 | 解释边界 |
| --- | --- | --- |
| 部分 API 异常被表示为空错误集合 | [jap_voting_machine.py](../Source%20Code/Paper_Generator/jap_voting_machine.py) 的 `check_for_error` 等 | 调用失败可能被上层当作无问题；应区分成功、失败、解析失败和未检查 |
| 投票失败时的有效模型数需要核对 | [jap_voting_machine_v3_4.py](../Source%20Code/Paper_Generator/jap_voting_machine_v3_4.py) 的 `get_voting_result` | 有模型报错时仍按配置总权重算阈值，票数含义会变化 |
| 投票组名和权重不是效果证据 | 同文件 `get_experiment_config` | High Quality、Fast 等是实验配置名，没有因此得到已验证的性能结论 |
| 比较只检查题干和选项变化 | [v3_2](../Source%20Code/Paper_Generator/jap_question_generator_v3_2_qwen_based.py) 的 `paper_comparison` | 仅修改答案列不会被识别，题目改变也不等于修正成功 |
| Correct Rate 是修改标记一致率 | 同文件 `batch_process_excel_files` 的位序列异或统计 | 统计包括双方均不修改的题，不能报告为答案正确率或教学效果 |
| 题号位序列假定每类 10 题 | `paper_comparison` 与 `extract_bad_questions.py` | 其他题量可能错位或越界，不能直接与每类 3 题的默认生成配置混用 |
| 比较失败时返回值数量不一致 | `paper_comparison` 若干提前返回路径 | 有的返回 2 项，调用处按 3 项解包；空文件或数量不匹配可能继续报错 |
| 分片流程没有完整兼容所有菜单模式 | [split_paper_test_pipeline.py](../Source%20Code/Paper_Generator/split_paper_test_pipeline.py)、[v3_4](../Source%20Code/Paper_Generator/jap_question_generator_v3_4_qwen_based.py) | 单模型模式读取不同输入目录，全部组模式返回 `all_voting_groups`，后续合并不能自动遍历各组 |
| v3 有额外依赖与不同默认根路径 | [v3](../Source%20Code/Paper_Generator/jap_question_generator_v3_qwen_based.py) | `langchain.prompts` 未在依赖中直接声明，路径带额外仓库目录层级 |
| 历史模型与参数需重新验证 | 各模型创建函数 | API 权限、模型名称、地域和思考参数不能只根据历史代码认定可用 |

## 界面、部署与目标差距

当前前端仅是 Tkinter 窗口，按钮依赖已有数据库记录。不存在浏览器前端、统一后端 API、上传服务、任务队列、登录权限或完整部署脚本。空查询、错误选项和异常恢复仍需完善。

当前知识点主要从文档标签读取，错题由标准答案比较得到。还没有完整的 LLM 学习障碍诊断、持久学习画像、按历史记录生成解释或自动复习调度。

仓库有微调样本准备代码，没有训练入口和模型评测。原 README 声称的 MIT 许可也没有相应 LICENSE 文件，本次文档已纠正这一描述。

## 本次文档核对范围

核对范围包括 Python 文件职责、实际调用关系、输入输出路径、数据库结构、模型配置读取方式、现有材料清单和逐目录文档覆盖。只做只读检查和文档一致性校验，不运行存在副作用的业务模块。正式交接验收仍需在修复上述关键问题后，以匿名数据完成[安装指南末尾的验收步骤](SETUP.md)。
