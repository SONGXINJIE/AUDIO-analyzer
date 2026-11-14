# 音频分析平台设计规格

**日期**：2026-11-14
**状态**：待用户复核
**项目根目录**：仓库根目录
**方法**：brainstorming，经路线选择、插件分发、架构第 1 节、数据流第 2 节、错误处理与测试第 3/4 节共五轮确认
**目标**：在清理后的最小骨架上，构建专业级、可扩展、纯 CPU 的音频分析平台

---

## 1. 规格边界

### 1.1 目标

1. 建立稳定的四层架构，使特征提取器、编解码器、报告器可独立开发、独立测试、按需组合。
2. 契约层按可独立分发的标准定义，未来拆分为独立 pip 包时无需修改公开 API。
3. 交付首版可用的 CLI 适配器与 Python 库 API，对本地音频文件产出可复现、可解释的分析结果。
4. 把正确性不变量（内容寻址缓存、失败必留痕、输入白名单、禁用 pickle）固化为架构约束，而非依赖实现者自觉的约定。

### 1.2 非目标

1. 不实现 GPU 加速：不使用 CUDA / CuPy 计算后端，不注册多计算后端插件。
2. 「纯 CPU」的精确定义：**不实现 GPU 计算路径，也不编写自定义 Numba JIT 内核**；但 `numba` / `llvmlite` 作为 `librosa>=0.11` 的硬传递依赖必须保留（实测 `librosa` 声明 `Requires-Dist: numba>=0.51.0`，移除会破坏依赖解析）。
3. 首版不实现 REST / 桌面适配器；不实现插件沙箱、热加载、远程市场、`entry_points` 实际分发（仅预留接口）。
4. 不实现实时流式分析、音频编辑、播放器、音频合成。
5. 不重建已删除的 131 份审计文档与 100+ 历史测试用例；历史工程经验以本规格中显式列出的约束继承。

### 1.3 验收标准

| 编号 | 标准 | 判定方式 |
|---|---|---|
| AC-1 确定性 | 同文件 + 同 profile 连跑两次，`AnalysisResult` 除 `timings` 外在相对容差 `1e-9` 内一致 | 集成测试 |
| AC-2 可复现 | 结果含 `source_hash`、契约版本、各插件版本，可离线复算 | 结果 schema 校验 |
| AC-3 可解释 | 每个 `FeatureResult` 带单位、置信度、诊断；`warnings` 不丢失 | 契约测试 |
| AC-4 可隔离 | 任一插件异常或超时，其余特征仍产出，且 `warnings` 必含该插件 | 集成测试 |
| AC-5 分层 | `import audio_analyzer.contracts` 不加载任何第三方包；import-graph 满足 4.3 规则 | ast 契约测试 |
| AC-6 质量门禁 | ruff 0 错、mypy strict 0 错、覆盖率 ≥70%、`filterwarnings=error` 全通过 | 第 11 节验证命令 |

### 1.4 边界条件

- 空文件、全静音、单样本、超长文件、非有限样本值（NaN / Inf）
- 不支持的编码、损坏文件头、采样率异常、多声道输入
- profile 名称非法、插件依赖成环、依赖缺失、契约主版本不匹配
- 缓存目录不可写、同路径内容被原地覆写、并发写缓存
- 插件超时、插件抛出非预期异常、插件返回不可序列化值

---

## 2. 项目上下文

### 2.1 骨架现状（实测 2026-11-14）

清理后非环境条目 43 个：`.gitignore`、`pyproject.toml`（6168 B）、`uv.lock`（128429 B，70 个包）、`README.md`、`CONTEXT.md`、`configs/` 7 个 YAML、`src/audio_analyzer/{__init__.py, py.typed}`、`audio-test/` 20 个 flac（596 MB）、`test_audio.wav`、`_test_meta.wav`、`.venv/`（Python 3.12.0）、`.workbuddy/`。业务实现代码为零。

### 2.2 可复用资产

`pyproject.toml` 的 strict mypy / ruff / pytest 门禁；`uv.lock` 锁定的 70 个包；`.venv` 中已安装的全套依赖（无需重新下载）；`configs/` 的 profile 分层思想；`CONTEXT.md` 的领域术语；20 首多语种真实语料，作为金标回归基线。

### 2.3 实测阻塞项（P0）

`pytest --version` 与 `pytest --collect-only` 当前即崩溃：`filterwarnings=["error"]` 把 `PytestConfigWarning: No files were found in testpaths` 升级为异常，在配置解析阶段终止，早于任何测试收集或覆盖率计算。工具链本身可用：ruff 0.15.21、mypy 1.20.2、pytest 8.4.2、coverage 7.15.1、uv 0.11.28、Python 3.12.0。

