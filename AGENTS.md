# 项目工作约定

先读 `PROJECT_WORKFLOW.md`。

## 每次推送后的固定部署

- 本项目或 `wrgmam` 提交推送后，及用户单独要求部署时，必须一起部署 `mizuki`、`wrgmam` 到全部五个 ESA 账户（国际一站、国内四站），共十项发布。
- **没有改动的项目也要一起部署，不能按 Git diff 跳过。** Windows 拉取两个仓库最新 master，构建并通过官方 ESA CLI 本地上传，不再依赖 ESA Git 授权构建。
- 权威流程与脚本在同级 `wrgmam` 仓库：`docs/ESA-Windows部署.md`、`scripts/deploy-esa-windows.ps1`。Windows 保存工具副本。
- 日常不删除项目、不修改域名 NS；每个账户核验生产版本和域名绑定，再检查独立调试域名的首页、文章、静态资源、404。未完成十项验收不能报告全部成功。
- 用户要求仅本地准备、不推送时不触发部署。凭据禁止入库。
