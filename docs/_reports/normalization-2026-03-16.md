---
doc_id: docs-normalization-report-2026-03-16
title: 文档规范化迁移报告 2026-03-16
concept: repository_document_normalization
topic: reports
created_at: '2026-03-16T00:00:00+08:00'
updated_at: '2026-03-16T00:00:00+08:00'
source_basis:
  - repository_migration_action
time_context: snapshot_2026_03_16
applicability: migration_audit
prompt_version: not_applicable
template_version: report_v1
quality_status: maintained_asset
related_docs:
  - docs/index.md
open_questions:
  - 是否需要为 legacy_migrated_needs_review 文档补一轮质量门禁检查？
---

# 文档规范化迁移报告 2026-03-16

## 迁移动作

- 将用户维护的知识文档从仓库根目录迁入 `docs/{topic}/` 结构
- 为迁移后的文档补充统一 frontmatter
- 修复已知相对链接
- 新增目录索引页 `docs/index.md`

## 本次纳入 canonical docs 的文件

- `learning_new_things_playbook.md` -> `docs/methodology/learning-new-things-playbook.md`
- `cognitive_modeling_playbook.md` -> `docs/methodology/cognitive-modeling-playbook.md`
- `ai-agent-mcp-skill-openclaw-concepts.md` -> `docs/ai-systems/agent-mcp-skill-openclaw-concepts.md`
- `cpp20_coroutine_playbook.md` -> `docs/programming-languages/cpp20-coroutine-playbook.md`
- `virtual_memory_learning_model.md` -> `docs/computer-systems/virtual-memory-learning-model.md`
- `guided_filter_derivation.md` -> `docs/image-processing/guided-filter-derivation.md`
- `fast_guided_filter_derivation.md` -> `docs/image-processing/fast-guided-filter-derivation.md`

## 未纳入本次规范化的文件

- `_bmad/` 下的工作流、模板与 agent 文档
- `_bmad-output/` 下的规划产物
- 任何由 BMAD 系统维护的内部资产

## 状态说明

- `maintained_asset`：方法论或索引类资产，当前保留为规范化后的维护文件
- `legacy_migrated_needs_review`：已迁入标准目录并补齐元数据，但正文尚未按新质量门禁逐项重构