---

## 3. 已确认决策

| ID | 决策 | 确认轮次 |
|---|---|---|
| D-1 | 采用可扩展平台 / 框架路线（方案 C） | 路线选择 |
| D-2 | 同仓库起步，契约层按可独立分发标准定义；现在不引入 `entry_points` | 插件分发 |
| D-3 | 永久纯 CPU；计算后端概念下移核心层内部策略，仅一个 NumPy 实现，不做多后端插件 | 第 1 节 |
| D-4 | `configs/*.yaml` 的 `features.enabled` 改为插件名清单 `plugins.enabled` | 第 1 节 |
| D-5 | 契约层用标准库 dataclass + Protocol，不用 Pydantic；Pydantic 用于配置加载与 CLI 校验 | 第 1 节 |
| D-6 | 共享 `AnalysisContext` 只读，预处理只执行一次 | 第 2 节 |
| D-7 | 线程池并行；单插件失败降级并强制写入 `warning`，不中断整次分析 | 第 2 节 |
| D-8 | 内容寻址缓存，只缓存 `FeatureResult` | 第 2 节 |
| D-9 | 异常基类上移 `contracts/errors.py`；`core/errors.py` 负责分类映射与降级策略 | 第 3 节自查 |
| D-10 | 缓存用 JSON 序列化 + 显式 schema 校验，禁用 pickle | 第 3 节 |
| D-11 | 确定性验收用相对容差 `1e-9`，不用严格逐字段相等 | 第 3 节精化 |

---

## 4. 架构

### 4.1 四层模型

| 层 | 角色 | 稳定性 |
|---|---|---|
| 契约层 | 数据模型、协议、插件元数据、异常基类；零第三方依赖 | 稳定基座 |
| 核心层 | 注册表、DAG 编排、共享数值上下文、内容寻址缓存、错误映射 | 稳定基座 |
| 插件层 | 特征提取器、编解码器、报告器 | 可变外围 |
| 适配层 | 首版仅 CLI | 可变外围 |

依赖单向向下。基座不感知插件，插件不感知适配器。

### 4.2 包结构

```text
src/audio_analyzer/
├── __init__.py          仅再导出 contracts 公共 API
├── py.typed
├── contracts/          仅标准库
│   ├── models.py        AudioSegment / AnalysisContext / FeatureResult / AnalysisResult
│   ├── protocols.py     FeatureExtractor / Codec / Reporter
│   ├── metadata.py      PluginMeta / CONTRACT_VERSION / capabilities
│   └── errors.py        AudioAnalyzerError / PluginError / DecodeError / ConfigError
├── core/               依赖 contracts
│   ├── registry.py      注册、能力校验、按名发现 discover(source)
│   ├── orchestrator.py  DAG 解析、调度、结果聚合
│   ├── compute.py       NumPy 共享数值上下文（预处理、谱上下文）
│   ├── cache.py         内容寻址缓存（JSON + 原子写）
│   └── errors.py        错误分类映射与降级策略
├── plugins/            首版内置，物理同仓库
│   ├── features/
│   ├── codecs/
│   └── reporters/
└── adapters/
    └── cli/            首版唯一适配器
```

`plugins/` 不含 `backends/`：D-3 已把计算后端降为 `core/compute.py` 的内部策略，不作为 Protocol 暴露，也不进注册表。`core/compute.py` 仅由核心层调用，插件只读其产出的 `AnalysisContext`。

### 4.3 依赖规则

| 层 | 允许 import | 禁止 import |
|---|---|---|
| `contracts` | 仅标准库 | 任何第三方包、项目其他层 |
| `core` | `contracts` + 标准库 | `plugins`、`adapters` |
| `plugins` | `contracts` + 自身计算库（numpy / scipy / librosa 等） | `core` 内部实现、其他插件、`adapters` |
| `adapters` | `core` + `contracts` | `plugins` 具体模块（必须经注册表发现） |

校验方式：pytest 中用标准库 `ast` 解析 import-graph 的契约测试，不引入 `import-linter` 等新依赖。

### 4.4 契约版本化与可分发预留

`CONTRACT_VERSION = 1`（整数主版本）。`PluginMeta` 为冻结 dataclass，字段：`name`、`version`、`contract_version`、`provides`、`requires`。注册时校验 `contract_version == CONTRACT_VERSION`，不匹配即拒绝注册，并给出包含插件名与实际 / 期望双版本的可诊断错误。

