# 生成题与题库流程材料

[返回项目首页](../../README.md) · [上级目录](../README.md)

本目录保存从知识点生成的题目、用于修订实验的输入，以及按旧导入流程归档的 Excel 样例。

`original_grammar_questions/` 存初稿，`shatin/` 是现有 Excel 修订与切题流程的固定输入，`stored_grammar_questions/` 是旧题库导入脚本使用的归档位置。这三个目录的文件不会仅因为放在这里就自动进入数据库。

代码还使用 `revised_grammar_questions/` 和 `question_num_significance_test/paper/`，但代码基线没有提交这两个目录中的文件。前者用于生成后的修订题与导入输入，后者用于题量实验的分片。Git 不保存空目录，部分结果还被 `.gitignore` 忽略，首次运行应按对应脚本说明创建目录。

生成与修订可能覆盖同名文件。先使用单独的实验副本和输出路径，保留原始题目及教师反馈。

## 本目录的每个文件和子目录

| 文件或目录 | 用途 |
| --- | --- |
| `README.md` | 当前目录的阅读说明与逐项索引。 |
| [original_grammar_questions/](original_grammar_questions/README.md) | 生成初稿样例，点击查看内部每个文件。 |
| [shatin/](shatin/README.md) | 修订实验的输入题目，点击查看内部每个文件。 |
| [stored_grammar_questions/](stored_grammar_questions/README.md) | 旧入库流程的归档样例，点击查看内部每个文件。 |

以上清单对应交接时仓库已有材料。运行脚本产生的新文件应按实际用途记录，不能仅凭目录名推断审核状态。
