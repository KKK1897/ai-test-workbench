# AI 辅助测试工作台与 jshERP 自动化测试

被测 ERP 来自开源 jshERP。本项目包含文档驱动的测试用例生成与评审工作台，以及既有进销存自动化回归测试，两者用途独立。

## Python 测试用例生成与评审

上传 TXT、Markdown、DOCX、文字型 PDF，生成带需求来源的结构化测试点与测试用例；支持网页编辑审核，导出 XLSX、CSV、Markdown、JSON。默认本地规则模式，选用 AI 模式需配置模型密钥。生成内容尚未执行测试。

首次运行（仅文档生成工作台，无需启动 ERP / MySQL / Redis）：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-workbench.txt
.\.venv\Scripts\python.exe -m beginner.ai_testing.server
```

打开 http://127.0.0.1:8765 。页面与业务处理由 Python 完成，无 JavaScript 或 Node.js 依赖。

需要 AI 模型时，在启动服务前设置环境变量（以下默认使用 DeepSeek 官方服务；第三方平台需改地址与模型名称）：

```powershell
$env:AI_TEST_API_KEY="替换为自己的密钥"
$env:AI_TEST_API_BASE="https://api.deepseek.com/v1"
$env:AI_TEST_MODEL="deepseek-chat"
```

密钥不得提交；`.env.example` 仅是说明，程序不会自动加载 `.env`。

### 自动验证

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s beginner/ai_testing/tests -p "test_*.py" -v
```

GitHub Actions 执行相同离线验证，模型响应使用替身，不需要 API 密钥，不访问 ERP。检查范围包括结构化生成、需求来源、响应校验、上传、编辑审核、导出和安全错误提示。

详见 [用例生成工作台说明](beginner/docs/用例生成工作台说明.md)，可上传 [登录权限需求示例](beginner/docs/登录权限需求示例.md) 体验。

## jshERP 测试

既有接口、UI、库存和权限测试参见 [测试项目说明](beginner/README.md)。历史质量报告与产品缺陷证据保留在 docs 和 beginner/docs；用例生成不代表这些回归已通过。
