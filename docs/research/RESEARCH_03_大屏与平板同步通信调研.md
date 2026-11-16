# RESEARCH_03｜大屏与平板同步通信调研

```yaml
artifact_id: TECH-TWIN-001-RESEARCH-03
current_version: v001
record_type: Research / 调研参考
status: Active（v001 基线；通信方案在最小验证用例 C3 实测后冻结）
workspace_id: WS-ORG-001
data_class: internal
humanizer_status: ExceptionReviewed
exception_reason: 内部技术调研文档，非业务受众文案。
created_at: 2026-11-12
```

## 1. 结论摘要

- 大屏 ↔ 平板指令同步选 **WebSocket**：双向、低延迟、浏览器原生（MDN 标准 API），内网单服务器即可承载；服务端用 Node.js `ws`（轻量、与后端同栈）。
- 数据面板刷新：状态类数据 1s 级轮询（HTTP）或同一条 WebSocket 推送，两种都保留，性能口径在 `PLAN_05` 冻结。
- **MQTT 本期不引入**：它适合海量设备连接、弱网、发布/订阅的设备端协议；本期是「几个浏览器端 + 一台内网服务器」的局域网场景，WebSocket 更简单直接（对比论文结论亦表明局域网内 WebSocket 延迟优于或相当 MQTT，见来源 4）。
- 消息协议：JSON 指令集（视角切换 / 楼层切换 / 聚焦 / 状态变更 / 心跳），大屏与平板同协议、同场景语义，保证「平板发指令、大屏执行同样操作」的需求语义。

## 2. 要点摘录

- 需求 4.2 语义：平板为简易模型 + 预设固定视角，**发操作指令**；大屏在同样场景下执行同样操作，保持全厂视野与渲染精度——同步的是「指令」不是「画面」，这是选 WebSocket 的关键（小指令包、双向确认）。
- 已公开的同类实现（专利公开文本 CN116368528A）：WebSocket 服务端接收控制端信息，向数据展示面板层与三维模型展示层分发，使两者同步调整——与本期架构一致，可作为落地形态佐证。
- 行业观点（鸣启解码第 30 期）：数字孪生大屏需要移动端联动才有演示价值，联动方式为指令级同步而非镜像画面。

## 3. 参考来源（留痕）

| # | 来源 | 类型 | 日期 | 要点 | 适用性 |
|---|---|---|---|---|---|
| 1 | [WebSocket API（MDN）](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket) | 官方文档 | 2026-11-12 记录（canonical URL，未抓取） | 浏览器 WebSocket 标准 API | 高：客户端实现依据 |
| 2 | [ws（npm，Node.js WebSocket 实现）](https://www.npmjs.com/package/ws) | 官方包 | 同上 | Node 服务端 WebSocket 库 | 高：服务端实现依据 |
| 3 | [CN116368528A 专利公开文本](http://patentimages.storage.googleapis.com/d0/fe/9e/11e9ddf417ffd2/CN116368528A.pdf) | 公开专利 | 检索命中 2026-11-12（未逐页抓取） | WebSocket 分发控制信息、面板与三维同步 | 中：同类架构佐证 |
| 4 | [Comparative Performance Analysis Between the MQTT and WebSocket Protocols（jurnal.kdi.or.id）](https://jurnal.kdi.or.id/index.php/bt/article/download/3223/1701) | 对比论文 | 检索命中 2026-11-12（未逐页抓取） | MQTT 与 WebSocket 延迟/吞吐对比 | 高：协议选型依据 |
| 5 | [鸣启解码 第30期：别让你的数字孪生大屏沦为高级壁纸！移动端联动才是正确打开方式（微信公众号）](http://mp.weixin.qq.com/s?__biz=Mzg2ODQ5NDY1OA==&mid=2247487788&idx=1&sn=371b6d4818fd5400321eb84d13a854ec) | 行业文章 | 检索命中 2026-11-12（未逐页抓取） | 大屏 + 移动端联动的演示价值与方法 | 中：需求 4.2 演示形态参考 |

> 注：本轮网络环境仅检索命中，未逐页抓取；来源按检索命中如实登记，正文要点为本机构技术判断。

## 4. 后续动作

- C3 用例实测：Node `ws` 服务器 + 两个浏览器页（模拟大屏/平板），验证指令往返延迟、断线重连、多端一致；数据回填本文件 v002。
- 若平板端被甲方裁剪：C3 仍做（架构含指令同步层，二期与远程维护都用得上），但验收清单 I6 移除。
- 浏览器兼容口径（大屏/平板统一）：Chrome / Edge 稳定版 ≥ 120（WebGL2 + WebSocket 全支持），部署时在目标机实测固定版本。
