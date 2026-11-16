# RESEARCH_01｜Web 三维引擎对比调研（Three.js vs Babylon.js）

```yaml
artifact_id: TECH-TWIN-001-RESEARCH-01
current_version: v001
record_type: Research / 调研参考
status: Active（v001 基线；正式选型决策落 PLAN_02）
workspace_id: WS-ORG-001
data_class: internal
humanizer_status: ExceptionReviewed
exception_reason: 内部技术调研文档，非业务受众文案。
created_at: 2026-11-12
```

## 1. 结论摘要

- **首选 Three.js**：生态与中文资料最广、数字孪生/大屏案例最多；团队已有工程经验（机构主站 hero 即 three.js），并有成套 Three.js 开发 Skill（`threejs-skill-router` 等，机构技能库）；渲染自由度大，适合按本期范围精确定制。
- **Babylon.js 为备选**：引擎级功能更完整（内置场景图、Thin Instances、Scene Optimizer、WebGPU 支持更早），适合超大型工业场景；但团队经验与技能库覆盖少。
- 本期场景（77 类设备模型、一期 ≤200 设备单体、LOD + 实例化）两个引擎都能满足。**正式选型在 PLAN_02 冻结**，并安排最小验证用例（同场景双引擎对比 demo）。

## 2. 对比表

| 维度 | Three.js | Babylon.js |
|---|---|---|
| 定位 | 轻量渲染库，自由度大、可深度定制 | 完整引擎，内置功能多、约定强 |
| 实例化 | `InstancedMesh`（官方 API） | `Instances` / `Thin Instances`（官方，API 更省内存、性能更好） |
| LOD | `THREE.LOD`（官方） | 内置 LOD（`MeshLODLevel`） |
| Draw Call 优化 | 手工合并/实例化，社区指南多 | `Scene Optimizer` + 官方优化向导 |
| WebGPU | `WebGPURenderer` 逐步成熟 | 支持更早、文档较完整 |
| glTF | `GLTFLoader` 官方 | glTF Loader 官方，Draco/KTX2 集成好 |
| 生态/社区 | 最大，教程/案例海量 | 活跃，但案例量少于 Three.js |
| 中文资料 | 多（含大量大屏/数字孪生教程） | 中 |
| 团队熟悉度 | 高（ORG-WEB-001 hero + 机构 Three.js Skills） | 低 |
| 大屏/数字孪生案例 | 多（阿里云社区系列、腾讯云实践等） | 有，工业类相对少 |

## 3. 关键性能机制（选型无关，本期必做项）

- **实例化渲染**：同类设备/货架用 `InstancedMesh` / `Thin Instances`，把 ≤200 台设备压到几十个 draw call。
- **LOD 分级**：远景全厂鸟瞰低模、近景高模（需求 1.3 明确要求）。
- **合并与共享**：同材质网格合并、贴图图集、减少材质切换。
- **glTF + Draco/meshopt 压缩**：轻量化简模的传输与内存占用。
- **静态/动态分离**：静态场景烘焙合并；动态设备只改实例变换与材质状态，不重建几何。
- **状态驱动**：设备状态变化只切颜色/标签/可见性，保持帧率稳定。

## 4. 参考来源（留痕）

| # | 来源 | 类型 | 日期 | 要点 | 适用性 |
|---|---|---|---|---|---|
| 1 | [WebGL开发数字孪生项目（阿里云开发者社区）](https://developer.aliyun.com/article/1709504) | 案例/教程 | 检索命中 2026-11-12（未逐页抓取） | WebGL 数字孪生项目开发流程 | 高：大屏 + 数字孪生落地参考 |
| 2 | [WebGL数字孪生项目的全栈开发（阿里云开发者社区）](https://developer.aliyun.com/article/1741250) | 案例/教程 | 同上 | 全栈开发视角（前后端/数据） | 高：数据后台与接入参考 |
| 3 | [Thin Instances（Babylon.js 官方文档）](https://doc.babylonjs.com/features/featuresDeepDive/mesh/copies/thinInstances) | 官方文档 | 同上 | 薄实例化，省内存、高性能 | 中：选 Babylon 时的必用 API |
| 4 | [Thin instances visibility and optimization（Babylon.js 论坛）](https://forum.babylonjs.com/t/thin-instances-visibility-and-optimization/61237/2) | 社区问答 | 同上 | 实例可见性与优化讨论 | 中 |
| 5 | [Three.js 大场景渲染优化：从 Draw Call 合并到 LOD 分级（华为云社区）](https://bbs.huaweicloud.com/blogs/509054) | 中文教程 | 同上 | Draw Call 合并 + LOD | 高：本期性能优化直接参考 |
| 6 | [3D 模型加载太慢？5 步解决 Web3D 性能瓶颈（腾讯云开发者社区）](https://cloud.tencent.com.cn/developer/article/2688572) | 中文教程 | 同上 | 模型加载/压缩五步 | 高：glTF 压缩流程 |
| 7 | [Three.js企业级可视化实战（腾讯云开发者社区）](https://cloud.tencent.cn/developer/article/2733088) | 中文实战 | 同上 | 企业级可视化架构设计 | 高：大屏架构参考 |
| 8 | [threejs-too-many-meshes-merge-or-instance（paprika3d 指南）](https://github.com/Keeno-Corp/paprika3d/blob/594f0094b239f7729fa4fb536967589416ca0a4a/docs/guides/threejs-too-many-meshes-merge-or-instance.md) | 开源指南 | 同上 | 网格过多时「合并 vs 实例化」决策 | 高 |
| 9 | [Instanced Mesh 研究笔记（suryashch/3d_modelling）](https://github.com/suryashch/3d_modelling/blob/f391991a03cec7608eba126ba6ee2861a7faa444/research/optimizing-the-scene/instanced-mesh.md) | 开源笔记 | 同上 | InstancedMesh 场景优化实践 | 中 |

> 注：本轮网络环境只能做检索命中，无法逐页抓取验证；上表按检索命中如实登记，正文要点为本机构技术判断，未复制来源原文。

## 5. 后续动作

- PLAN_02 技术选型冻结（含最小验证用例计划：同场景双引擎对比）。
- 调研补强：glTF/Draco 压缩实测、大屏浏览器兼容口径（Chrome/Edge 版本）、WebSocket 大屏-平板同步案例。
