# MountSea 功能开发路线图

## 文档目的

本文把 [`architecture.md`](architecture.md) 中的目标架构拆解为可实施、可验收的功能清单。
顺序遵循以下原则：

1. 先固化领域语义、配置和可测试端口，再接入外部基础设施。
2. 先完成确定性的离线回测闭环，再建设 Web 控制面和在线运行面。
3. 先实现单账户、单 Node、Sandbox 闭环，再进行真实资金交易。
4. 每个阶段必须有可自动验证的交付物，未通过 Gate 不进入下一阶段。
5. `catalog_capture` 负责行情连接、采集和 Parquet Catalog 生命周期，MountSea 只负责控制面集成。

## 状态标记

- `[ ]` 未开始
- `[-]` 进行中
- `[x]` 已完成
- `Gate` 阶段进入下一阶段前必须通过的验收条件

当前仅完成 Workspace、模块边界和架构文档骨架，业务功能均视为未开始。

## 阶段 0：工程基线

目标：建立后续开发所需的质量、配置和决策基线，避免功能增长后再补工程约束。

### 0.1 仓库与 CI

- [ ] 增加 `rustfmt`、Clippy、单元测试和文档链接检查 CI。
- [ ] 固定 Rust 工具链，增加 `rust-toolchain.toml`。
- [ ] 配置依赖审计、许可证检查和供应链策略，例如 `cargo-deny`。
- [ ] 建立版本规则、Changelog 和发布构建约定。
- [ ] 为 Linux 生产目标增加 CI，Windows 只作为开发环境验证目标。

### 0.2 工程公共约定

- [ ] 统一错误模型、时间类型、ID、序列化格式和日志字段。
- [ ] 统一 tracing 初始化、敏感字段脱敏和关联 ID 传播规则。
- [ ] 建立测试目录规范：单元测试、集成测试、契约测试和故障注入测试。
- [ ] 建立 ADR 目录，记录 PostgreSQL、任务队列、IPC 和 NautilusTrader 版本策略。
- [ ] 决定 NautilusTrader 与 `catalog_capture` 的固定 Commit、依赖接入方式和升级流程。

**Gate 0：** CI 可在干净 Linux 环境完成格式、静态检查、测试、许可证和文档校验；外部依赖版本均可复现。

## 阶段 1：领域模型与配置内核

目标：在不依赖数据库、Web 或 NautilusTrader 的情况下固化平台业务语义。

### 1.1 `domain`

- [ ] 实现强类型 ID 和公共值对象：Organization、User、Account、Strategy、Dataset、Job、Deployment、Runtime、Node。
- [ ] 实现 Strategy、Dataset、Backtest、Deployment 和 Runtime 状态机。
- [ ] 实现 Approval、AuditRecord、ReconciliationCase 和 Alert 领域模型。
- [ ] 定义领域错误、状态迁移原因和不可变业务规则。
- [ ] 为合法与非法迁移增加表驱动测试和属性测试。

### 1.2 `config`

- [ ] 定义平台、数据库、Catalog、回测、Supervisor、Node 和风险配置结构。
- [ ] 实现 TOML 加载、环境覆盖、默认值、版本字段和严格未知字段检查。
- [ ] 实现跨字段校验，例如账户与 Venue、Node 上限、路径和风险阈值。
- [ ] 确保 Secret 仅保存引用，序列化和日志不会泄漏密钥。
- [ ] 增加开发、测试和生产配置样例及快照测试。

### 1.3 `strategy`

- [ ] 定义 StrategyDescriptor、StrategyVersionMetadata 和能力声明。
- [ ] 定义参数 Schema、配置校验错误和稳定的策略工厂接口。
- [ ] 实现策略工厂注册表、重复注册检测和版本解析。
- [ ] 定义数据需求、可交易 Instrument 类型和运行模式约束。
- [ ] 增加最小示例策略，用于后续回测和 Sandbox 验证。

