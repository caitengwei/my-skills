---
name: codebase-analyzer
description: Use when asked to analyze a codebase, generate project documentation, understand architecture, or create CODEBASE_KNOWLEDGE files for a repository
---

# Codebase Analyzer

## Overview

探索当前代码库，生成结构化知识文档，供开发者或 LLM 实现功能、修复 bug、安全重构。输出保存到 `codebase-analysis-docs/`。

## Workflow

**阶段 1 探索：** 目录结构 → README/配置文件 → 技术栈 → 核心模块清单

**阶段 2 规划：** 按代码量决定文档数量

| 规模 | 文档数 | 执行方式 |
|------|--------|----------|
| <1 万行 | 1–2 个 | 单 agent 完成 |
| 1–10 万行 | 3–5 个 | 单 agent 完成 |
| >10 万行 | 5–10 个 | 主 agent 规划，按模块边界拆分，用 `superpowers:dispatching-parallel-agents` 并行分析各模块后整合 |

**阶段 3 编写：** 按需读取源文件，提取关键代码片段，创建架构图（Mermaid）

**阶段 4 校验：**
- [ ] 文件路径引用正确（相对路径）
- [ ] 代码示例可运行
- [ ] 术语一致，无断链

## 文档模板（CODEBASE_KNOWLEDGE.md）

```markdown
## 1. 项目概述 — 目的、技术栈、主要功能
## 2. 架构设计 — 架构图、组件关系、数据流
## 3. 核心模块 — 模块说明、关键类/函数、代码示例
## 4. 使用指南 — 安装配置、API 使用、典型用例
## 5. 开发指南 — 扩展方法、调试技巧、常见陷阱
```

## 大型代码库策略（>10 万行）

主 agent 负责阶段 1–2（探索 + 规划），识别模块边界后按以下方式拆分并行任务：

**Dispatch 前：主 agent 须为每个 subagent 构造完整上下文**（不让 subagent 重新探索整个仓库）：
- 该模块的目录路径和入口文件列表
- 已知的技术栈和依赖关系
- 整体架构中该模块的定位

**每个 subagent 的任务结构：**
- **Scope：** 一个明确的模块或子目录
- **Goal：** 生成该模块的 `MODULE_X.md`，包含组件说明、关键接口、依赖关系
- **Constraints：** 只分析指定目录，不读取其他模块
- **Output：** 返回文件路径 + 简要摘要（供主 agent 整合用）

**主 agent 整合步骤（收到所有 subagent 结果后）：**
1. 检查各模块文档的术语一致性
2. 补充模块间的交叉引用
3. 编写顶层 `CODEBASE_KNOWLEDGE.md` 和 `README.md` 索引

## 文件优先级

```
高：README、CHANGELOG、入口文件、核心模块
中：配置文件、测试文件、工具脚本
低：文档网站、示例代码、CI/CD 脚本
```

分层阅读：入口/配置 → 核心模块 → 关键实现细节。理解 80% 核心逻辑即可。

## 输出结构

**必需：**
- `codebase-analysis-docs/CODEBASE_KNOWLEDGE.md`
- `codebase-analysis-docs/README.md`（文档导航）

**可选（按项目特点）：**
`API_REFERENCE.md` / `ARCHITECTURE.md` / `DEPLOYMENT.md` / `TESTING.md` / `PERFORMANCE.md`

## Integration

产出文档可作为 `superpowers:requesting-code-review` 的 reviewer context 传入，帮助 code-reviewer subagent 快速建立代码库认知。
