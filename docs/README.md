# 教学材料与文档索引

[返回项目首页](../README.md)

安装与运行指南已移到仓库根目录，见 [SETUP.md](../SETUP.md)。

这里同时保存项目说明、输入样例、教师反馈和历史实验产物。它不是一个自动同步的线上题库，也不是已部署数据库的备份。

非开发人员可先查看原试卷和标准答案，再打开 `Generated_paper/shatin/` 的题目与 `paper_with_feedback/4.11shatin/` 的教师反馈。需要了解程序时，再回到各代码目录 README。

Word 文件用 Word 或兼容软件打开，Excel 用 Excel 或兼容软件打开，JSONL 和日志可以用文本编辑器查看。`.docx` 和 `.xlsx` 在 GitHub 上可能只提供下载入口，不需要运行 Python 才能阅读。

材料名称中的 original、revised、stored 表示流程位置或历史文件命名，不是教学质量认证。学生样例包含个人信息的可能性需要由材料负责人核查，演示和继续分发应使用获得授权的匿名副本。不要把真实学生资料复制到新的公开说明里。

## 本目录的每个文件和子目录

| 文件或目录 | 用途 |
| --- | --- |
| `README.md` | 当前目录的阅读说明与逐项索引。 |
| [Generated_paper/](Generated_paper/README.md) | 生成题与题库流程材料，点击查看内部每个文件。 |
| [paper_with_feedback/](paper_with_feedback/README.md) | 教师反馈与词条材料，点击查看内部每个文件。 |
| [processed test paper/](processed%20test%20paper/README.md) | 试卷清理中间文件，点击查看内部每个文件。 |
| [revised_shatin/](revised_shatin/README.md) | 按模型组织的修订结果，点击查看内部每个文件。 |
| [student_test_sample/](student_test_sample/README.md) | 学生答卷格式样例，点击查看内部每个文件。 |
| [ROADMAP.md](ROADMAP.md) | 建议的开发阶段、目标数据库能力和验收标准，不是已完成功能清单。 |
| [Test 1 Model Answer.docx](Test%201%20Model%20Answer.docx) | Test 1 的标准答案样例，供逐题比较学生选择。 |
| [Test 1 Question Paper.docx](Test%201%20Question%20Paper.docx) | 原始试卷样例，供拆题和知识点读取，与标准答案和学生答卷配套使用。 |
| [new_db_test.docx](new_db_test.docx) | 旧学生导入流程生成的数据库内容 Word 报告样例，不是数据库备份或当前数据库状态。 |
| [test_grammar.docx](test_grammar.docx) | 知识点生成器默认读取的编号知识点文档；不同版本按自己的规则选择条目。 |

以上清单对应交接时仓库已有材料。运行脚本产生的新文件应按实际用途记录，不能仅凭目录名推断审核状态。
