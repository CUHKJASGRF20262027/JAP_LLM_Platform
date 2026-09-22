# 数据库模块

[项目首页](../../README.md) · [安装指南](../../docs/SETUP.md) · [已知问题](../../docs/KNOWN_ISSUES.md)

这里存放连接和操作 MySQL 的 Python 脚本。数据库的真实数据由 MySQL Server 保存在部署它的机器上，不保存在这些 Python 文件里。仓库没有可直接还原完整业务数据的数据库备份。

## 每个文件的用途

| 文件 | 负责的工作 | 输入与输出 | 当前注意事项 |
| --- | --- | --- | --- |
| [db_question_students_results.py](db_question_students_results.py) | 定义三张表和常用 SQL，并连接、删除和重建表 | 连接目标库后建立 `questions`、`students`、`exam_results` | **导入文件也会执行删表和重建，随后关闭连接**；不能作为无副作用的配置模块 |
| [insert_db.py](insert_db.py) | 从试卷、标准答案和学生答卷提取数据并写入数据库，导出 Word 报告 | 默认读取 `docs/` 的 Test 1 和学生样例；写入三表，导出 `docs/new_db_test.docx` | 导入上一个文件会删表；还存在 SQL 参数数量错误、重复题目插入和删除学生历史记录等问题 |
| [Excel2Db.py](Excel2Db.py) | 将生成题 Excel 导入题库，然后移动文件到归档目录 | 从 `revised_grammar_questions/` 到数据库与 `stored_grammar_questions/` | 活跃逻辑依赖旧 Vocabulary 文件名和题干，不能直接接收全部现有语法题；导入范围包含 `Low_Q` |
| [join_search.py](join_search.py) | 提供 `join_search()`，按姓名或学号联合查询三张表，打印错题信息 | 终端输入学生信息，终端输出记录 | 只定义函数，单独执行该文件不会自动打开查询流程；查询字符串仍需要参数化改造 |
| [__init__.py](__init__.py) | Python 包标记 | 无 | 空文件，不需要单独运行 |
| `README.md` | 数据库部署、数据结构和文件说明 | 无 | 当前文档 |

## 数据库中的三种记录

| 表 | 通俗解释 | 当前字段 |
| --- | --- | --- |
| `students` | 学生名册 | `student_id` 内部编号，`student_no` 学号，`name` 姓名，`email` 邮箱 |
| `questions` | 题目卡片 | `question_id` 内部编号，`question_index` 业务题号，`content` 题干和选项，`correct_answer` 标准答案，`type` 知识点文本，`level` N4/N5，`is_gpt` 是否由 AI 生成 |
| `exam_results` | 某个学生对某道题的一次作答记录 | `result_id` 记录编号，`question_id` 关联题目，`student_id` 关联学生，`student_answer` 学生答案，`is_correct` 对错标志 |

`student_id` 和 `question_id` 把作答记录与学生、题目关联起来。程序因此能找到“某个学生在哪些知识点上答错”，再找同知识点题目。当前没有单独的作业批次、提交时间、题目审核、解释或标准知识点表，因此还不能完整支持目标中的长期学习档案。

**`is_correct` 的实际含义存在冲突。** 答案比较函数返回 0 表示正确、1 表示错误；导入脚本直接保存此值，GUI 和 `join_search()` 把 1 当作错题。但建表模块的 `select_mistake_query` 使用 `is_correct = 0`。接手时必须先统一约定并核对已有记录，不能只按字段英文名称理解。

## MySQL Server 与 DataGrip 的区别

MySQL Server 是必须实际运行的数据库服务。DataGrip、MySQL Workbench 或命令行 `mysql` 是连接该服务的客户端，可以任选合适的工具。安装 DataGrip 后仍需有数据库服务器、账号、数据库和表。

单人开发可以把服务装在自己的电脑上。团队共用可放在项目服务器上，所有人用同一套经过授权的连接参数。服务器应由明确的维护者负责启动、备份、恢复和账号管理。仓库中留下的历史内网地址不是已经验证可访问的公共服务。

## 由技术同事准备隔离测试库

