# Go CLI Meta Scaffold Blueprint

本文件用于给 AI 一个稳定、可推断的 Go CLI 元工程蓝图。不是所有目录都必须强制存在，但需要保持命令层级、配置入口、输出策略与发布链路清晰。

## 推荐目录

```text
.
├── AGENTS.md
├── ARCHITECTURE.md
├── README.md
├── .env.example
├── .gitignore
├── .gitlab-ci.yml
├── .goreleaser.yml
├── Taskfile.yml
├── go.mod
├── go.sum
├── cmd/
│   └── <cli-name>/
│       ├── main.go
│       ├── root.go
│       ├── version.go
│       ├── completion.go
│       └── <business-command>.go
├── internal/
│   ├── app/
│   │   └── app.go
│   ├── config/
│   │   └── config.go
│   ├── output/
│   │   └── printer.go
│   ├── clierror/                 # 按需；结构化错误与退出码映射
│   │   └── error.go
│   ├── version/
│   │   └── version.go
│   └── <domain>/
│       └── runner.go
├── pkg/                          # 按需；仅放可复用公共库
├── scripts/
│   └── ci/
├── dist/                         # 构建产物，不提交
└── tests/
    ├── e2e/
    └── testdata/
```

## 依赖方向

- `cmd` 只负责命令行入口、flags 注册、错误返回
- `internal/app` 负责初始化配置、输出器、业务 runner
- `internal/config` 负责配置默认值、配置文件解析、环境变量覆盖
- `internal/output` 负责统一文本/JSON/quiet 等输出策略
- `internal/clierror` 负责结构化错误、错误类别、retry 语义与退出码映射
- `internal/<domain>` 负责具体业务命令逻辑
- `pkg` 只有在其他仓库也要 import 时才启用；不要把本项目私有逻辑塞进去

## 命令组织约束

- `main.go` 只负责执行根命令
- `root.go` 持有全局 flags 与配置入口
- `version.go` 输出版本、提交 SHA、构建时间
- `completion.go` 生成 shell completion
- 业务命令单独分文件维护，并通过 `init()` 注册到 root command
- 用户未给出具体业务命令时，可先用 `run` 作为占位命令，但交付时应明确这是待替换的业务入口
- 高频任务优先做 workflow/shortcut 命令；主业务对象做 curated commands；raw API 或底层透传只作为 escape hatch
- 位置参数只用于不可混淆的单一主对象；容易填反或可选的输入必须使用命名 flags
- 复杂对象输入支持 `--data @file.json` 或 `--data -`，避免让 agent 拼接超长 shell quoting

## 配置与输出约束

- 如果需要配置文件，统一使用 `--config` / `-c`
- 环境变量应有稳定前缀，例如 `FOO_BAR_`
- 配置读取集中在 `internal/config`，不要在各命令文件中零散读取环境变量
- 普通输出和错误输出应通过统一 printer 或 output helper 处理
- 根命令应提供 `--format json|ndjson|table|pretty|yaml`，并可保留 `--output` 作为兼容别名
- 支持 `--quiet`、`--no-input`、`--no-color`、`--trace-id`；自动化路径默认不进入交互式 prompt
- stdout 只输出主结果；stderr 输出进度、warning、诊断和 human hint
- JSON 成功输出建议包含 `ok`、`command`、`trace_id`、`dry_run`、`mutated`、`result`、`warnings`、`next`
- JSON 错误输出建议包含 `ok=false`、`error.code`、`error.category`、`error.field`、`retryable`、`safe_to_retry`、`suggested_commands`
- 大结果集优先提供分页、`--fields`、`--query` 或 `--format ndjson`
- secrets、token、cookie、连接串默认脱敏；必要时支持 `--output none`

## 副作用与安全

- 所有写、删、发送、审批、改权限命令必须提供 `--dry-run` 或等价 preview
- dry-run 输出应回显目标资源、resolved config、权限/scope、风险等级和预计副作用
- destructive 操作使用 `--confirm <resource-id>`；`--yes` 只能跳过普通确认，不能绕过权限或安全策略
- 批量操作应提供 `--limit`、`--max-mutated`、`--fail-fast`、`--continue-on-error`
- 重试敏感命令应提供 idempotency key、client token 或冲突检测
- 外部文档、网页、消息等不可信内容进入输出时，应带 `trusted=false`、`source`、`content_type` 等元数据

## Taskfile 约束

至少建议包含：

- `lint`
- `test`
- `build`
- `run`
- `release:dry`
- `deps`
- `completion:bash`

建议：

- `lint` 串联 `go vet` 与 `golangci-lint`
- `deps` 执行 `go mod tidy`
- `build` 输出到 `dist/bin/<cli-name>`
- `run` 直接执行 `go run ./cmd/<cli-name> -- --help` 或用户传入参数
- `release:dry` 使用 `goreleaser release --snapshot --clean`

## 发布与 CI 约束

- `.goreleaser.yml` 用于最小多平台二进制发布
- GitLab CI 优先通过 `task` 执行公共任务
- stages 至少覆盖 `lint`、`test`、`build`、`release`
- `release` 阶段不应重复定义另一套复杂构建逻辑

## 测试策略

- 单元测试覆盖配置解析、输出格式、核心业务 runner
- 端到端测试至少覆盖：
  - `version`
  - 业务占位命令的 happy path
  - 至少一个错误路径
  - `--format json` 成功与失败路径
  - `--no-input` 缺参路径
  - `--dry-run` 对写操作的 preview
- 如果 CLI 依赖外部服务，应为 e2e 提供可替换的假实现或显式说明不在 scaffold 层内覆盖

## AI 协作约束

项目仓库内应生成以下文档：

- `AGENTS.md`
  - 说明目录职责
  - 说明 `dist/`、release artifacts、completion 输出属于构建产物，不直接修改
  - 指导 AI 先读什么，再改什么
- `ARCHITECTURE.md`
  - 解释命令组织、配置流、输出流、发布流
- `README.md`
  - 面向人类，写运行、开发、测试、发布
