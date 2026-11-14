# audio-analyzer

纯 CPU、插件化的音频分析平台。采用四层架构（契约 / 核心 / 插件 / 适配器），特征提取器、编解码器与报告器可独立开发、独立测试、按需组合。

> **项目状态：Alpha / 设计阶段。** 当前仓库只包含工程治理规范、设计规格与最小包骨架，业务实现尚未开始。请勿在生产环境使用。

## 基本信息

| 项目 | 值 |
|---|---|
| 版本 | `0.1.0`（SemVer；pre-1.0，公开 API 可能变更） |
| Python | `>=3.12,<3.13` |
| 计算后端 | 仅 CPU；不提供 GPU / CUDA 路径 |
| 许可证 | MIT |
| 开发状态 | 3 - Alpha |

## 架构

| 层 | 职责 |
|---|---|
| `contracts` | 数据模型、协议、插件元数据、异常基类；零第三方依赖 |
| `core` | 注册表、DAG 编排、共享数值上下文、内容寻址缓存、错误映射 |
| `plugins` | 特征提取器、编解码器、报告器 |
| `adapters` | 首版仅 CLI |

依赖单向向下：基座不感知插件，插件不感知适配器。完整设计见 [设计规格](docs/superpowers/specs/2026-11-14-audio-analyzer-platform-design.md)。

## 安装

```bash
git clone https://github.com/SONGXINJIE/AUDIO-analyzer.git
cd AUDIO-analyzer
uv sync
```

依赖由 `uv.lock` 锁定，`uv` 版本与开发环境配置见 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 当前可用能力

现阶段仅保证包可导入，不含分析功能：

```bash
uv run python -c "import audio_analyzer; print(audio_analyzer.__version__)"
```

## 开发

```bash
.venv/Scripts/ruff.exe check src
.venv/Scripts/python.exe -m mypy src
.venv/Scripts/python.exe -m pytest -q
```

质量门禁、提交约定与分支策略见 [CONTRIBUTING.md](CONTRIBUTING.md)。测试目录建立前，pytest 会因 `testpaths` 配置在解析阶段报错，这是设计规格中登记的 P0 前置修复项。

## 文档索引

| 文档 | 内容 |
|---|---|
| [CONTRIBUTING.md](CONTRIBUTING.md) | 分支策略、约定式提交、PR 流程、本地验证 |
| [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) | 社区行为准则 |
| [SECURITY.md](SECURITY.md) | 漏洞报告渠道与支持版本 |
| [CHANGELOG.md](CHANGELOG.md) | 版本变更记录 |
| [CONTEXT.md](CONTEXT.md) | 领域术语与关键约束 |
| [设计规格](docs/superpowers/specs/2026-11-14-audio-analyzer-platform-design.md) | 目标、非目标、验收标准与架构设计 |

## 测试数据

仓库根目录的 `test_audio.wav` 与 `_test_meta.wav` 是程序生成的合成信号，可自由使用。`audio-test/` 目录含商业版权音源，已被 `.gitignore` 排除，**不得提交或分发**；本地回归测试请使用自备的合法音源。

## 许可证

[MIT](LICENSE) — 详见 `LICENSE` 文件。
