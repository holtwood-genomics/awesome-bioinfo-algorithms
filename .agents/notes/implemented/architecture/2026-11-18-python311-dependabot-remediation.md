# Agent Note: Python 3.11 minimum for secure dependency resolution

Status: implemented

## Problem

`requirements-dev.txt` 的 `pre-commit`/`virtualenv` 传递依赖在 Python 3.9 和 3.10 上锁定到受影响的 `filelock`，产生 17 项 Dependabot 告警。修复版 `filelock 4.2.1` 起要求 Python 3.11，无法在保持 3.9/3.10 支持的同时完成安全升级。

## Decision

最低支持版本提升到 Python 3.11，CI 只验证 3.11/3.12，并使用 `--python-version 3.11 --universal` 重建带哈希锁。`README.md` 由模板同步生成，避免支持声明漂移。

## Alternatives considered

- 保留 3.9/3.10 并 dismiss 告警：这些是实际可执行依赖，关闭告警会掩盖风险。
- 仅升级 `filelock`：上游版本元数据明确要求 Python 3.11，锁解析无法在旧解释器上成立。
- 为旧版本维护单独锁文件：增加双锁与双矩阵维护成本，不符合本项目轻量维护取向。

## Consequences

- 使用 Python 3.9/3.10 的贡献者需要升级解释器。
- 锁文件去掉旧 Python 分支，安全更新可以正常解析。
- CI 运行面收敛为 3.11/3.12。

## Verification

- `uv pip compile pyproject.toml --extra dev --generate-hashes --universal --python-version 3.11 -o requirements-dev.txt`
- `pip install --require-hashes -r requirements-dev.txt`
- `ruff format --check awesome_bioinfo/ tests/`
- `ruff check awesome_bioinfo/ tests/`
- `pytest tests/ -v --tb=short`
- `python -m awesome_bioinfo generate && git diff --exit-code -- README.md`