**Gate 1：** 所有状态机、配置和策略注册规则均有自动测试；基础 crate 不依赖 I/O 框架或 NautilusTrader。

## 阶段 2：存储与数据集注册

目标：建立事务、审计和版本化数据集基础，提供后续用例的可靠持久化边界。

### 2.1 `storage`

- [ ] 选择 PostgreSQL 客户端和迁移工具并记录 ADR。
- [ ] 定义 Repository 和 Unit of Work 端口，避免 Application 依赖具体 SQL 类型。
- [ ] 实现 Strategy、Dataset、BacktestJob、Deployment、Approval、Runtime 和 Audit Repository。
- [ ] 实现乐观并发控制、事务边界、分页和幂等记录。
- [ ] 为迁移增加正向、回滚策略和真实 PostgreSQL 集成测试。

### 2.2 `catalog`

- [ ] 实现 Dataset、DatasetVersion、Manifest、Partition、Checksum 和 CommitMarker 模型。
- [ ] 实现已提交版本读取规则，拒绝未完成或校验失败版本。
- [ ] 实现时间范围、Schema、重复、缺口和 Checksum 验证接口。
- [ ] 实现 Catalog 路径解析和数据版本引用，不重复 Parquet 写入逻辑。
- [ ] 增加临时目录损坏、缺失分片和部分提交测试。

### 2.3 `capture` 与 `catalog_capture` 集成 Spike

- [ ] 比较子进程 CLI、库依赖和独立服务三种集成方式。
- [ ] 固化采集计划、运行状态、Manifest 和错误交换契约。
- [ ] 实现单个采集任务的启动、取消、超时和结果导入原型。
- [ ] 将外部提交结果登记为 DatasetVersion，并保存原始证据。
- [ ] 通过 ADR 选定正式集成方式，再完善 `mountsea-capture-worker`。

**Gate 2：** PostgreSQL 可从零迁移；DatasetVersion 只有在 Manifest 和校验通过后可见；采集集成能复现成功、失败、取消和部分写入场景。

## 阶段 3：可复现回测闭环

目标：交付第一个可用产品增量，从固定数据和策略版本生成可复现结果。

### 3.1 `runtime-adapter`

- [ ] 固定 NautilusTrader 版本并建立兼容性测试。
- [ ] 将 MountSea Instrument、时间、数值和配置映射到 NautilusTrader 类型。
- [ ] 将 `strategy` 工厂映射为 BacktestEngine 可加载策略。
- [ ] 明确转换错误、版本不兼容和不支持能力的错误语义。
- [ ] 增加最小引擎构建和事件回放集成测试。

### 3.2 `backtest`

- [ ] 实现 BacktestJob 校验和不可变运行快照。
- [ ] 加载已提交 DatasetVersion 和固定 StrategyVersion。
- [ ] 支持种子、时钟、手续费、滑点和初始资金的显式配置。
- [ ] 实现执行、进度、取消、超时和失败分类。
- [ ] 保存订单、成交、仓位、PnL、指标和复现元数据。
- [ ] 使用相同输入重复运行并验证结果一致。

### 3.3 `mountsea-backtest-worker` 与 `mountsea-cli`

- [ ] 实现任务领取、Lease、心跳、重试和孤儿任务回收。
- [ ] 实现 CLI 的数据集登记、策略登记、提交、查询和取消命令。
- [ ] 实现人类可读输出和稳定 JSON 输出。
- [ ] 增加一个端到端示例和可重复运行脚本。

**Gate 3：** 从 CLI 可完成“登记数据集 → 登记策略 → 提交回测 → 查询结果”；相同输入产生一致结果；失败、取消和进程崩溃可恢复并可审计。

## 阶段 4：Application 与 Web 控制面

目标：将已验证的回测能力开放为安全、幂等、可审计的平台 API。

### 4.1 `application`

