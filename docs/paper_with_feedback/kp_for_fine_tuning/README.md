# 复合动词与指令数据

[返回项目首页](../../../README.md) · [上级目录](../README.md)

这里保存复合动词源文档和由预处理程序输出的指令样本。源文档供教师阅读、修订词义和例句，JSONL 供后续开发者准备模型训练。

当前 JSONL 有 84 行，每行包含 `instruction`、`input`、`output`。这只是已有样本的数量，不能据此认定完成了微调或形成独立测试集。源材料名称中的 NINJAL 不等于仓库已经完整说明再分发许可，继续使用前应确认来源和授权。

`Source Code/Fine_Tuning_Module/data_preprocessor.py` 读取 Word 正文段落并按标题拆分词条，会覆盖同名 JSONL。它不调用模型 API、不训练模型。

## 本目录的每个文件和子目录

| 文件或目录 | 用途 |
| --- | --- |
| `README.md` | 当前目录的阅读说明与逐项索引。 |
| [84 compound verbs from NINJAL (Updated)_17Nov2025.docx](84%20compound%20verbs%20from%20NINJAL%20%28Updated%29_17Nov2025.docx) | 复合动词词条源文档，包含词义与例句，供教师阅读和预处理程序转换。 |
| [japanese_vocab_unsloth.jsonl](japanese_vocab_unsloth.jsonl) | 已提交的 84 条指令样本，每行一个 JSON 对象；不是模型权重。 |

以上清单对应交接时仓库已有材料。运行脚本产生的新文件应按实际用途记录，不能仅凭目录名推断审核状态。
