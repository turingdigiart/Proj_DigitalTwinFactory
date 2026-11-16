# TECH-TWIN-001｜数字工厂三维数字孪生（一期）工程仓库

```yaml
artifact_id: TECH-TWIN-001-REPO-ENTRY
current_version: v008
git_remote: git@github-org:turingdigiart/Proj_DigitalTwinFactory.git（私有）
push_identity: GH-ACC-002 turingdigiart（SSH Host 别名 github-org）
repo_boundary: DEC-ORG-091——本仓库只收工程与资产；报价/运营/商务资料以同步盘为唯一载体
workspace_id: WS-ORG-001
data_class: internal
updated_at: 2026-11-16
```

## 仓库边界（DEC-ORG-091，2026-11-16）

- **入库**：代码、三维/媒体资产、技术规划与研究（`docs/plan`、`docs/research`）、构建与验证配置。
- **不入库**：报价交付物、谈判与商务记录、客户需求资料与聊天证据、运营与对外文案底稿、来源/哈希台账、项目状态（含报价数字）。这些以同步盘 `开发/TECH-TWIN-001_数字工厂数字孪生/` 为唯一载体（`交付/`、`历史版本/`、`记录/`、`PROJECT_STATUS.md`）。
- 历史说明：2026-11-12 迁入 `${DEV_ROOT}` 并本地 Git 化（38 文件 SHA-256 一致）；2026-11-16 按 `DEC-ORG-091` 重建工程史并首推远端；含商务资料的旧完整历史只保留本地分支 `main-full-backup-20261116`，**不进远端**。

## 目录

```text
docs/
  plan/        PLan_00 总纲、PLAN_01 范围基线、PLAN_02 技术选型、PLAN_03 架构、
               PLAN_04 资产管线、PLAN_05 性能预算、PLAN_06 里程碑排期、PLAN_07 RAID
  research/    RESEARCH_01 引擎对比、RESEARCH_02 glTF 压缩、RESEARCH_03 同步通信
apps/          （C1 启动时落地：web-3d 大屏 / web-ctrl 平板 / web-admin 后台）
server/        （C3/C4 启动时落地：Express + ws + SQLite）
验证/          C1–C4 最小验证用例回执
```

## 权威状态与台账

- 权威状态：中央 `项目管理/PROJECT_REGISTRY.md` + 同步盘 `开发/TECH-TWIN-001_数字工厂数字孪生/PROJECT_STATUS.md`。
- 远端与协作者：中央 `.codex/registries/DEV_PROJECTS_MANIFEST.md`；推送身份与凭证见 `DEC-ORG-089/090`。

## 下一步

- C1 引擎冒烟（200 实例 + LOD + 状态切换基准）→ C2 资产链路 → C3 指令同步 → C4 数据后台（见 `docs/plan/PLAN_02_技术选型.md` §3）。
