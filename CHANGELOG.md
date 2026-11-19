# Changelog

All notable user-visible changes to the Psitransfer TOS application package.

## 2.4.1-4 — 2026-11-19

### Changed（针对 2.4.1-2 审核报告 C3 / C7 / T1）
- **C3（隐私政策缺失）**：重写 `PRIVACY.md`，覆盖本应用真实的数据面（上传文件与元数据、访问日志含
  remote IP、无数据库/无账号）、保留期（每次上传自选，默认 7 天；`maxAge` 75 天硬上限；一次性链接
  首次下载即删）、安全措施（uid 1000、单端口、`/admin` 默认关闭）、第三方披露（无遥测、无更新检查，
  可选 webhook）。
- **C7（用户数据权利渠道）**：政策 §4/§5 明确「访问 / 更正 / 删除 / 限制」四类权利的具体操作路径
  ——上传者用分享链接自助删除、NAS 主人用 `/admin` 页或直接删除 `data/<bucket-id>`、以及
  `moechz@163.com`（30 天内答复）作为辅助与遗失链接的渠道；README 同步。
- **C3 可达性**：包内 `config.ini.help` 改为
  `https://github.com/Moechz/psitransfer/blob/main/PRIVACY.md`，23 语 `important` 也写明该链接与
  用法/保留期。
- **文案纠偏（F5 风险）**：`descript`/`important` 不再写“首次使用请完成初始化”（本应用没有初始化
  向导，`/admin` 需设 `adminPass`），改为真实操作说明与保留期默认值。
- **NOTICE**：补上游项目地址 <https://github.com/psi-4ward/psitransfer>、许可证 **BSD-2-Clause**、
  镜像 digest，打包部分许可纠正为 MIT（与 `LICENSE` 一致）。
- **T1**：上游 v2.4.6 已发布；本环境镜像加速器（docker.1panel.live 等）当前无法拉取镜像层，
  本轮保持 `v2.4.1` 并记录待办，拉取恢复后立即 pin 新 tag 重发（见 `docs/TASK_STATE.md`）。

## 2.4.1-2 — 2026-11-12

### Changed
- Icon: replaced the letter monogram with a functional vector mark that describes what the app does (the upstream project does not publish a compliant SVG).

## 2.4.1-1 — 2026-11-10

### Added
- First release. Psitransfer (`psitrax/psitransfer:v2.4.1`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8045 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/psitransferdocker/`.
