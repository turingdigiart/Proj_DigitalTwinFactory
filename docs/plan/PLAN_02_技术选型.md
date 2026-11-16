# PLAN_02｜技术选型（冻结）

```yaml
artifact_id: TECH-TWIN-001-PLAN-02
current_version: v001
record_type: Plan / 技术选型
status: Active（选型已冻结；改动需负责人确认并版本递增；验证结果回填 §5）
workspace_id: WS-ORG-001
data_class: internal
humanizer_status: ExceptionReviewed
exception_reason: 内部工程规划文档，非业务受众文案，不对甲方交付原文。
created_at: 2026-11-12
sources:
  - docs/research/RESEARCH_01_Web三维引擎对比调研.md（v001）
  - docs/research/RESEARCH_02_glTF压缩与轻量化调研.md（v001）
  - docs/research/RESEARCH_03_大屏与平板同步通信调研.md（v001）
  - 记录/SOURCE_proposal_v004_20261111.md（对外方案 v004 技术路线）
```

## 1. 冻结决策（2026-11-12）

| 层 | 选型 | 理由 | 主要风险与对策 |
|---|---|---|---|
| 三维引擎 | **Three.js**（WebGLRenderer 为主；WebGPURenderer 仅评估） | 生态/中文资料/数字孪生案例最多；团队已有工程经验与机构 Three.js Skills；渲染自由度大（R01） | 复杂交互要自己搭（用 R01 机制清单兜底） |
| 前端壳 | **Vite + TypeScript**（纯静态 SPA） | 构建快、类型安全；不需要重型框架（本项目是单页三维应用） | — |
| 大屏端 | 浏览器全屏运行（Chrome/Edge ≥120，目标机实测固定版本） | 需求 4.1.1「单机运行完整精度场景」；零安装 | 目标机显卡未知 → PLAN_05 性能预算验证 |
| 平板端（可裁） | 浏览器控制端（可加 PWA 安装；不发商店包） | 需求 4.2 双端（Windows/Android）用浏览器一套代码覆盖 | 若甲方裁剪，验收 I6 移除、架构保留 |
| 实时同步 | **WebSocket**（Node.js `ws`；JSON 指令协议 + 心跳/重连） | 双向低延迟、浏览器原生、内网单服务器足够（R03） | 断线重连策略在 C3 验证 |
| 后端 | **Node.js + Express**（数据 API + 后台 + WebSocket 网关同进程或双进程） | 与前端同栈、单人维护成本低；机构有 Node 工程经验 | 单机故障 → 内网单实例即可，文档写明重启方式 |
| 存储 | **SQLite**（优先 Node ≥22.5 内置 `node:sqlite`，免原生编译；备选 better-sqlite3） | 数据量小（设备/库位/看板配置）、单管理员、部署简单；与机构题库同套路 | 原生模块编译坑 → 首选内置模块规避 |
| 三维资产 | **glTF 2.0** + Draco 为主、gltfpack/meshopt 评估；纹理量大再上 KTX2（R02） | 标准格式 + Blender 直出（机构 `TOOL-ORG-BLENDER-001`）；C2 实测后定组合 | 工业模型不可导出 → PLAN_04 管线规划 |
| 数据 | 静态文件导入（CSV/JSON/XLSX 口径 C4 冻结）+ 后台表单 | 需求「静态数据读取或表单录入」 | — |
| 部署 | 内网服务器；大屏主机浏览器 + 服务器运行 Node 服务 | 报价口径「无需 GPU 渲染服务器」 | 服务器配置未知 → PLAN_03 部署拓扑细化 |

## 2. 不选清单（明确边界）

- **Unity/Unreal 原生双端**：二期可选，不进本期（报价口径）。
- **重型前端框架**（React/Vue 全家桶）：单页三维应用不需要，避免构建与学习成本；后续看板复杂度上升可局部引入组件库（仅 UI 层）。
- **MQTT/物联网网关**：本期无真实设备接口；二期接真实数据时再评估。
- **GPU 渲染服务器 / 云渲染**：报价已明确不需要。

## 3. 最小验证用例计划（选型落地证据，本机先行，不依赖甲方）

| 用例 | 内容 | 判定标准（暂定，实测后回填） |
|---|---|---|
| C1 引擎冒烟 | Vite+TS+Three.js：200 个 InstancedMesh 设备实例 + LOD + 状态配色切换，大屏分辨率运行 | 1920×1080 ≥30 FPS（本机实测记录帧率） |
| C2 资产链路 | Blender 出 1 个测试设备 → glTF（原始/Draco/gltfpack 三路）→ 加载 | 单设备体积 <500 KB；加载时长与目视损失记录（R02 回填） |
| C3 指令同步 | Node `ws` + 两浏览器页（大屏/平板模拟）：视角/楼层/状态指令 | 局域网指令往返 <200 ms；断线重连可用（R03 回填） |
| C4 数据后台 | Express + SQLite：文件导入设备清单 → 表单改状态 → 大屏 WebSocket 刷新 | 全链路手动走通，数据落库可回读 |

- 执行顺序：C1 → C2 → C3 → C4（每项产出回执：脚本/步骤/数据/结论，存 `验证/` 目录）。
- 验证失败不推翻选型，先记录根因再决定（如 C1 不达标先查实例化与材质切换，再考虑降面或关特性）。

## 4. 依赖与版本管理

- `package-lock.json` 入库；依赖清单：`three`、`vite`、`typescript`、`express`、`ws`（+ 可选 `better-sqlite3` 备选）；避免临时性大依赖。
- Node 版本口径：本机与部署机统一 LTS（开发时记录实测版本）；`node:sqlite` 需要 ≥22.5。
- 代理/网络：本机 npm 可直连（机构经验：npm registry 直连可用，GitHub 需代理）。

## 5. 验证结果回填区（Pending）

- C1：待执行（预计 PLAN_03 后启动）。
- C2：待执行。
- C3：待执行。
- C4：待执行。
