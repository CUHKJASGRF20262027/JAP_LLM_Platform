# 数据库模块

[项目首页](../../README.md) · [安装指南](../../SETUP.md)

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

## DataGrip 逐步操作指南

本节适合第一次接触数据库的使用者。已有团队数据库时，只需完成安装、连接和查看数据；只有准备全新测试环境时，才需要建库建表。以下步骤按官方界面说明和本仓库表结构整理，未连接实际教学数据库进行操作。

### 第一步 安装软件并确认数据库位置

从 [DataGrip 安装说明](https://www.jetbrains.com/help/datagrip/installation-guide.html)选择适合自己系统的安装方式，可以使用安装包或 JetBrains Toolbox。首次启动后按软件提示配置账号及适用的使用许可。

接下来确认 MySQL 已经运行。连接老师或团队的服务器时，无需在自己的电脑上再安装一个 MySQL Server；独立本机试用则需要另外安装并启动 MySQL Server。Windows 用户可在系统“服务”中检查实际安装的 MySQL 服务是否处于运行状态，服务名随安装配置而异。

先向数据库维护者取得以下信息，不要照抄代码中留下的旧地址或密码。

| 需要的信息 | 含义 | 本机测试示例 |
| --- | --- | --- |
| Host | MySQL 所在电脑或服务器地址 | `localhost` 或 `127.0.0.1` |
| Port | MySQL 接收连接的端口 | 通常为 `3306`，以安装配置为准 |
| User | 数据库账号，不是 GitHub 用户名 | 维护者提供的数据库账号 |
| Password | 该数据库账号的密码 | 安装时设置或维护者分配的密码 |
| Database | 服务器内具体使用哪个数据库 | 现有库使用其准确名称；下方练习新建 `japgpt_demo` |
| 网络接入方式 | 是否需要校园网、VPN 或 SSH 隧道 | 本机连接通常不需要 |

这里的 `localhost` 永远指当前电脑。假设 MySQL 在老师的服务器上，把 Host 填成 `localhost` 会连接自己的电脑，无法因此访问老师的数据。

### 第二步 在 DataGrip 添加 MySQL 连接

1. 打开 DataGrip，选择 `File → New → Data Source → MySQL`，也可以在 `Database Explorer` 中点击加号添加 MySQL 数据源。
2. 给连接取一个易辨认的名字，例如 `JAP 本机测试`。这个名称只是 DataGrip 内的连接标签，不会创建或重命名数据库。
3. 在 `General` 页面填写上表中的 Host、Port、User、Password 和 Database。已有数据库填写准确库名；尚未建库的本机测试连接暂时不指定 Database。
4. 如果出现 `Download missing driver files`，点击下载连接 MySQL 所需的驱动。驱动只帮助 DataGrip 通信，不会安装数据库服务器。
5. 点击 `Test Connection`。通过后保存连接；失败时先按本节末尾的故障表排查。
6. 在 `Schemas` 页面勾选需要显示的数据库，然后点击 `OK`。新建库后也可以回到这里重新选择。

菜单名称可能随版本和语言略有差异，具体位置可对照 [DataGrip 官方 MySQL 连接教程](https://www.jetbrains.com/help/datagrip/mysql.html)。测试连接成功只说明连接可用，并不说明项目需要的表和学生记录已经存在。

### 第三步 打开 SQL 查询窗口

在左侧选中刚才的连接，右键选择 `New → Query Console`。这个窗口用于把 SQL 指令发送到该连接对应的 MySQL 服务。执行前检查窗口绑定的连接和数据库，避免在另一个项目的库里操作。

可以先粘贴并执行下面的只读查询。选中要执行的语句后使用窗口的执行按钮，结果会出现在查询结果区域。

```sql
SELECT VERSION() AS mysql_version,
       DATABASE() AS selected_database;
```

如果第二列是 `NULL`，通常只是当前没有选择具体数据库。控制台操作和默认数据库选择见 [Query Console 官方说明](https://www.jetbrains.com/help/datagrip/query-consoles.html)。

### 第四步 仅在全新测试环境中建库建表

**连接已有教学库时跳过这一步。** 下方使用独立的 `japgpt_demo` 名称，不自动连接项目旧库。执行者需要建库建表权限，权限不足时由数据库维护者代为准备。

先单独运行建库语句。这里故意不使用 `IF NOT EXISTS`，如果提示同名库已经存在，应先确认内容，或换一个新的测试库名，不继续盲目执行后续建表语句。

```sql
CREATE DATABASE japgpt_demo
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

创建成功后，在同一查询窗口运行以下 SQL。这些字段对应当前程序的三张表，不包含删表指令，也不插入学生数据。

```sql
USE japgpt_demo;

CREATE TABLE questions (
    question_id INT AUTO_INCREMENT PRIMARY KEY,
    question_index VARCHAR(255) NOT NULL UNIQUE,
    content TEXT NOT NULL,
    correct_answer TEXT NOT NULL,
    type TEXT NOT NULL,
    level ENUM('N4', 'N5') NOT NULL,
    is_gpt BOOLEAN NOT NULL DEFAULT 0
) ENGINE=InnoDB;

CREATE TABLE students (
    student_id INT AUTO_INCREMENT PRIMARY KEY,
    student_no BIGINT NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE
) ENGINE=InnoDB;

CREATE TABLE exam_results (
    result_id INT AUTO_INCREMENT PRIMARY KEY,
    question_id INT NOT NULL,
    student_id INT NOT NULL,
    student_answer TEXT NOT NULL,
    is_correct BOOLEAN NOT NULL,
    FOREIGN KEY (question_id) REFERENCES questions(question_id) ON DELETE CASCADE,
    FOREIGN KEY (student_id) REFERENCES students(student_id) ON DELETE CASCADE
) ENGINE=InnoDB;

SHOW TABLES;
```

预期看到 `questions`、`students` 和 `exam_results`。表是空的属于正常现象，建表不会自动把仓库中的 Word 和 Excel 导入数据库。重复执行建表语句会提示表已存在，不需要为了重试而删除已有表。

### 第五步 查看题目、学生和答题记录

回到连接设置的 `Schemas` 页面，勾选新建的 `japgpt_demo`，刷新左侧列表。展开数据库及其表列表，双击一张表即可查看数据。双击表和结果区域的使用方式见 [DataGrip 数据查看器说明](https://www.jetbrains.com/help/datagrip/data-editor-and-viewer.html)。

也可以在查询窗口执行以下只读 SQL。如果使用已有库，将第一行替换为维护者确认的准确库名。

```sql
USE japgpt_demo;

SELECT COUNT(*) AS question_count FROM questions;
SELECT COUNT(*) AS student_count FROM students;
SELECT COUNT(*) AS answer_record_count FROM exam_results;

SELECT question_index, content, correct_answer, type, level, is_gpt
FROM questions
ORDER BY question_id
LIMIT 20;
```

前三个结果分别是题目数、学生数和作答记录数。最后一个查询显示前 20 道题目。空库会显示数量为 0 和空结果，不能因此判断连接失败。

需要查看某位学生的作答时，可以使用以下查询，把示例学号替换成测试库中已有的学号。

```sql
SELECT s.student_no,
       q.question_index,
       q.type AS knowledge_points,
       q.correct_answer,
       r.student_answer,
       r.is_correct AS stored_flag
FROM students AS s
JOIN exam_results AS r ON r.student_id = s.student_id
JOIN questions AS q ON q.question_id = r.question_id
WHERE s.student_no = 1000000001
ORDER BY r.result_id
LIMIT 100;
```

`1000000001` 仅为示例，本说明没有创建这名学生。查询保留原始对错标志而不替它转换成“正确”或“错误”，因为当前代码的字段含义存在上文说明的冲突。

### 第六步 让 Python 程序连接同一个数据库

DataGrip 和 Python 是两个分别连接 MySQL 的客户端。DataGrip 中保存的配置不会自动传给 Python，仍需要由技术同事同步下方“需要统一的连接位置”中各文件的参数。

如果 Python 也在数据库所在电脑运行，可采用 `localhost`；Database 应为实际使用的库名，例如本节的 `japgpt_demo`。如果 Python 运行在另一台电脑上，Host 必须改成它能够访问的数据库地址。使用 DataGrip 的 SSH 隧道时，隧道设置也不会自动供 Python 使用，需要单独准备相应连接方式。

到这里完成的是连接和表结构准备。要在项目界面中查询学生，还需要有效的题目、学生和作答记录。数据导入仍须先处理下方记录的旧脚本问题；不能把 DataGrip 能连接当作整个应用已经可用。

### 常见现象与处理

| 现象 | 首先检查 |
| --- | --- |
| 找不到 MySQL 数据源选项 | 使用 `File → New → Data Source` 查找 MySQL，而不是创建普通文件 |
| 驱动下载失败 | DataGrip 的网络或代理设置；这不等于 MySQL 密码错误 |
| `Connection refused` 或连接超时 | MySQL 是否运行、Host 和 Port 是否正确、是否接入必要网络 |
| `Access denied` | 数据库用户名、密码及该账号是否允许从当前机器连接 |
| `Unknown database` | 库名是否正确且已经创建，是否存在大小写差异 |
| 连接成功但左侧看不到库或表 | `Schemas` 是否勾选目标库，是否刷新，账号是否有访问权限 |
| `No database selected` | 查询窗口选择默认库，或先执行 `USE 准确库名;` |
| `Table already exists` | 该表已经创建，先查看内容，不要为消除提示直接删表 |
| 表里没有数据 | 是否只完成建表；项目材料不会自动导入 |
| DataGrip 能连接但 Python 不能 | 两边的地址、端口、账号、库名与隧道配置是否一致 |

关闭 DataGrip 只会关闭客户端，MySQL 服务仍可继续运行。数据库数据也不会因为关闭 DataGrip 而自动变成仓库中的文件。

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
