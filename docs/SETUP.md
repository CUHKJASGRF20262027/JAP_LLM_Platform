# 安装、配置与试用

[项目首页](../README.md) · [已知问题](KNOWN_ISSUES.md)

本指南把阅读材料、运行模型实验和运行桌面界面分开说明。当前源码有明确的运行阻碍，本指南不是已经通过验收的一键部署承诺。初次交接应由技术同事在隔离环境中完成配置，再把可用入口交给教师。

## 1. 选择要做的工作

| 工作 | 必须准备 |
| --- | --- |
| 看已有题目或教师评价 | Word、Excel 或兼容阅读软件 |
| 生成或修订题目 | Python 环境、依赖包、网络和 DashScope API 凭据 |
| 使用桌面界面 | Python 环境、Tkinter、运行中的 MySQL、兼容表结构及样例记录 |
| 看数据库表 | MySQL 连接信息，以及 DataGrip、MySQL 命令行或其他客户端 |
| 生成微调 JSONL | Python 环境、`python-docx` 和源文档 |

仓库没有网页前端的 `package.json`，不用执行 `npm install` 或 `npm run dev`。前端需要 Python 运行后才会显示桌面窗口。

## 2. 下载仓库

安装 Git 后，打开 PowerShell 或终端，在希望保存项目的位置执行以下命令。

```sh
git clone https://github.com/CUHKJASGRF20262027/JAP_LLM_Platform.git
cd JAP_LLM_Platform
```

后续命令均从此目录运行。GitHub 的 Code 菜单也可以下载 ZIP，但 ZIP 不包含 Git 版本历史，不适合直接提交修改。

## 3. 准备独立 Python 环境

建议以 Python 3.10 作为此历史依赖集合的起点，而不是照旧 README 使用任意 Python 3.8+。这是一项环境建议，并非完整兼容性测试结论。使用 Miniconda 或 Anaconda 时，在能识别 `conda` 的终端执行以下命令。

```sh
conda create -n japllm python=3.10
conda activate japllm
python -m pip install -r requirements.txt
python -m pip check
```

如果没有 conda，但已经安装 Python 3.10，Windows 也可使用独立虚拟环境。

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m pip check
```

后一种方式运行后续命令时，将 `python` 替换为 `.\.venv\Scripts\python.exe`，无需修改系统的 PowerShell 执行策略。`.venv/` 目前不在仓库的忽略规则中，不要把环境目录提交到 Git。

依赖中的 `python-docx` 用于读写 Word，`pandas` 和 `openpyxl` 用于 Excel，`mysql-connector-python` 用于连接数据库，`langchain_openai` 和 `langchain_core` 用于组织模型调用。包名中的 OpenAI 不表示本项目一定调用 OpenAI 服务，现有脚本配置的是 DashScope 兼容接口。

`requirements.txt` 是历史固定版本列表，包含多个当前入口未直接使用的包，也不保证覆盖全部历史脚本。例如 `v3` 还导入了列表中没有直接声明的 `langchain.prompts`。若安装或导入失败，应记录具体错误并检查目标脚本，不要无差别升级所有包。

## 4. 检查环境

```sh
python --version
python -c "import docx, pandas, openpyxl, mysql.connector, langchain_openai; print('Core imports OK')"
python -m tkinter
```

最后一个命令应打开一个小窗口，关闭窗口即可结束。这只检查 Tkinter，并没有连接项目数据库。[Python 官方说明](https://docs.python.org/3/library/tkinter.html)将此命令作为 Tkinter 安装检查方式。

如果没有 `_tkinter`，需要给所用 Python 环境安装或启用 Tcl/Tk 支持。单独安装依赖列表中的 `tk==0.1.0` 不能据此断定桌面组件已齐全。远程无图形桌面的服务器也不会直接弹出本机窗口。

## 5. 运行 LLM 实验前的配置

由项目负责人提供或分配 DashScope API 使用权限。现有脚本读取环境变量 `DASHSCOPE_API_KEY`，没有自动读取 `.env` 文件的代码。

在 Windows PowerShell 中，可以用以下方式为当前终端输入密钥，输入内容不会明文显示，也不必把实际密钥写入命令历史。

```powershell
$credentialInput = Read-Host "DashScope API key" -AsSecureString
$env:DASHSCOPE_API_KEY = [System.Net.NetworkCredential]::new("", $credentialInput).Password
Remove-Variable credentialInput
python -c "import os; print('API key configured' if os.getenv('DASHSCOPE_API_KEY') else 'API key missing')"
```

模型脚本须从这个终端运行。该检查只确认变量非空，不验证密钥、额度或模型权限。关闭终端后需要重新设置；也可按团队的凭据管理方式配置。

源码中的 API 地址为 `https://dashscope.aliyuncs.com/compatible-mode/v1`。密钥所属地域、调用地址、模型名称和思考模式参数必须匹配；以[服务方的调用说明](https://www.alibabacloud.com/help/en/model-studio/first-api-call-to-qwen)及账户实际可用模型为准。代码里的历史模型名不是当前可用性的保证。

### 按知识点出题

