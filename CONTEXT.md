# CONTEXT.md — audio-analyzer 领域上下文

> 本文件记录当前架构（v0.1.0）的领域术语与关键约束。术语或约束变更时同步更新。
> 早期 v5.8.0 的 GPU / CUDA / CuPy 设计已废弃，不再作为本项目约束。

## 术语表

### Contract（契约）

`contracts` 层定义的冻结数据模型、`typing.Protocol` 与插件元数据。零第三方依赖，是唯一允许被所有层引用的稳定接口。契约主版本由 `CONTRACT_VERSION` 标识，插件注册时校验一致性。

### Plugin（插件）

实现某个 `Protocol` 的可插拔单元，分三类：特征提取器（`FeatureExtractor`）、编解码器（`Codec`）、报告器（`Reporter`）。首版内置于同仓库 `plugins/`，通过注册表按名发现，暂不独立分发。

### Registry（注册表）

`core/registry.py` 中负责插件注册、能力校验与 `discover(source)` 的组件。加载期冻结，运行期只读；契约主版本不匹配的插件会被拒绝注册。

### AnalysisContext（分析上下文）

预处理（mono / resample / normalize）只执行一次后产出的只读对象，供所有特征插件共享。插件不得修改它，也不得持有可变共享状态。

### Profile（配置档案）

`configs/*.yaml` 中的预设参数组合。`plugins.enabled` 声明本次分析启用的插件名清单；新增插件无需修改配置 schema。

### Content-Addressed Cache（内容寻址缓存）

缓存键 = `source_hash + 插件名 + 插件版本 + 契约版本 + 参数摘要`。只缓存 `FeatureResult`，序列化为 JSON，禁用 pickle。

## 关键约束

- **计算后端**：仅 CPU。不实现 GPU / CUDA / CuPy 路径，不编写自定义 Numba JIT 内核。
- **依赖事实**：`librosa 0.11` 硬依赖 `numba>=0.51.0`，故 `numba` / `llvmlite` 作为传递依赖保留。
- **Python 版本**：`>=3.12,<3.13`。
- **正确性不变量**：内容寻址缓存、插件失败降级且必写 warning、profile 名称白名单、错误信息截断 200 字符、缓存禁用 pickle、共享上下文只读。
- **开源合规**：`audio-test/` 中的商业版权音源不得提交或分发；`.workbuddy/` 为内部数据，不得公开。

## 参见

设计规格：`docs/superpowers/specs/2026-11-14-audio-analyzer-platform-design.md`
