# CI-workflow

> 用户仓库 **CI workflow（含门禁）** 的权威模板：零依赖运行器、五条 lane、报告/用例 schema、可复制的 starter。
> 公共多仓 Dashboard：[`agent-CI-Infra/dashboard`](https://github.com/agent-CI-Infra/dashboard)
> 单仓/私仓看板模板：[`dashboard/single-repo`](https://github.com/agent-CI-Infra/dashboard/tree/main/single-repo)

## 它解决什么

把“门禁自动跑起来”所需的最小文件放进用户仓：

- GitHub Actions 工作流：PR / merge queue / push / schedule / release 自动触发；
- `gate.mjs`：零依赖 runner，按事件执行 lane、裁决、写报告；
- `ci-gates.json`：lane 命令、等级、observe/enforce；
- `case-catalog.yaml`：用例目录，ready/todo/skipped；
- 报告 artifact：`ci-gates-reports`。

> **命名兼容**：仓名从 `ci-gates` 改为 `CI-workflow`；协议标识 `ci-gates/*`、artifact `ci-gates-reports`、配置 `.github/ci-gates.json` 保留。

它不负责：分支保护设置、merge queue 开启、单仓看板 UI、公共 Dashboard 聚合。这些分别由仓库设置、[`single-repo`](https://github.com/agent-CI-Infra/dashboard/tree/main/single-repo) 和 [`dashboard`](https://github.com/agent-CI-Infra/dashboard) 负责。

## 文件

```text
scripts/gate/gate.mjs          # 运行器，Node 20+
scripts/gate/gate.test.mjs     # 运行器自测
template/.github/workflows/ci.yml
template/.github/ci-gates.json
template/test/cases/case-catalog.yaml
template/test/cases/example.test.ts
protocol/*.schema.json         # gate-report / case-catalog / board-summary
examples/                      # 协议示例
AGENT.md                       # 面向用户 AI 的操作手册
```

## 三步接入

在用户仓根目录执行（把 `<CI-workflow>` 换成本仓路径）：

```bash
# 1. 复制 workflow、配置和用例骨架
mkdir -p .github scripts/gate test/cases
cp -R <CI-workflow>/template/.github/. .github/
cp <CI-workflow>/scripts/gate/gate.mjs scripts/gate/
cp <CI-workflow>/template/test/cases/case-catalog.yaml test/cases/
cp <CI-workflow>/template/test/cases/example.test.ts test/cases/

# 2. 改 .github/ci-gates.json 里的命令为项目真实命令
# 3. 提交并开 PR
```

要求：

- workflow name 为 `CI`（或让看板 `workflows.gate` 匹配它）；
- 上传 artifact `ci-gates-reports`；
- 先不填用例：空 catalog + 从 `pr-ready` 去掉 `verify-cases`；
- 填用例：`ready` 需要 `file` + `owner` + 文件内 `@case <id>`。

完整步骤见 [`AGENT.md`](AGENT.md)。

## 事件与 lane

一个 `gate` job，按事件执行对应 lane：

| 事件 | lane |
| --- | --- |
| `pull_request` | `build`、`pr-ready` |
| `merge_group` | `build`、`pr-merge` |
| `push` 到 main | `build` |
| `schedule` | `daily` |
| `release` | `release` |
| `workflow_dispatch` | `build`、`pr-ready` |

需要单跑某条 lane：

```bash
node scripts/gate/gate.mjs run --lane daily
```

## 用例

```yaml
schema: ci-gates/case-catalog@1
version: 1
cases:
  - id: build.app
    lane: build
    layer: L0
    title: 应用可以构建
    status: ready
    file: src/app.js
    owner: '@owner'
```

规则：

- `ready`：必须有 `file`、`owner`，且文件中有 `@case <id>`；
- `todo`：可以没有实现，不要加 marker；
- `skipped`：必须有 `until`、`reason`；
- `verify-cases` 扫描 `caseRoot`（默认 `test`，可配置数组），未登记 marker 会失败。

## 报告

```text
gate-reports/
  build.json
  pr-ready.json
  ...
```

artifact：`ci-gates-reports`。报告 schema 见 [`protocol/gate-report.schema.json`](protocol/gate-report.schema.json)。

单仓看板消费报告后展示「质量门禁」卡片；公共 Dashboard 消费单仓看板发布的公开 summary。

## 版本

- `v1`：当前稳定模板，用户仓和两个 Dashboard 引用该 tag；
- 破坏性变更升 `v2`；
- 不要在用户仓直接跟踪 `main`，用 tag 固定版本。

## 本地验证

```bash
node --test scripts/gate/gate.test.mjs
node scripts/gate/gate.mjs verify-cases --config .github/ci-gates.json
```

## License

MIT