`v2_1` 是读取集中式 `config.py` 的一个入口。运行前先检查该配置中的输入路径、知识点范围、题型数量和每个知识点的题量。默认是第 1 到 8 个知识点、5 种题型、每个知识点共 15 题；这是目标生成量，不保证每次响应都满足要求。首次试跑应缩小到一个知识点，并使用新的输出目录。

```sh
python "Source Code/Paper_Generator/jap_question_generator_v2_1_qwen_based.py"
```

预期在配置指定的位置得到 Word 或 Excel 产物。查看终端是否存在失败、空输出、截断和最大修订轮数提示，再人工检查题干、选项和答案。成功保存文件不等于内容质量通过审核。

### 修订已有 Excel

```sh
python "Source Code/Paper_Generator/jap_question_generator_v3_1_qwen_based.py"
```

按终端菜单选择账户可用模型。默认读取 `docs/Generated_paper/shatin/`，保存到 `docs/revised_shatin/<模型名>/`。先改为实验副本路径或备份已有输出，避免覆盖历史同名结果。更详细的版本与投票流程见[出题目录说明](../Source%20Code/Paper_Generator/README.md)。

## 6. 准备 MySQL 后再运行界面

数据库服务、数据库和数据库客户端是三件不同的东西。

- MySQL Server 是运行在电脑或服务器上的服务，真正存储数据。
- `japgpt` 是代码中使用的数据库名称。下载仓库不会自动创建它。
- DataGrip 是连接数据库的图形客户端。它不是数据库服务器，也不是本项目运行的必需软件。

可以先把 MySQL 放在开发者自己的电脑上，使用 `localhost`。多人共用时，可以部署到一台团队管理的服务器，再由各电脑连接。此时 `localhost` 指向每个人自己的电脑，必须改为实际服务器地址，并由管理员配置账号和网络访问。

技术同事应按[数据库目录说明](../Source%20Code/SQL_Database/README.md)创建隔离测试库、准备表结构与匿名样例，并修复已知导入问题。不要直接执行旧建表或导入脚本来“自动准备数据库”。

如果选择 DataGrip，创建 MySQL 数据源，填写管理员提供的 Host、Port、User、Password 和数据库名，按提示下载驱动，然后 Test Connection，最后选择对应 schema 查看表。操作依据见[DataGrip 连接说明](https://www.jetbrains.com/help/datagrip/connecting-to-a-database.html)。关闭客户端不会等同于停止服务器。

数据库可用后，在 `Source Code/Front_End/interface.py` 的连接块中填入正确参数。当前配置散落在多个文件，修改一个文件不会自动改变其他脚本。先建立输出目录，例如在 PowerShell 执行以下命令。

```powershell
New-Item -ItemType Directory -Force "work/generated_practice"
python "Source Code/Front_End/interface.py"
```

出现窗口后，打开 `setting`，填入已经存在的输出文件夹绝对路径，查询数据库中确实存在的学生，选择试卷，再点击 `Analyze` 或 `Generate`。详见[界面使用说明](../Source%20Code/Front_End/README.md)。

## 7. 不调用模型的微调数据转换

```sh
python "Source Code/Fine_Tuning_Module/data_preprocessor.py"
```

该脚本会覆盖已有的 `docs/paper_with_feedback/kp_for_fine_tuning/japanese_vocab_unsloth.jsonl`。保留原文件或在脚本中把 `OUTPUT_JSONL` 改成新路径后再运行。它只转换数据，不开始训练，详见[模块说明](../Source%20Code/Fine_Tuning_Module/README.md)。

## 8. 常见现象和定位方法

| 现象 | 首先检查 |
| --- | --- |
| `python` 或 `conda` 无法识别 | 是否完成安装、是否在相应终端中激活正确环境 |
| `ModuleNotFoundError` | 当前解释器是否就是安装依赖的环境；历史脚本是否有额外导入 |
| 找不到 `docs/...` | 当前目录是否为仓库根目录；目标是否为尚未生成的输出目录 |
| `Access denied` 或无法连接 MySQL | MySQL 服务、Host、Port、用户名、密码和该账号权限 |
| `Unknown database` 或 `Table doesn't exist` | 库与表是否已建立；代码中的 `JAPGPT` 和 `japgpt` 是否统一 |
| 查询学生时报索引错误 | 数据库里是否有该学生；当前代码没有完整处理查询为空 |
| 生成练习为空或数量偏少 | 相关知识点是否有足够题目，题干是否满足旧排版规则 |
| API 返回 401、403 或模型不存在 | 密钥、地域、API 地址、模型权限和模型名称 |
| 输出文件不能保存 | 文件夹是否存在，文件是否正被 Excel 或 Word 占用 |
| 出现 JSON、题号或答案解析失败 | 模型输出格式是否匹配该版本的解析规则 |

## 9. 交接完成的建议验收

先使用不含真实个人信息的小样例。确认数据库连接、一个学生一次作答、错题与知识点对应、可审核的生成题、一次抽题导出和失败提示分别成立。进一步验证同一学生第二次作业不会删除第一次记录。完成这些检查后，才能把整条流程标为可用。

本次文档维护未调用付费模型 API，也未连接历史数据库或执行删表脚本。源码检查和文档校验不等于应用运行验收。
