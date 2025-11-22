# Prometheus Operator 学习路径

## 1. 前置学习 (Prerequisites)
- **Kubernetes 进阶**：深入理解 Controller / Scheduler 工作机制，熟悉 CRD、RBAC、Admission Webhook。
- **Golang**：掌握 interface、struct 组合、context、goroutine/channel 等模式，能阅读 `client-go` 示例。
- **监控体系**：了解 Prometheus、Alertmanager、Thanos Ruler 的角色，以及 ServiceMonitor/PodMonitor 的使用场景。

## 2. 项目简介
Prometheus Operator 通过 Kubernetes 原生的 CustomResource 自动部署与管理 Prometheus、Alertmanager 及相关组件。它抽象出 `Prometheus`、`Alertmanager`、`ServiceMonitor`、`PodMonitor`、`ScrapeConfig`、`PrometheusRule` 等 CRD，让 SRE 不用手写繁杂的 scrape 配置即可完成大规模集群监控。

## 3. 学习目标
- **Operator 模式**：理解 Reconcile 循环如何根据 CRD 描述生成 StatefulSet、ConfigMap、Service 等资源。
- **多租户监控治理**：掌握 ServiceMonitor/PodMonitor/Probe 的差异，学会以标签选择器划分监控边界。
- **可扩展性**：学会自定义 Admission Webhook、准入校验、以及如何向 CRD 增加字段并保持向后兼容。

## 4. 探索路径 (使用 Cursor)
1. **CRD 结构**
   - 打开 `pkg/apis/monitoring/v1/types.go`。
   - `Cmd+L` 询问：“解释 `ServiceMonitorSpec` 的关键字段，如 `endpoints`, `namespaceSelector`, `selector` 在底层生成 scrape config 时分别起什么作用？”
2. **Reconcile 流程**
   - 聚焦 `pkg/prometheus/operator.go`、`pkg/alertmanager/operator.go`。
   - 让 Cursor 以简洁 Mermaid 时序图描述：当用户创建 `ServiceMonitor` → Operator 监听 → 生成 scrape 配置 → 更新 Prometheus StatefulSet 的全过程（保持简洁，符合您偏好）。
3. **动态 Admission**
   - 阅读 `pkg/admission`。
   - 询问：“`PrometheusRule` 校验 webhook 如何阻止非法的 recording/alerting rule？底层语法检查依赖哪些开源库？”
4. **ServiceMonitor vs PodMonitor**
   - 对比 `Documentation/user-guides/getting-started.md` 中的示例。
   - 让 Cursor 输出“何时选 ServiceMonitor，何时选 PodMonitor，背后生成的 ServiceDiscovery 配置有什么不同？”
5. **高可用场景**
   - 检查 `Documentation/user-guides/high-availability.md`。
   - 设计任务：“如何让同一个 `Prometheus` CRD 分布在 IDC 与公有云两个节点池，并通过 `topologySpreadConstraints`/`affinity` 进行约束？”

## 5. 实战演练
- **任务一：新增字段实验**  
  在 `PrometheusSpec` 中加入自定义字段（例如 Sidecar 注入配置），编译并运行 `operator` 观察生成资源的变化，体验 CRD schema 演进流程。
- **任务二：多环境同步**  
  利用 `kustomize`/`jsonnet` 描述不同环境的 ServiceMonitor，结合 GitOps 流程 (Argo CD / Flux) 做多集群同步。
- **任务三：可观测性增强**  
  阅读 `example/user-guides/getting-started/example-app`，尝试扩展 `PrometheusRule`，并让 Cursor 输出底层 PromQL 语法解释和 Alertmanager 路由机制。

## 6. 深入思考
- 如何结合 Karmada/Cluster API，将 Prometheus Operator 的 CRD 分发到多云集群，做到同构监控？
- Admission Webhook 会影响 API Server 吞吐，如何通过水平扩缩或缓存机制保证性能？
- 当 CRD 数量极大时（数万 ServiceMonitor），Reconcile 延迟会影响报警实时性，可以如何分片或做事件去抖？
# Prometheus Operator 学习路径

## 1. 前置学习 (Prerequisites)
*   **Kubernetes 基础**: 熟悉 Pod, Service, StatefulSet, ConfigMap, Secret 等核心资源。
*   **Golang**: 掌握 Interface, Struct, Goroutine Channel, Context。
*   **Kubernetes 扩展**: 了解什么是 CRD (Custom Resource Definition) 和 Controller Pattern。

## 2. 项目简介
Prometheus Operator 是 Kubernetes Operator 模式的经典代表作。它通过扩展 Kubernetes API (CRD) 来自动化管理 Prometheus 监控实例。

## 3. 学习目标 (SRE 核心)
- **掌握 Operator 模式**: 理解 Controller 如何通过 Reconcile 循环将系统状态收敛到期望状态。
- **Kubernetes API 编程**: 学习如何使用 `client-go` 与 K8s API Server 交互。
- **Go 高级并发**: 学习 Go 在处理大量并发事件（Informer 机制）时的最佳实践。

## 4. 探索步骤 (使用 Cursor)
1.  **CRD 定义**:
    - 搜索 `pkg/apis/monitoring` 目录。
    - **Cursor 交互**: "解释 `Prometheus` 这个 CRD 的结构体定义。它是如何映射到 K8s YAML 文件的？"
2.  **核心循环 (Reconcile)**:
    - 找到 `pkg/operator` 下的核心 Controller 逻辑。
    - **Cursor 交互**: "请生成一个 Mermaid 流程图，展示当用户创建一个 `ServiceMonitor` 对象时，Operator 内部发生了什么？" [[memory:6151402]]
3.  **K8s 资源生成**:
    - 查看 Operator 是如何动态生成 StatefulSet (Prometheus) 和 ConfigMap (配置) 的。
    - **重构思考**: 观察其错误处理和重试机制，对比您日常写的脚本，思考如何改进。

## 5. 进阶任务
- 尝试修改 Operator 代码，增加一个新的 CRD 字段（例如给 Prometheus Pod 注入一个 Sidecar），并在本地通过 `make` 编译验证。
