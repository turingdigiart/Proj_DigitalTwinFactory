# PLAN_03｜系统架构方案

```yaml
artifact_id: TECH-TWIN-001-PLAN-03
current_version: v001
record_type: Plan / 系统架构
status: Active（v001 基线；接口契约字段级在 C3/C4 实现时冻结并回填）
workspace_id: WS-ORG-001
data_class: internal
humanizer_status: ExceptionReviewed
exception_reason: 内部工程规划文档，非业务受众文案，不对甲方交付原文。
created_at: 2026-11-12
sources:
  - docs/plan/PLAN_02_技术选型.md（v001，选型冻结）
  - docs/plan/PLAN_01_范围基线与验收标准.md（v001，S/I/D 清单）
  - 记录/REQUIREMENTS_EXTRACT_20261111.txt（甲方需求原文提取）
  - 记录/SOURCE_proposal_v004_20261111.md（对外方案 v004）
```

## 1. 架构总览

单体内网部署：一个 Node 进程承载「静态站点 + HTTP API + WebSocket 网关 + SQLite」，三端浏览器接入：

```text
┌──────────────┐   HTTP(S)/WS   ┌───────────────────────────────┐
│ 大屏浏览器     │◄─────────────►│  Node 服务（内网）              │
│ 完整精度场景   │               │  ├ 静态站点托管（大屏/平板/后台）│
├──────────────┤               │  ├ REST API（数据读写/导入）    │
│ 平板浏览器     │◄─────────────►│  ├ WebSocket 网关（指令/状态）  │
│ 简易模型+预设视角│              │  ├ SQLite（设备/库位/看板配置）  │
├──────────────┤               │  └ 日志（文件，滚动）           │
│ 后台浏览器     │◄─────────────►│                               │
│ 导入/表单/配置  │               └───────────────────────────────┘
└──────────────┘
```

- 无真实设备接口（一期）；二期新增「数据接入模块」即可，不改渲染层。
- 无 GPU 渲染服务器：渲染全部在浏览器端 WebGL 完成。

## 2. 模块划分

| 模块 | 职责 | 关键点 |
|---|---|---|
| `web-3d`（大屏端） | 场景加载、LOD/实例化、聚焦/剖切、设备状态渲染、物流动态 | Three.js；静态场景与动态设备分离（R01 机制） |
| `web-ctrl`（平板端，可裁） | 简易模型 + 预设固定视角 + 指令发送 | 与大屏共用场景数据，用低精度 LOD 档 |
| `web-admin`（后台） | 文件导入、表单维护、看板与状态配置 | 单管理员登录会话 |
| `server` | Express：静态托管 + REST；`ws`：指令/状态广播；SQLite 数据层；导入解析 | 单进程；启动脚本 + 日志滚动 |
| `shared`（共享协议） | 指令协议、状态模型、DTO 的 TypeScript 类型 | 三端共用，避免协议漂移 |

## 3. 状态模型（核心实体草案，字段级在 C4 冻结）

- `device`：编码、名称、类别（77 类之一）、所在层/线/位置、状态、状态配色策略、标签；
- `line`：编号、楼层、设备列表、线级或设备级标记；
- `floor`：楼层、可见性、剖切/透明配置；
- `area`：厂区/仓库/道路等区域分组；
- `storage_location`：库位编号、区域、状态（占用/空闲/预留/异常）；
- `logistics_path`：路径 id、设备类型、路点序列、速度参数、循环标记；
- `dashboard_config`：看板模块启用、数据源、展示参数；
- `user`：单管理员账号（内置初始化 + 可改密）。

## 4. 指令与消息协议（v1 草案）

消息信封（JSON）：

```json
{ "type": "<消息类型>", "src": "<端标识: screen|pad|admin|server>", "ts": 1710000000000, "seq": 1, "payload": {} }
```

| type | 方向 | payload 要点 |
|---|---|---|
| `viewport.set` | pad → server → screen | 预设视角 id / 楼层 / 聚焦目标（line 或 device） |
| `floor.switch` | pad/admin → screen | floor id、显示模式（显示/隐藏/剖切/透明） |
| `device.select` | pad/screen → server | device id；server 广播选中态 |
| `device.status` | admin → all | device id + status；大屏刷新配色/标签 |
| `logistics.run` | admin → screen | path id + 启停/速度 |
| `dashboard.sync` | admin → screen | 看板配置更新后全量同步 |
| `ack` | server → src | 指令回执（seq 对应）；失败带 error 码 |
| `heartbeat` | 双向 | 保活；超时触发重连 |

- 一致性语义：指令由 server 转发并广播，大屏执行后回 `ack`；平板「发指令、大屏执行同样操作」即「同一指令广播到两端，两端各自按同一语义执行」。
- 字段级契约在 C3 实现时冻结，本文件升 v002 回填。

## 5. REST API 草案（C4 冻结）

| 方法/路径 | 用途 |
|---|---|
| `POST /api/auth/login` / `POST /api/auth/logout` | 后台登录会话 |
| `GET /api/devices`、`PUT /api/devices/:id` | 设备列表/状态维护 |
| `POST /api/import` | 上传 CSV/JSON，解析入库（dry-run 返回校验报告） |
| `GET/PUT /api/lines`、`/api/floors`、`/api/storage-locations` | 场景结构维护 |
| `GET/PUT /api/dashboard` | 看板配置 |
| `GET /api/health` | 健康检查（部署验收用） |

## 6. 部署拓扑（两种形态，待甲方服务器信息确认）

- **形态 A（演示首选）**：Node 服务直接跑在大屏主机；大屏浏览器访问本机，平板/后台走内网 IP。零额外服务器依赖，现场演示最稳。
- **形态 B（正式部署）**：Node 服务在甲方内网服务器；大屏/平板/后台全部浏览器访问同一地址。部署文档按此形态写，形态 A 作为子集。
- 运行方式：Windows/Linux 均可（Node 跨平台）；启动脚本 + 开机自启说明 + 日志滚动 + SQLite 文件备份说明，写入部署文档（报价第 12 项）。

## 7. 仓库目录结构草案（C1 启动时落地）

```text
apps/
  web-3d/     大屏三维前端（Vite+TS+Three.js）
  web-ctrl/   平板控制端（同一代码库、低精度模式或独立入口）
  web-admin/  数据后台
server/       Express + ws + SQLite
packages/
  shared/     协议与状态模型类型
assets/       glTF 资产与导出脚本（大文件按需 git-lfs 或排除评估）
验证/         C1–C4 回执与脚本
docs/         规划/调研/经验（DEV_EXPERIENCE.md 随开发启动）
```

## 8. 非功能与边界

- 性能：大屏 1920×1080 ≥30 FPS、状态刷新 1s 级、指令往返 <200ms（局域网），细则在 PLAN_05。
- 安全：仅内网访问；后台登录会话；不暴露公网、不存第三方账号；SQLite 文件与日志不入发布包。
- 可运维：重启即恢复（进程守护说明）、数据备份 = 复制 SQLite 文件、日志按天滚动。
- 扩展：二期真实接口 = server 增加采集模块写入同一状态模型，前端零改动（设计目标，不承诺）。

## 9. 开放问题

1. 甲方内网服务器 OS 与配置（Windows/Linux、CPU/内存/磁盘）。
2. 部署机是否允许联网（影响 npm 依赖安装方式；离线则打包 node_modules 随发布走）。
3. 端口占用与网络策略（默认 8080，可配置）。
4. 平板端是否保留（影响 `web-ctrl` 是否实现）。