- [ ] 定义用户、策略、数据集、回测、部署、审批和审计用例。
- [ ] 定义 Repository、任务队列、时钟、ID、授权和审计端口。
- [ ] 实现事务编排、权限检查、幂等键和审计写入。
- [ ] 保证 CLI 与 API 复用同一用例，不复制业务规则。
- [ ] 为每个用例增加成功、冲突、权限和幂等测试。

### 4.2 `mountsea-api`

- [ ] 建立 Axum 启动、健康检查、优雅关闭和依赖装配。
- [ ] 实现认证和角色、团队、账户级授权。
- [ ] 实现 Strategy、Dataset、Backtest、Deployment、Runtime 和 Audit REST 资源。
- [ ] 统一错误响应、请求 ID、分页、版本和 `Idempotency-Key`。
- [ ] 增加 OpenAPI、契约测试、限流和敏感数据脱敏。
- [ ] 提供回测进度的 SSE 或 WebSocket 查询能力。

### 4.3 Web 前端最小版本

- [ ] 实现登录、数据集、策略、回测提交和结果查看页面。
- [ ] 实现任务状态、错误详情和审计查询。
- [ ] 暂不实现 Live 一键启动和复杂可视化。

**Gate 4：** API 和 Web 可完成阶段 3 的全部回测闭环；所有副作用请求均幂等、鉴权并记录审计；数据库备份恢复演练通过。

## 阶段 5：Node 协议与 Sandbox 运行面

目标：在无真实资金风险的环境中验证进程隔离、生命周期和恢复语义。

### 5.1 `node-protocol`

- [ ] 定义版本化消息 Envelope、关联 ID、命令、事件和状态快照。
- [ ] 定义 Accepted、Succeeded、Failed、InProgress、Unknown 和 Rejected 语义。
- [ ] 实现兼容性规则、大小限制、认证和超时。
- [ ] 增加 Golden File、版本兼容和模糊输入测试。

### 5.2 `node-runtime`

- [ ] 构建单个 Sandbox LiveNode 并注册策略、Data Client 和 Execution Client。
- [ ] 实现启动、停止、停止新增敞口、撤单、状态查询和安全关闭。
- [ ] 上报心跳、行情陈旧、订单、仓位、队列和持久化状态。
- [ ] 确保一个进程只运行一个并发 LiveNode。
- [ ] 对断线、慢消费、数据库不可用和策略 panic 增加故障测试。

### 5.3 `supervisor`

- [ ] 实现 Node 子进程 spawn、隔离目录、环境和凭证引用注入。
- [ ] 实现 IPC 会话、协议协商、心跳、资源监控和退出分类。
- [ ] 实现重启预算、退避、人工停止和 Kill Switch 编排。
- [ ] 实现 Supervisor 重启后的 RuntimeInstance 恢复和 ReconciliationBlocked 状态。
- [ ] 禁止对 Unknown 资金操作执行无条件自动重试。

### 5.4 应用入口与控制面集成

- [ ] 完成 `mountsea-node-runtime` 和 `mountsea-supervisor` 的依赖装配。
- [ ] 将 Deployment 审批、运行命令和状态查询接入 Application/API。
- [ ] 实现日志、指标和运行事件的统一关联。
- [ ] 在 Sandbox 执行长时间 Soak 和故障注入。

**Gate 5：** Sandbox 下断线、Node 崩溃、Supervisor 重启、队列积压和数据库延迟均有确定结果；对账通过前不会自动恢复策略。

## 阶段 6：平台风险、对账与可观测性加固

目标：在连接真实 Venue 前，使系统具备可检测、可停止、可恢复和可审计能力。

### 6.1 `risk`

- [ ] 实现账户、策略、Instrument 和组合级限额。
- [ ] 实现 Stop-new-exposure、Cancel-all 和 Kill Switch 决策。
- [ ] 定义风险策略版本、优先级、覆盖规则和人工审批。
- [ ] 验证平台风控与 NautilusTrader RiskEngine 不冲突且不能绕过后者。

### 6.2 对账与 Unknown Outcome

