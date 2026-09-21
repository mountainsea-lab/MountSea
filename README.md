# MountSea

MountSea 是一个 Rust-first 的量化策略研究、回测、Sandbox 和实盘交易平台。

## 定位

MountSea 构建于 NautilusTrader 之上，负责应用层和控制面，不重新实现交易引擎、订单状态机、
RiskEngine、ExecutionEngine、Portfolio 或 Venue Adapter。

## 项目结构

```text
mountsea/
├── apps/                         # 可部署的薄应用入口
│   ├── api/                      # Web API
│   ├── cli/                      # 命令行入口
│   ├── supervisor/               # 进程监督入口
│   ├── node-runtime/             # 隔离交易 Node 入口
│   ├── backtest-worker/          # 回测 Worker 入口
│   └── capture-worker/           # 行情采集 Worker 入口
├── crates/                       # 可复用、可组合的功能库
│   ├── application/              # 应用用例与事务编排
│   ├── domain/                   # 领域对象与状态机
│   ├── config/                   # 配置加载与校验
│   ├── storage/                  # PostgreSQL 与运行状态边界
│   ├── catalog/                  # 数据集版本、Manifest 与提交协议
│   ├── runtime-adapter/          # NautilusTrader 运行时适配
│   ├── node-protocol/            # Supervisor 与 Node 的版本化 IPC
│   ├── risk/                     # 账户、策略和组合级风控
│   ├── strategy/                 # 策略契约、元数据、校验与工厂注册
│   ├── supervisor/               # 生命周期、健康检查与恢复编排
│   ├── node-runtime/             # LiveNode 构建与运行编排
│   ├── backtest/                 # 回测执行能力
│   ├── capture/                  # 行情采集能力
│   └── cli/                      # 可复用 CLI 命令用例
├── docs/
├── migrations/
├── configs/
├── examples/
├── python/
└── frontend/
```

## 组合原则

`apps/` 只负责配置加载、依赖装配、进程启动和关闭。业务能力位于 `crates/`，可按场景组合为研究
回测工具、Sandbox、单机交易系统或完整 Web 平台。库使用功能名称，不绑定 MountSea 品牌。

进程组合不改变安全边界。生产实盘仍保持一个 `LiveNode` 对应一个隔离的 Node 进程。

## 当前状态

当前仓库是架构骨架。交易功能、数据库迁移、外部凭证和生产部署尚未实现。完整设计参见
[`docs/architecture.md`](docs/architecture.md)。

## 开源协议

MountSea 与 `catalog_capture` 保持一致，采用
[GNU Lesser General Public License v3.0 or later](LICENSE)。


## 代码仓库

- 默认分支：`main`
- 远程仓库：<https://github.com/mountainsea-lab/MountSea>
