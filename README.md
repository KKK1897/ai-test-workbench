# AI 测试用例生成与评审工作台

Python 实现的文档驱动测试用例生成、人工评审和导出工具，同时保留接口、UI 与性能自动化测试示例。

## 功能

- 上传 PDF、DOCX、TXT、Markdown，提取需求并生成结构化测试点与用例。
- 关联需求原文及提取文本行号，校验字段、需求来源和需求覆盖。
- 支持本地规则模板与配置模型的 AI 生成；两种模式明确区分。
- 网页逐条编辑、审核，导出 Excel、CSV、Markdown、JSON。
- 模型认证、额度、连接超时等错误有独立提示，不展示密钥。

生成用例需人工审核，并不代表已执行测试。PDF 扫描件需要 OCR；需求筛查和模型语义判断仍有局限。

## 快速启动

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m qa_workbench.ai_testing.server
```

浏览器打开 http://127.0.0.1:8765 。工作台独立运行，无需 ERP、数据库或 Node.js。

需要 AI 时，在同一 PowerShell 窗口、启动前配置：

```powershell
$env:AI_TEST_API_KEY="你的密钥"
$env:AI_TEST_API_BASE="https://api.deepseek.com/v1"
$env:AI_TEST_MODEL="deepseek-chat"
```

默认地址适用于 DeepSeek 官方；其他平台请使用对应地址和模型。程序读取环境变量，不自动读取 `.env`。

## 验证

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s qa_workbench/ai_testing/tests -p "test_*.py" -v
```

GitHub Actions 运行同一组离线测试，模型使用模拟响应，不消耗 API 额度。

## 目录

```text
qa_workbench/
  ai_testing/        # 文档生成、评审、导出及对应验证
  helpers/           # ERP 接口客户端与测试数据管理
  tests/             # 接口、业务、权限和 UI 测试
  performance/       # Locust 任务与负载组织
  postman/           # 接口集合及环境示例
support/             # ERP 锁与定向清理所需的工具
docs/                # 必要业务规则、缺陷记录和使用说明
.github/workflows/   # 离线验证 CI
```

## ERP 自动化测试

安装 `requirements-qa.txt` 后，可在已配置的隔离 jshERP 环境中执行 `python -m pytest`。这些用例不是无环境的单元测试；需先配置本机 ERP、Redis 验证码读取、测试身份及 `runtime/local.json`。执行会创建业务数据，失败现场默认留存。

端口与配置要求见 [ERP 环境说明](docs/ERP环境说明.md)。本仓库不打包 ERP 源码、数据库、下载工具或本机凭据。

## ERP 测试设计与历史验证


被测系统固定为[jishenghua/jshERP](https://github.com/jishenghua/jshERP) v3.5，Commit `7ca0f9d68fef761d98b00854d09e975eba291465`。本项目的工作是测试设计、自动化脚本、执行、报告与缺陷分析。ERP财务验证指内部收付款结算，不包含外部支付网关。

- 19条基础接口用例＋3条进阶结算用例，覆盖登录会话、基础资料、采购入库、调拨、销售、权限和必要异常。
- 完整链路：登录→A仓采购100件→付款1000元→调拨30件至B仓→B仓销售10件→分次收款60元与90元。核对6张单据、A仓70件/B仓20件/总库存90件、欠款及账户余额2000→1150元。
- 3条Playwright UI流程：登录退出、页面采购销售、只读新增按钮限制；条件等待、截图和Trace辅助定位失败。
- Locust按10/30/50/100虚拟用户逐级查询库存，检查HTTP与业务码，保存P50/P95、吞吐量、错误率和CPU/内存观察。仅允许本机隔离环境，共享执行锁避免与功能测试混跑。
- fixture管理独立会话、唯一QA标识与定向清理；失败保留现场及脱敏证据，不以跳过或无条件重试掩盖缺陷。

### 已执行结果与边界

|范围|已记录结果|报告|
|---|---|---|
|基础接口19条|17通过、2失败，保留产品缺陷|[接口/UI历史报告](docs/测试报告.md)|
|UI 3条|修正测试脚本后3通过，保留初轮失败记录|[接口/UI历史报告](docs/测试报告.md)|
|收付款3条|3通过；另验证完整链路与原库存链路定向清理，2通过|[进阶结算验证](docs/进阶收付款链路.md)|
|10/30/50/100用户负载|四档均执行；48209次请求、11次失败。100用户P95约715ms、吞吐约175次/秒，低于50用户档|[逐级负载验证](docs/逐级负载验证.md)|

以上来自不同轮次，不是一次全量回归或全量产品验收。负载测试记录到业务500与Redis连接异常，尚未确认根因；不宣称100用户稳定通过或测出最大容量。历史并发重复单号风险见[缺陷记录](docs/缺陷记录.md)。Postman材料已检查，尚未在Postman中执行。


## 文档

- [工作台操作说明](docs/用例生成工作台说明.md)
- [体验需求](docs/登录权限需求示例.md)
- [业务规则与测试用例](docs/业务与测试用例.md)
- [缺陷记录](docs/缺陷记录.md)

被测 ERP 来源为 [jishenghua/jshERP](https://github.com/jishenghua/jshERP)，v3.5 / `7ca0f9d68fef761d98b00854d09e975eba291465`；ERP 本身不是本项目开发成果。历史缺陷与报告只针对该基线，不代表所有测试通过。原始复杂版本、归档、运行产物与上传文件均未纳入当前发布目录。
