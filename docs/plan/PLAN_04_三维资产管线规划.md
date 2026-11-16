# PLAN_04｜三维资产管线规划

```yaml
artifact_id: TECH-TWIN-001-PLAN-04
current_version: v001
record_type: Plan / 三维资产管线
status: Active（v001 基线；77 类完整清单与工业模型策略待甲方资料后冻结）
workspace_id: WS-ORG-001
data_class: internal
humanizer_status: ExceptionReviewed
exception_reason: 内部工程规划文档，非业务受众文案，不对甲方交付原文。
created_at: 2026-11-12
sources:
  - docs/plan/PLAN_01_范围基线与验收标准.md（S4 设备级建模口径）
  - docs/plan/PLAN_02_技术选型.md（glTF/Draco 选型）
  - docs/research/RESEARCH_01_Web三维引擎对比调研.md（实例化/LOD 机制）
  - docs/research/RESEARCH_02_glTF压缩与轻量化调研.md（压缩方案）
  - 记录/REQUIREMENTS_EXTRACT_20261111.txt（需求 1–4，77 类与 400 台口径）
  - 机构 Blender 工具链：`工具配置/Blender/README.md`（TOOL-ORG-BLENDER-001，5.2.3 LTS 自编译，blender.ps1 调用层）
```

## 1. 目标与口径

- 交付：**77 类设备轻量化简模** + 厂区/厂房/仓库静态场景 + 物流设备基础动画部件；一期场景内设备单体 **≤200**（内部口径，S4）。
- 展示模型**全部新建**：甲方只有工业模型，即使可导出也需重拓扑/减面才能进 Web（工业 CAD 精度模型面数级远超出 Web 承受）。
- 所有资产统一：glTF 2.0、米制单位、右手坐标系（glTF 约定）、原点/朝向规范。

## 2. 资产分类与 LOD 分级

| 资产层 | 内容 | 面数预算（三角面，基准） | LOD 档 |
|---|---|---|---|
| L0 静态场景 | 建筑外观/道路/围墙/绿化/厂房骨架 | 全厂远景合并后 ≤200k | 远档实例化；近景局部高模 |
| L1 设备级（77 类） | 产线设备/物流设备/立体仓库等 | 每类 2k–20k（按复杂度分档） | 远：≤1k 简模；中：3k–8k；近：全量 |
| L2 物流可动件 | AGV/OHT/堆垛机/输送线/穿梭车 | 随 L1，可动部件独立对象 | 同 L1；运动件不做 LOD 切换 |
| L3 示意体块 | 线级体块、物料暂存、装饰 | ≤1k/个 | 远中档实例化 |

- LOD 切换：距离触发（远/中/近三档），大屏 1920×1080 下以「帧率稳定优先」为准（PLAN_05 校验）。
- 实例化：同类别/同状态设备一律 `InstancedMesh`；不同线直接复制模型 → 实例引用，不是几何复制。

## 3. 77 类设备建模工作流

### 3.1 编码规则草案（与 PLAN_01 S6 设备资产编码衔接）

- 类别码（按需求文档已知类别，完整 77 类待甲方清单）：
  `CAR-01..38`（车载线 38 台/线）、`ITM-01..30`（IT 模组线 30 台/线）、`AGV-01..03`、`SST-01`、`OHT-01..02`（轨道+车）、`WH-01`（立体仓库）、`SHUTTLE-01`、`OHCV-01`；
- 设备编码 = `类别码-序号`（如 `AGV-01-007`），场景内唯一，后台可维护（PLAN_03 状态模型 device）。

### 3.2 单类设备流程（每类走同一流水线）

```text
参考（图纸/照片/参考视频截图）
  → 建模：box-modeling 或参数化脚本（Blender bpy；可复用机构 sculpt_base/chibi 类脚本思路）
  → 检查：面数报告 / 非流形 / 法线一致 / 单位与原点 / 命名
  → UV + 材质：PBR 简材或程序化材质（一期统一材质库，同材质共享）
  → 可动部件分离（轮/货叉/轨道车体）
  → 导出 glTF 2.0 → Draco/meshopt 压缩 → 入库 + ASSET_MANIFEST 登记
```

- 工具：机构自编译 **Blender 5.2.3 LTS**（`TOOL-ORG-BLENDER-001`，`${TOOLS_ROOT}/blender`，Cycles GPU 可用），批处理一律走 `工具配置/Blender/blender.ps1 run <脚本> -Args "..."`（`--python-exit-code 1` 防静默失败，机构经验）。
- 检查脚本化：面数/岛数/非流形/比例/朝向批量报告（借鉴 `scene_report.py` 与 sculpt_base 的 BASE_JSON 输出思路）。
- 材质策略：一期统一 8–16 个共享材质（金属/塑料/玻璃/地面/货架色带等），禁止每资产独立贴图；需要贴图的按 512/1K 图集。

## 4. 导出与压缩

