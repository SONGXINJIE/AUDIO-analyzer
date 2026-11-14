# Contributing

感谢参与。本项目处于 Alpha 阶段，架构决策以
[设计规格](docs/superpowers/specs/2026-11-14-audio-analyzer-platform-design.md) 为准；
参与贡献即表示同意遵守 [行为准则](CODE_OF_CONDUCT.md)。

## 1. 开发环境

| 项目 | 要求 |
|---|---|
| Python | `>=3.12,<3.13` |
| 依赖管理 | `uv`（`uv.lock` 为唯一依赖真相） |
| 操作系统 | Windows / Linux / macOS 均可；CI 运行于 `ubuntu-latest` |

```bash
git clone https://github.com/SONGXINJIE/AUDIO-analyzer.git
cd AUDIO-analyzer
uv sync
uv pip install pre-commit
pre-commit install
```

不要提交 `.venv/`、`uv.lock` 的本地改动（`uv lock` 后方可提交）、缓存目录或任何音频素材。

## 2. 分支策略（GitHub Flow）

- `main` 是唯一长期分支，受保护，禁止直接推送。
- 所有变更从 `main` 切出短生命周期分支：`feat/<slug>`、`fix/<slug>`、`docs/<slug>`、`chore/<slug>`。
- 通过 Pull Request 合并，采用 **squash merge**，因此 **PR 标题即最终提交信息**，必须符合约定式提交。
- 合并后删除源分支；不设 `develop` / `release` 长期分支。

## 3. 约定式提交

格式：`type(scope)!: subject`

| type | 用途 |
|---|---|
| `feat` | 新功能 |
| `fix` | 缺陷修复 |
| `docs` | 文档 |
| `style` | 格式，不改变行为 |
| `refactor` | 重构，不改变行为 |
| `perf` | 性能优化 |
| `test` | 测试 |
| `build` | 构建系统或依赖 |
| `ci` | CI 配置 |
| `chore` | 其他维护事项 |
| `revert` | 回滚 |

- `scope` 可选，建议使用层名或模块名：`contracts`、`core`、`plugins`、`cli`、`deps`、`ci`。
- `!` 表示破坏性变更（pre-1.0 阶段同样标注），需在 PR 描述中说明迁移方式。
- `subject` 使用祈使句、小写开头、不加句号，长度不超过 72 字符。
- 示例：`feat(contracts): add FeatureExtractor protocol`、`chore(deps): bump ruff`、`docs: align README with CPU-only design`。

PR 标题由 `.github/workflows/conventional-commits.yml` 自动校验；本地不强制安装 commitlint。

## 4. Pull Request 流程

1. 创建分支并在本地完成变更。
2. 运行第 5 节全部验证命令，确保通过。
3. 若变更影响公开行为或架构，先更新设计规格与 `CHANGELOG.md` 的 `Unreleased` 段。
4. 开 PR，填写 `.github/PULL_REQUEST_TEMPLATE.md` 中的检查清单，关联对应 Issue。
5. CI 全绿 + 至少一次审查通过后由维护者 squash 合并。

## 5. 本地验证

```bash
uv run ruff check .
uv run ruff format --check .
uv run mypy src
uv run pytest -q
```

> 当前状态：业务实现尚未开始，`tests/` 目录为空。pytest 会因
> `pyproject.toml` 的 `testpaths` 与 `filterwarnings = ["error"]` 组合在配置解析阶段报错，
> 这是设计规格登记的 P0 前置修复项。CI 在 `tests/**/*.py` 存在前自动跳过 pytest 步骤。

## 6. 设计变更

以下变更必须先修改设计规格并获得批准，再动代码：

- 调整四层边界或依赖方向；
- 修改 `contracts` 中的公开数据模型、`Protocol` 或 `CONTRACT_VERSION`；
- 改变验收标准（AC-1 ~ AC-6）或非目标范围；
- 引入新的运行时第三方依赖。

## 7. 开源合规红线

- **不得提交商业版权音源**。`audio-test/` 已被 `.gitignore` 排除；仓库仅允许根目录两个合成 WAV fixture。
- 不得提交密钥、令牌、`.env`、证书或含本地绝对路径的内部记录（`.workbuddy/` 已排除）。
- 单文件超过 5 MiB 的产物不得入库，CI 会阻断。
- 新增依赖须在 PR 中说明理由，并同步 `uv.lock`。

## 8. 报告问题

缺陷与需求请使用 `.github/ISSUE_TEMPLATE/` 中的模板。安全漏洞**不要**开公开 Issue，
按 [SECURITY.md](SECURITY.md) 的私密渠道报告。
