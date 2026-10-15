# AI 测试用例生成与评审工作台

使用 Python 实现的文档驱动测试用例生成工具。用户上传需求或接口文档后，可生成结构化测试点与测试用例，在网页中修改、审核，并导出交付文件。

## 功能

- **文档解析：** 支持 TXT、Markdown、DOCX 和文字型 PDF，单文件不超过 3 MB、提取文字不超过 32000 字符。
- **用例生成：** 输出测试点、用例名称、优先级、类型、前置条件、操作步骤、预期结果和待确认项。
- **需求追溯：** 为识别出的需求分配编号，将用例关联到需求原文和提取文本行号。
- **结果校验：** 检查模型响应格式、必填字段、优先级、来源编号及需求覆盖；不合法结果不保存为成功用例。
- **人工评审：** 网页逐条修改用例，保存审核备注，标记待审核、已确认、需修改或已排除。
- **文件导出：** 导出 Excel（XLSX）、CSV、Markdown、JSON，包含最新修改和审核状态。
- **错误提示：** 区分缺少密钥、认证失败、额度限制、连接异常和生成超时，不展示密钥或原始错误响应。

## 快速启动

需要 Python 3.12 或更新版本。在仓库根目录打开 PowerShell，逐行执行：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m qa_workbench.ai_testing.server
```

浏览器打开 **http://127.0.0.1:8765**。页面由 Python 生成 HTML，使用原生表单提交，无 JavaScript 或前端构建依赖。

停止服务：在运行窗口按 `Ctrl+C`。

## 生成模式

### 本地规则模板

不需要 API 密钥、不调用外部模型。程序根据需求关键词匹配正常流程、数值边界、权限或异常输入等候选方向，供人工补充。

该模式是规则模板，不具备模型的语义理解能力，结果会明确标注“未调用模型”。

## 使用流程

1. 上传需求、接口说明或测试文档，并选择生成模式。
2. 查看生成用例，核对需求原文、操作步骤、预期结果及待确认项。
3. 修改用例，选择审核状态，保存修改与备注。
4. 点击导出链接下载 Excel、CSV、Markdown 或 JSON。

可上传 [登录权限需求示例](docs/登录权限需求示例.md) 体验。

## 处理链路

```text
文档上传 → 文件校验与文字提取 → 需求筛选与编号
        → 本地规则 / 模型分批生成 → 结构与需求关联校验
        → 网页编辑审核 → 导出
```

AI 模式每批最多分析 6 条需求。这里没有向量化入库或检索流程，不属于 RAG。上传文档只作为待分析资料，不执行其中的指令。

## 自动化验证

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s qa_workbench/ai_testing/tests -p "test_*.py" -v
```

当前有 12 项自动化验证，覆盖需求关联、本地生成、模型响应结构及遗漏检查、文档解析、导出、错误处理，以及 HTTP 上传—编辑—审核—下载流程。

GitHub Actions 执行同一组验证。模型响应使用模拟数据，不消耗 API 额度；这验证的是工作台自身，不代表真实模型服务验收或用户业务测试已执行。

## 目录

```text
qa_workbench/
  __init__.py
  ai_testing/
    case_generator.py   # 需求拆分、生成与结构校验
    file_parser.py      # 文档文字提取
    llm.py              # 模型客户端与错误处理
    server.py           # Python 网页服务、编辑与审核
    case_export.py      # Excel、CSV、Markdown、JSON 导出
    tests/              # 工作台自身自动化验证
docs/                   # 操作说明与示例需求
.github/workflows/      # GitHub Actions 验证
requirements.txt        # requests、pypdf
.env.example            # 配置说明，不包含真实密钥
```

每次上传的原文件、提取文本和生成结果保存在 `qa_workbench/artifacts/ai/case_jobs/<任务编号>/`，该目录不提交到 Git。

## 已知边界

- 生成的是待审核的测试用例文档，不生成或执行自动化测试脚本。
- 需求筛选使用关键词和句子规则，可能漏判、误判，跨段依赖需人工核对。
- 来源行号属于提取文本，不等同于 Word/PDF 原始页码。
- 扫描 PDF 需要先 OCR；超限文档需拆分上传，不会静默截断。
- 字段完整和需求关联有效，不代表业务预期正确或所有测试场景均已覆盖。

详细操作见 [使用说明](docs/用例生成工作台说明.md)。
