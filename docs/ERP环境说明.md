# ERP 自动化执行条件

文档生成工作台不依赖 ERP。以下仅适用于接口、UI、Locust 业务测试。

- 被测系统固定 jshERP v3.5，后端 `http://127.0.0.1:9999/jshERP-boot`，前端 `http://127.0.0.1:3000`。
- Redis 在本机 16379，验证码由只读客户端获取；不得关闭验证码验证。
- 项目根目录创建 `runtime/local.json`，包含本地 `redis_password`，定向数据库清理还需 `db_password` 等原环境配置。不要把真实配置提交到 Git。
- 权限/UI 测试需要 `runtime/identities.json` 中独立的管理员、业务操作员和只读身份；字段参见测试代码。不得把真实身份文件提交到 Git。
- 浏览器默认读取 `tools/pw-browsers`；可在安装前配置 `PLAYWRIGHT_BROWSERS_PATH` 后执行 `python -m playwright install chromium`。
- 测试用例通过 `support/execution_lock.py` 互斥使用环境；通过的测试也默认留存数据，只有明确指定 `--cleanup-passed` 才尝试定向清理。
- 清理只允许项目隔离数据库 `jsh_erp_qa`，会核验运行数据目录与 manifest，缺少完整原配置时不要开启清理。

本仓库不包含旧的部署工具和初始化脚本。自行部署时应满足上述条件；测试收集成功不代表真实环境执行通过。