`discover(source)` 当前只有内置源；未来新增 `entry_points` 源不改变任何调用方签名。这是 D-2「预留可分发契约」的唯一技术兑现点，首版不实现外部发现。

---

## 5. 组件契约

### 5.1 数据模型

全部为 `@dataclass(frozen=True, slots=True)`。

| 模型 | 字段 |
|---|---|
| `AudioSegment` | `samples`（只读数组视图）、`sample_rate: int`、`channels: int`、`duration_s: float`、`source_hash: str` |
| `AnalysisContext` | `segment`、`preprocessed`（只读）、`params`、`profile` |
| `FeatureResult` | `name`、`plugin_version`、`value`、`unit`、`confidence`、`diagnostics`、`status`（`ok` / `degraded`） |
| `AnalysisResult` | `source`、`profile`、`contract_version`、`features: dict[str, FeatureResult]`、`timings`、`warnings`、`schema_version` |

**序列化约束**：`FeatureResult.value` 仅允许 JSON 可序列化类型（`int` / `float` / `str` / `bool` / `None` / `list` / `dict`），且不得为 NaN 或 Inf。违反即视为插件级失败并降级。数组须先归约为标量或显式列表。此约束是 D-10 禁用 pickle 的前提，否则缓存无法序列化。

### 5.2 协议

均为 `typing.Protocol` 且 `runtime_checkable`。

| 协议 | 成员 |
|---|---|
| `Codec` | `meta`；`supports(suffix) -> bool`；`decode(path, limits) -> AudioSegment` |
| `FeatureExtractor` | `meta`；`compute(ctx: AnalysisContext) -> FeatureResult` |
| `Reporter` | `meta`；`render(result: AnalysisResult) -> str` |

插件不得持有可变共享状态；`compute` 必须只读 `ctx`。插件不接触任何计算后端抽象——共享数值能力由 `core/compute.py` 在构建 `AnalysisContext` 时一次性提供，插件只消费产出的上下文。

### 5.3 异常层次

`AudioAnalyzerError` 为基类，派生 `DecodeError` / `PluginError` / `ConfigError`。`PluginError` 携带插件名与失败阶段，供 `core/errors.py` 映射为降级记录与 `warnings` 条目。基类位于 `contracts`，保证插件无需 import `core` 即可抛出框架可识别异常（D-9）。

---

## 6. 数据流

1. **输入**：路径校验（名称白名单、大小与时长上限）→ 计算 `source_hash`。
2. **解码**：`Codec.decode` → `AudioSegment`；校验采样率、声道数与有限值。
3. **预处理**：`core/compute.py` 按 profile 声明执行 mono / resample / normalize，**仅一次**，产出只读 `AnalysisContext`。
4. **编排**：`orchestrator` 依 `PluginMeta.requires` 构建 DAG；`registry` 做环检测与依赖缺失拒绝；查询内容寻址缓存。
5. **计算**：无依赖的插件在线程池并行，上限由 profile 的 `max_workers` 控制；单插件超时或异常 → 降级 `FeatureResult` 并写入 `warnings`，不中断整次分析。
6. **聚合**：生成 `AnalysisResult`，含契约版本、各插件版本、timings 与 warnings。
7. **输出**：`Reporter.render` → CLI（JSON / 表格 / Markdown）。

---

## 7. 错误处理与边界

### 7.1 三级处理

| 级别 | 触发 | 行为 | CLI 退出码 |
|---|---|---|---|
| 输入级 | 文件缺失、编码不支持、路径越界、参数非法 | 结构化错误，终止分析 | 2 |
| 插件级 | 异常、超时、NaN / Inf、契约校验失败、不可序列化值 | 降级 `FeatureResult` + 强制 `warning`，继续其余特征 | 0 |
| 系统级 | `KeyboardInterrupt` / `MemoryError` / `SystemExit` | 透传，不吞 | 130 / 1 / 原样 |

沿用遗留顶层屏障的分级原则：业务异常 2、Ctrl+C 130、`SystemExit` 与其他 `BaseException` 透传。

### 7.2 输入与安全边界

- profile 名称白名单 `^[A-Za-z0-9_-]{1,64}$`；拒绝 `..`、绝对路径越界、符号链接逃逸。
- 文件大小与时长上限由 profile 配置，防解码炸弹与 OOM。
- 解码后校验采样率、声道数、样本有限值。
- 对外错误信息截断 200 字符，防泄漏 traceback、绝对路径与内部状态。
- 缓存序列化使用 JSON + 显式 schema 校验，**不使用 pickle**，从根上消除反序列化 RCE 面。

### 7.3 并发与资源边界