1. 安装并启动 MySQL Server，或取得团队测试服务器的连接信息。首次验证先使用独立测试库，不要指向既有教学数据。
2. 使用管理员账号连接服务。先创建空数据库，再在该库中准备表结构。[MySQL 建库说明](https://dev.mysql.com/doc/refman/8.4/en/creating-database.html)说明了数据库创建与选择步骤。
3. 不运行现有删表脚本。可阅读 `db_question_students_results.py` 中的三个 `CREATE TABLE` 定义，在空库中单独执行，依次创建 `questions`、`students`、`exam_results`，不执行任何 `DROP TABLE`。
4. 请管理员提供专用开发账号。界面主要需要查询权限；数据导入需要相应写入权限。不要把应用日常运行绑定到管理员 root 账号。
5. 先修复下文列出的导入问题，再导入匿名样例；或者由开发者手工插入一套已知答案和知识点的测试记录。
6. 在客户端核对记录数量、关系和对错含义，再启动桌面界面。

在确认选用本机独立测试库后，建库示例可以采用以下 SQL。它只创建库，不创建三张表，也不会导入数据。

```sql
CREATE DATABASE IF NOT EXISTS japgpt
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE japgpt;
```

UTF-8 字符集用于保存日文、中文等内容。仓库不同文件混用 `JAPGPT` 和 `japgpt`，跨机器时可能产生大小写问题，应统一数据库名称。

## 需要统一的连接位置

| 文件位置 | 影响范围 |
| --- | --- |
| `Front_End/interface.py` 顶部连接块 | 桌面界面 |
| `SQL_Database/db_question_students_results.py` 顶部连接块 | 建表与学生导入链路 |
| `SQL_Database/Excel2Db.py` 顶部连接块 | Excel 题库导入 |
| `SQL_Database/join_search.py` 的 `join_search()` 内 | 终端查询 |
| `Paper_Generator/test_paper_generation.py` 的主程序块 | 该文件单独执行时的连接 |

当前没有统一的数据库环境变量配置，创建 `DB_HOST` 或 `.env` 并不会自动让这些脚本生效。技术同事可以在本地修改连接块完成隔离验证，后续应重构为统一配置，避免把真实密码提交到仓库。

## 两条导入路线

### 作业和学生答案

设计流程是题目 Word、标准答案 Word、学生答卷 Word → 文档解析 → 按顺序比较答案 → 写入三张表 → 导出报告。

当前 `insert_db.py` 默认逐个读取学生样例，但每个学生的报告保存到同一个 `new_db_test.docx`，会覆盖同名报告。它还会按学生删除原有 `exam_results`，没有保留多个作业批次。学生邮箱也是由学号拼接而来，并未验证真实邮箱。

### 已生成并审核的题库

设计流程是题目 Excel → 确认审核标签 → 导入 `questions` → 归档处理过的 Excel。

当前 `Excel2Db.py` 只在文件名含 `Vocabulary` 时初始化若干关键变量，且只识别一组固定词汇题干。现有 `test_grammar_...` 文件可能触发变量未定义，不能直接视为兼容输入。它还把 `High_Q`、`Minor changes` 和 `Low_Q` 都纳入导入条件，并在扫描结束后移动文件，不能把归档动作当作“已经审核合格”的证据。

## 恢复完整导入前的最小修复清单

- 将 SQL 常量、连接函数和显式初始化分离，导入模块不能删表、写库或关闭共享连接。
- 统一对错字段和历史数据约定，并用一题答对、一题答错验证整个查询链路。
- 修复学生更新 SQL 的两个占位符对应三个参数的问题。
- 处理唯一题号的重复插入，不把普通 `INSERT` 当作已有的 upsert。
- 增加作业或提交批次，避免导入新作业时删除学生旧作业。
- 让 Excel 文件名、知识点、题干格式和审核标签与实际材料一致，仅将明确通过审核的题目发布到可抽题题库。
- 用事务和导入记录区分成功、失败、跳过，再决定是否归档文件。

以上为文档记录的待修复项，本次交接说明没有替你运行这些脚本或改动数据库。
