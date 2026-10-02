# Phase 2 — Architecture & Data Flow

> 执行前置：重读 `audit/01-discovery.md` 及 `audit/` 中其它已有产物。
> 输出：`audit/02-architecture.md`

建立：

- Project Structure
- Architecture Map
- Module Map
- Entry Point Map
- Request Flow
- Data Flow
- Trust Boundary
- External Dependency Map
- State Transition Map

对关键流程建立：

Input
→ Validation
→ Transformation
→ Processing
→ Storage / Network / Sink
→ Output

如果是 Web/API 项目，优先追踪：

Browser/Client
→ Route
→ Handler
→ Service
→ Database/External API
→ Response

如果是 CLI：

CLI Input
→ Parser
→ Validation
→ Business Logic
→ File/System/Network
→ Output

如果是 Worker/serverless：

Request/Event
→ Worker
→ Validation
→ External Resource
→ Response/Queue/Storage

不要假设项目一定属于上述类型，应根据实际代码调整。

## 系统模型（必须输出，是后续所有阶段的共同参照）

在产物末尾用如下 YAML 固定输出一份 Canonical System Model。后续各阶段、Phase 9B 漏报对账都以它为准，保证多代理/多透镜之间理解一致：

```yaml
entry_points:        # 每个外部可达入口：method + path / CLI 命令 / 事件
  - "POST /api/xxx"
trust_boundaries:    # client→api、api→db、worker→external…
  - "client -> api"
sensitive_sinks:     # database / filesystem / outbound_http / shell / html_dom / redirect / log
  - "outbound_http"
identities:          # anonymous / user / admin / service
  - "anonymous"
state_machines: []   # 有状态工作流才填：states 与 transitions（含非法/缺失/未授权转移）
```

有状态工作流（支付、审核、权限、恢复删除等）必须在 state_machines 中列出状态与转移，供 Phase 5 检查非法转移、缺失转移、重放与竞态。

输出内容：架构图（文字版）、模块地图、关键流程的端到端数据流、信任边界清单、**Canonical System Model（YAML）**。