- [ ] 保存原始 Venue Report、命令、请求关联和决策证据。
- [ ] 实现订单、成交、仓位、余额和账户状态差异分类。
- [ ] 实现 Unknown Outcome 查询、人工处置和恢复 Gate。
- [ ] 为每类差异建立 Runbook 和自动化演练。

### 6.3 可观测性与恢复

- [ ] 建立 API、Worker、Supervisor、Node、PostgreSQL 和 Catalog 指标。
- [ ] 监控最旧待写消息、队列深度、行情陈旧、Socket 状态和 Node RSS。
- [ ] 建立结构化日志、分布式关联、告警分级和值班 Runbook。
- [ ] 实现 PostgreSQL、Catalog、Artifact 和审计证据备份恢复演练。
- [ ] 建立容量基准和 1.5～2 倍峰值压力测试。

**Gate 6：** 风险动作、Unknown Outcome、对账、告警和备份恢复均通过故障注入；任何无法解释的状态差异都会阻止交易恢复。

## 阶段 7：目标 Venue 小资金 Live

目标：仅针对一个明确的 Venue 和账户，以最小资金验证真实交易闭环。

- [ ] 选择首个 Venue、账户类型、Instrument 范围和订单类型白名单。
- [ ] 建立 Venue 能力矩阵：行情、下单、改单、撤单、限频、幂等和历史报告。
- [ ] 集成 Secrets Manager，默认禁用提现权限并实施最小权限。
- [ ] 完成网络超时、发送后断开、重复回报和限频测试。
- [ ] 完成部署双人审批、启动前对账和人工接管流程。
- [ ] 使用最小资金灰度，设置严格损失、仓位和运行时间上限。
- [ ] 完成观察期复盘，要求无未解释订单、成交、仓位或余额差异。

**Gate 7：** Venue、Live、Capacity Gate 全部通过；人工 Kill Switch 和恢复演练成功；观察期内没有未解释差异，才允许扩大范围。

## 阶段 8：扩展能力

以下功能不得提前阻塞阶段 0～7：

- [ ] 多账户和多 Venue。
- [ ] 同账户多 Node 的集中风险、订单归属和组合汇总。
- [ ] 执行算法、套利、做市和期权能力。
- [ ] Python Research SDK 和 Notebook 工作流。
- [ ] 策略构建服务、Artifact 签名和受控第三方策略。
- [ ] 高可用控制面和多主机 Node 调度。
- [ ] 更复杂的前端分析、报表和运营能力。

## 横向完成标准

每一个功能项在标记完成前都必须满足：

1. 业务规则和错误语义已有文档或 ADR。
2. 公共接口具有单元测试，基础设施边界具有集成或契约测试。
3. 新增副作用具备幂等、审计、超时和失败恢复设计。
4. 新增后台循环具备有界队列、取消、关闭和可观测性。
5. 不记录凭证、密钥和其他敏感信息。
6. `cargo fmt --check`、Clippy、测试和文档检查通过。
7. 对应 README、架构、配置样例和运维 Runbook 已更新。

## 当前建议的第一批开发任务

在尚未开始业务实现的当前状态下，建议严格按以下顺序领取工作：

1. 阶段 0.1：CI、固定工具链和依赖审计。
2. 阶段 1.1：领域 ID、值对象和五个核心状态机。
3. 阶段 1.2：配置模型、严格解析和 SecretReference。
4. 阶段 1.3：策略描述、参数 Schema 和工厂注册表。
5. 阶段 2.1：Repository 端口、PostgreSQL 选型和首批迁移。
6. 阶段 2.2：DatasetVersion、Manifest 和 CommitMarker。
7. 阶段 2.3：`catalog_capture` 集成 Spike。
8. 阶段 3.1：NautilusTrader BacktestEngine 最小适配。
9. 阶段 3.2：单策略、单数据集可复现回测。
10. 阶段 3.3：CLI 与 Backtest Worker 端到端闭环。