| 环节 | 工具 | 说明 |
|---|---|---|
| 导出 | Blender 内建 glTF 导出器 | 勾选压缩选项视版本；导出后校验回读 |
| 几何压缩 | Draco（`gltf-transform` CLI 或 gltfpack） | 压缩率约 10:1；加载 WASM 解码（R02） |
| 备选 | gltfpack / meshopt | 体积更小、解码更轻，C2 实测后二选一冻结 |
| 纹理 | 按需 KTX2；一期简模以程序化材质为主 | 纹理量大再引入（R02） |
| 校验 | `blender.ps1 report` + 三端回读（浏览器加载 smoke） | 入库前必过 |

## 5. 入库与管理

```text
assets/
  devices/<类别码>/<资产id>.glb
  scene/<厂区|厂房|仓库>/*.glb
  materials/（共享材质定义）
  ASSET_MANIFEST.json   ← 唯一资产台账
```

- `ASSET_MANIFEST.json` 字段：`asset_id / category / name / faces / size / sha256 / lod_levels / movable_parts / status / updated_at`。
- 资产台账与设备编码一一对应；导入后台（D1）与三维场景共用同一编码，不出现两套命名。
- 仓库策略：一期资产预估 <50 MB，直接 Git 跟踪；总量超限再评估 git-lfs（决策留档）。
- 版本：资产改版走提交历史 + 台账版本号；对外交付物（GLB 集）在交付阶段另出清单。

## 6. 与渲染层衔接

- 场景装配由「加载清单」驱动：`scene.manifest.json`（哪些资产、实例数、LOD 档位、状态绑定），与 `ASSET_MANIFEST` 分开（前者是场景实例数据，后者是资产本体台账）。
- 状态驱动只改「实例颜色/材质参数/标签可见性」，不换几何（R01 状态驱动机制）。
- 物流设备动画：部件级动画（轮转/货叉伸缩/轨道移动）在资产内定义，运行时按 `logistics_path` 参数驱动。

## 7. 工作量与批次（对应报价第 2 项 ¥13,000）

- 批次 1（演示必需，先做）：AGV×3、OHT×2、堆垛机、输送线、穿梭车、立体仓库、货架+库位、车载线/IT 线代表设备各 2–3 类 —— 约 15–18 类；
- 批次 2：车载线 + IT 模组线其余设备（38+30 类中的大部分）；
- 批次 3：全厂场景、厂房骨架、道路绿化、示意体块；
- 批次 4（二期）：400 台全量与剩余线体精模。
- 排期细化在 PLAN_06（建模约占 10 周中的 3–4 周，按批次穿插验证）。

## 8. 风险与开放问题

1. **工业模型格式未知**（STEP/IGES/JT/3DXML?）：若可导出 → FreeCAD 或 `fascat`（STEP/IGES/BREP/JT → glTF/USD，开源）中转 → Blender 重拓扑；若不可导出 → 按图纸/参考图估形（与甲方书面说明差距）。
2. **77 类完整清单未给**：先按需求文档已知类别推进批次 1；完整清单到位后冻结编码规则。
3. 图纸版本：以甲方书面确认版本为准（PLAN_01 §9-5）。
4. 面数 vs 帧率：LOD 与实例化后仍不达标 → 降批次 2/3 面数预算（PLAN_05 联动）。
5. 资产知识产权：定制成果归属按报价说明 4.2-3；我方通用组件与工具链保留（不写入资产仓库的第三方模型）。

## 9. 参考来源（留痕）

| # | 来源 | 类型 | 日期 | 要点 | 适用性 |
|---|---|---|---|---|---|
| 1 | [fascat（PyPI）](https://pypi.org/project/fascat/0.6.0/) | 开源库 | 检索命中 2026-11-12（未逐页抓取） | STEP/IGES/BREP/JT → glTF/USD/OBJ 转换 | 高：工业模型中转备选 |
| 2 | [STEP → glTF（bitbybit 教程，FreeCAD → Blender → Khronos Compressor）](https://github.com/bitbybit-dev/bitbybit/blob/2ab11c81d17f1b9a7c4a7304d6d4c22b137fa87f/docs/learn/3d-bits/tutorials/videos-tutorials/step-to-gltf.md) | 开源教程 | 同上 | CAD 转 Web 资产的完整链路 | 高：甲方工业模型处理参考 |
| 3 | [Blender-Developer-Tools：ai-asset-pipeline-template（TMHSDigital）](https://github.com/TMHSDigital/Blender-Developer-Tools/tree/main/templates/ai-asset-pipeline-template) | 开源模板 | 同上 | AI 资产管线模板 | 中：流程结构参考 |
| 4 | [glTF 2.0 规范（Khronos）](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html) | 标准 | R02 已登记 | 资产格式标准 | 高（复用 R02 #1） |
| 5 | [Draco 3D（google/draco）](https://github.com/google/draco) | 官方仓库 | R02 已登记 | 几何压缩 | 高（复用 R02 #2） |
| 6 | 机构 Blender 工具链 `工具配置/Blender/README.md`（`TOOL-ORG-BLENDER-001`） | 机构台账 | 2026-11-12 读取 | 自编译 Blender 5.2.3 + blender.ps1 调用层 + 随附脚本 | 高：建模执行环境 |

> 注：网络来源仅检索命中、未逐页抓取；正文为本机构技术判断。
