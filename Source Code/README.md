# 代码目录说明

[返回项目首页](../README.md)

本目录存放 Python 程序。各子目录是不同工作环节，并不是四个已部署的服务。初次接手可以先看输入文件和输出结果，再看对应程序。

| 子目录 | 给非开发人员的解释 | 主要输入 | 主要输出 |
| --- | --- | --- | --- |
| [Front_End](Front_End/README.md) | 使用者操作的桌面窗口 | 学生姓名或学号、已有试卷记录 | 分析窗口、抽题生成的 Word |
| [Paper_Generator](Paper_Generator/README.md) | 出题、修题、整理试卷和抽题的工具箱 | 知识点 Word、题目 Excel、标准答案、学生答案或数据库记录 | Word、Excel、修订日志、错题知识点统计 |
| [SQL_Database](SQL_Database/README.md) | 与教学档案数据库交互的程序 | 数据库连接参数、试卷和题目文件 | 三张表中的记录、查询结果和报告 |
| [Fine_Tuning_Module](Fine_Tuning_Module/README.md) | 将专家词条整理成未来训练可用的数据 | 复合动词 Word 文档 | JSONL 指令样本 |
| `README.md` | 本目录的地图 | 无 | 阅读说明 |

## 现有调用关系

- `Front_End/interface.py` 调用 `Paper_Generator/test_paper_generation.py`，后者查询 MySQL、统计错题并排版 Word，不调用 LLM。
- `SQL_Database/insert_db.py` 调用 `Paper_Generator/jap_paper_revise.py` 解析文档和比较答案，同时导入建表模块。该导入会删表，当前不能作为常规启动步骤。
- `Paper_Generator` 中的 `jap_question_generator_*` 调用模型 API，借助 `jap_excel_processor.py` 保存或读取题目表格。它们需要单独运行，目前没有被桌面窗口统一调度。
- `Fine_Tuning_Module/data_preprocessor.py` 独立转换文件，没有接入模型训练。

## 阅读和运行约定

所有运行命令默认在仓库根目录执行，也就是能看到 `requirements.txt` 和 `docs/` 的位置。`Source Code` 含有空格，命令中的文件路径必须加引号。各脚本大量使用相对路径，直接切换到本目录后运行可能找不到材料。

`__init__.py` 是 Python 模块目录的标记，当前三个同名文件均为空，不需要逐个运行。`test_paper_generation.py` 的名字虽然含有 `test`，实际是应用功能代码，并非自动化测试套件。

先按[安装指南](../docs/SETUP.md)准备环境，再阅读对应模块的前置条件。当前脚本保留了多个实验版本，不能用文件名中的版本数字推断哪个版本已经验证最好或适合全部场景。