`AnalysisContext` 只读；注册表加载期冻结、运行期只读；缓存写入用临时文件 + 原子 rename，避免并发写坏；单插件超时只中断该插件；`max_workers` 受限，插件不得自建线程池；超长文件首版明确拒绝并给出可诊断错误，而非实现流式处理。

---

## 8. 测试策略

| 层次 | 覆盖内容 |
|---|---|
| 契约测试 | ast 解析 import-graph 校验 4.3 分层；契约主版本校验；冻结 dataclass 不可变；`FeatureResult.value` 的 JSON 可序列化约束 |
| 插件单元测试 | 每个插件的正常路径、边界路径（空 / 静音 / 单样本 / 多声道）、异常路径（抛错 / 超时 / NaN） |
| 编排集成测试 | DAG 环检测、依赖缺失拒绝、降级与 warning 必达、超时隔离、**缓存键内容寻址回归**：同一路径被原地覆写为不同内容后必须得到不同结果，不得返回旧缓存 |
| 金标语料回归 | 以 `audio-test/` 20 首 flac 建立特征基线快照，检测数值漂移；标记 `slow`，不进默认 CI |

缓存内容寻址回归用例直接对应已丢失审计文档中的 R6-01 教训（旧实现以 path + size 为键，原地覆写会静默返回错误结果），是本规格中最重要的单条回归测试。

---

## 9. 配置与工具链变更

1. `[project.scripts]` 改为 `audio-analyzer = "audio_analyzer.adapters.cli:main"`。
2. `[tool.pytest.ini_options] testpaths=["tests"]`：在 `tests/` 建立之前需临时移除该配置或豁免对应警告，否则 `filterwarnings=["error"]` 会让 pytest 无法启动（见 2.3）；测试目录建立后恢复。
3. 删除 `[tool.ruff] extend-exclude = ["scripts"]`。
4. 删除 mypy 中 `audio_analyzer.backend.cuda` 与 `audio_analyzer.backend._numba_kernels` 两条 override；保留对实际使用的第三方库的 `ignore_missing_imports`。
5. 删除 `[project.optional-dependencies].gpu`（`cupy-cuda11x`）；`uv.lock` 在下次 `uv lock` 时同步。
6. **保留** `numba` / `llvmlite` 显式声明：实测 `librosa 0.11` 硬依赖 `numba>=0.51.0`，移除会破坏依赖解析（见 1.2 第 2 条）。
7. `configs/*.yaml`：`features.enabled` 改为 `plugins.enabled`（D-4）；profile 只声明启用的插件名，不再枚举特征实现。
8. `README.md` / `CONTEXT.md`：同步为纯 CPU 平台描述，移除 v5.8.0 全功能描述与 CUDA / CuPy / GTX 950 / TDR 等已失效的硬约束。

---

## 10. 已知风险

1. 目录无 git 仓库、无备份；本规格无法按 brainstorming 流程提交 commit，且后续所有改动不可回滚。
2. 已删除审计文档中的工程教训仅部分可恢复；本规格显式继承 6 条不变量（内容寻址缓存、失败必留痕、profile 白名单、错误信息截断、禁用 pickle、共享上下文只读），其余若有遗漏只能在金标语料回归阶段发现。
3. `audio-test/` 596 MB 是唯一不可再生的用户资产，金标基线必须只读使用，不得被任何测试写入或覆写。
4. `librosa` 在 cache miss 路径会发 `ComplexWarning`；`filterwarnings=["error"]` 下必须保留现有豁免，否则测试误报。
5. 契约层不用 Pydantic（D-5）失去运行时类型校验，`FeatureResult.value` 的类型与有限值约束只能靠契约测试保障。
6. 纯 CPU 下大文件分析耗时未知，需以 `slow` 标记的基准测试量化，不纳入首版验收范围。
7. 线程池并行可能影响浮点末位一致性，AC-1 已改用 `1e-9` 相对容差；若实测仍不稳定，需固定 BLAS 线程数后重新评估。

---

## 11. 验证命令

工具链版本已实测，命令在仓库根目录执行：

```bash
.venv/Scripts/ruff.exe check src tests
.venv/Scripts/python.exe -m mypy src
.venv/Scripts/python.exe -m pytest -q
```

上述命令在 2.3 的 P0 阻塞项修复、且 `tests/` 目录建立后方可通过。

---

## 12. 实施顺序约束（非实施计划）

依赖顺序：契约层 → 契约测试 → 核心层（注册表 / 编排 / 缓存）→ 内置插件 → CLI 适配器 → 配置与门禁修复 → 金标语料基线。

详细任务拆分、里程碑与验收检查点由后续 writing-plans 阶段生成，本规格不承担计划职责。
