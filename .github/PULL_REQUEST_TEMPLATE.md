## 变更说明

用 1-3 句话说明改了什么、为什么改。若涉及架构或公开行为变更，先确认设计规格已同步。

## 关联 Issue

Closes #

## 变更类型

- [ ] `feat` 新功能
- [ ] `fix` 缺陷修复
- [ ] `docs` 文档
- [ ] `refactor` / `perf` / `test`
- [ ] `build` / `ci` / `chore`

## 检查清单

- [ ] PR 标题符合约定式提交（squash 合并后即为提交信息）
- [ ] `uv run ruff check .` 与 `uv run ruff format --check .` 通过
- [ ] `uv run mypy src` 通过
- [ ] `uv run pytest -q` 通过（若 `tests/` 尚为空，在此注明）
- [ ] 未提交商业版权音源、超过 5 MiB 的文件、密钥或本地绝对路径
- [ ] 公开行为或架构变更已同步设计规格与 `CHANGELOG.md` 的 `Unreleased` 段
- [ ] 新增依赖已同步 `uv.lock` 并在描述中说明理由
