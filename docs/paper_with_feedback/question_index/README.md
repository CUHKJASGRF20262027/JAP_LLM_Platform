# 教师问题题号索引

[返回项目首页](../../../README.md) · [上级目录](../README.md)

这里存放从教师反馈提取的机器可读索引，供 `jap_question_generator_v3_2_qwen_based.py` 比较模型修改位置。

`Bad Questions Index` 记录被标为需要修改的题号，`String_record` 是按题目位置排列的 0/1 序列，其中 1 表示需要修改。它记录的是题目质量问题，不是学生答错记录。

`extract_bad_questions.py` 可以重建这些文件，但会覆盖同名输出，且索引公式假定每种题型 10 题。改变试卷题量或编号后需要先修改、验证索引逻辑。

## 本目录的每个文件和子目录

| 文件或目录 | 用途 |
| --- | --- |
| `README.md` | 当前目录的阅读说明与逐项索引。 |
| [test_grammar_2_\['No. 91_～ば〈条件(じょうけん)〉 '\]_revised - with HO's Comments.xlsx](test_grammar_2_%5B%27No.%2091_%EF%BD%9E%E3%81%B0%E3%80%88%E6%9D%A1%E4%BB%B6%28%E3%81%98%E3%82%87%E3%81%86%E3%81%91%E3%82%93%29%E3%80%89%20%27%5D_revised%20-%20with%20HO%27s%20Comments.xlsx) | 与同名教师反馈对应的问题题号和 0/1 标记，供修改位置一致性比较。 |
| [test_grammar_3_\['No. 23_～たら〈条件(じょうけん)〉 '\]_revised - with HO's Comments.xlsx](test_grammar_3_%5B%27No.%2023_%EF%BD%9E%E3%81%9F%E3%82%89%E3%80%88%E6%9D%A1%E4%BB%B6%28%E3%81%98%E3%82%87%E3%81%86%E3%81%91%E3%82%93%29%E3%80%89%20%27%5D_revised%20-%20with%20HO%27s%20Comments.xlsx) | 与同名教师反馈对应的问题题号和 0/1 标记，供修改位置一致性比较。 |

以上清单对应交接时仓库已有材料。运行脚本产生的新文件应按实际用途记录，不能仅凭目录名推断审核状态。
