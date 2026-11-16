# RESEARCH_02｜glTF 压缩与模型轻量化调研

```yaml
artifact_id: TECH-TWIN-001-RESEARCH-02
current_version: v001
record_type: Research / 调研参考
status: Active（v001 基线；压缩方案在最小验证用例 C2 实测后冻结）
workspace_id: WS-ORG-001
data_class: internal
humanizer_status: ExceptionReviewed
exception_reason: 内部技术调研文档，非业务受众文案。
created_at: 2026-11-12
```

## 1. 结论摘要

- 资产格式统一用 **glTF 2.0**（Khronos 标准，Three.js `GLTFLoader` 官方支持，Blender 直接导出——机构已有 Blender 工具链 `TOOL-ORG-BLENDER-001`）。
- 几何压缩首选 **Draco**（压缩率高、Three.js 内置解码器，浏览器 WASM 解码），同时评估 **meshopt/gltfpack**（顶点缓存/量化优化 + 压缩，产物体积更小、解码更轻）。
- 纹理量大时再上 **KTX2/Basis**；一期简模纹理量小，可先不引入（降低链路复杂度）。
- 最终组合由最小验证用例 **C2（资产链路）** 实测数据冻结：以「单设备 glTF 体积、加载时长、内存、视觉损失」为准。

## 2. 要点摘录

- **Draco**：几何压缩可达约 10:1 级（点/UV/法线量化 + 熵编码），加载需 WASM 解码器，Three.js `DRACOLoader` 开箱即用；解码开销小，适合网络受限的内网/局域网场景。
- **meshopt / gltfpack**：以顶点缓存优化、索引/属性量化为核心，配合压缩（EXT_meshopt_compression），体积极小且解码轻量；Three.js 需挂 `meshopt_decoder`（体积小，几十 KB）。
- 参考试验：dev.to 的 132 次加载试验对比 Draco / Meshopt / KTX2 的加载与解码差异（见来源表），结论层面两者均满足 Web 场景，选择取决于「压缩率 vs 解码开销 vs 工具链」权衡。
- 常规加载优化五步（腾讯云文章，R01 已登记 #6）：模型面数控制 → 纹理压缩与图集 → Draco/meshopt 压缩 → 懒加载/分块 → CDN/缓存策略。

## 3. 参考来源（留痕）

| # | 来源 | 类型 | 日期 | 要点 | 适用性 |
|---|---|---|---|---|---|
| 1 | [glTF 2.0 规范（Khronos Registry）](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html) | 标准规范 | 2026-11-12 记录（canonical URL，未抓取） | 资产格式标准 | 高：统一资产格式依据 |
| 2 | [Draco 3D（google/draco，GitHub）](https://github.com/google/draco) | 官方仓库 | 同上 | 几何压缩库与编解码器 | 高：Three.js 官方集成 |
| 3 | [meshoptimizer（zeux/meshoptimizer，GitHub）](https://github.com/zeux/meshoptimizer) | 官方仓库 | 同上 | 网格优化与 EXT_meshopt_compression | 高：gltfpack 上游 |
| 4 | [Draco vs Meshopt and KTX2: 132 Loading Trials in Three.js（dev.to）](https://dev.to/alicia_lu/draco-vs-meshopt-and-ktx2-132-loading-trials-in-threejs-lhm) | 实测对比 | 检索命中 2026-11-12（未逐页抓取） | 132 次加载试验对比三种压缩 | 高：C2 判定口径参考 |
| 5 | [3D 模型加载太慢？5 步解决 Web3D 性能瓶颈（腾讯云开发者社区）](https://cloud.tencent.com.cn/developer/article/2688572) | 中文教程 | R01 已登记（检索命中 2026-11-12） | 加载/压缩五步法 | 高（复用 R01 #6） |
| 6 | [WebGL开发数字孪生项目（阿里云开发者社区）](https://developer.aliyun.com/article/1709504) | 案例/教程 | R01 已登记（检索命中 2026-11-12） | 数字孪生资产与性能实践 | 中（复用 R01 #1） |

> 注：本轮网络环境仅检索命中，未逐页抓取；来源按检索命中如实登记，正文要点为本机构技术判断。

## 4. 后续动作

- 在 C2 用例中实测：同一测试设备分别走「原始 glTF / Draco / gltfpack」三路，记录体积、加载时长、内存与目视损失，数据回填本文件 v002 并据此冻结压缩组合。
- 若甲方后续提供工业模型：先评估导出可行性（STEP/IGES 等 → 中转 Blender → glTF），管线细节落 `PLAN_04_三维资产管线规划.md`。
