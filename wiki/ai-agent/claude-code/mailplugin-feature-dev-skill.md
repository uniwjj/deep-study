---
title: 邮件插件需求开发 Skill（94% AI 代码生成率）
description: 企业微信团队基于 Claude Code 构建的 iOS/macOS 邮件插件功能开发 Skill，8 阶段流水线，代码生成率 94%
aliases: [mailplugin-feature-dev, WeCom Mail Plugin Feature Dev Skill, 企业微信邮件插件 Skill]
tags: [ai-agent, practice, tool]
sources: [2026/07/20/AI代码生成率94%：我们用一个Skill跑通需求开发全流程.html]
created: 2026-07-21
updated: 2026-07-21
---

# 邮件插件需求开发 Skill（94% AI 代码生成率）

企业微信团队（gomezlai）基于 Claude Code 构建的 iOS/macOS 邮件插件功能开发 Skill，将需求开发拆成 8 个语义化阶段，流程化、原子化、可校验化，实现 **94% AI 代码生成率**。

## 问题背景

AI 辅助编码在企业级真实项目中的典型翻车场景：

| 痛点 | 现象 |
|------|------|
| 上下文塞不下 | 9000+ 源文件、跨 5~6 层调用链，单轮对话喂不进去 |
| 物料分散 | PRD 在 TAPD、设计稿在 Figma、协议在企微文档、UI Token 命名不一致 |
| 模糊指令 | 用户一句"按 PRD 改一下"，AI 跳过拆解直接改代码，越界、漏改、改错位 |
| 验证不闭环 | AI 报"完成"，结果编译都没过；改完一处没顾上同步另一处 |
| 跨会话失忆 | 上一次的设计决策、改了什么文件、为什么这么改，下一次会话全忘 |

核心诊断：**AI 不是不会写代码，是不会"按工程规范"开发需求**。解法不是换更大的模型，而是把需求开发流程化、原子化、可校验化。

## 整体架构：8 阶段流水线

Skill 内部采用「阶段·动作」式命名（如 `设计稿·脚本筛选`、`实现·UI·切图`、`拆解·TAPD收料`），让 AI 自报家门时永远清楚自己在哪一格。

| 阶段 | 输入 | 关键产出 | 灵魂动作 |
|------|------|---------|---------|
| ① 设计稿 | Figma 链接 | 移动端候选稿清单 + PNG 概览 | 脚本化直方图筛选，绝不允许 LLM "手感"分桶 |
| ② 拆解 | PRD + 设计稿 + CGI + TAPD | 五列需求清单 + `subtasks.json` 接力台账 | 多源收料 + 归宿校验（每张设计稿必须归到三类之一） |
| ③ 定位 | 需求点 | 文件 + 行号 + 调用链 | [[code-knowledge-base-pyramid|五步定位法]] |
| ④ 实现 | 调用链 + 上下文 | 代码改动 | 自底向上：数据 → 解析 → 枚举 → 业务 → UI → 日志 |
| ⑤ 验证 | 源码改动 | 编译报告（退出码 0） | `bazel build` + 最多 3 轮自修复 |
| ⑥ 模拟器验证 | 编译通过的产物 | 装机截图 + 日志 | 人机秒级确认 + 阶段内重试 ≤ 2 轮 |
| ⑦ 沉淀 | git diff + 时间线 | `TECH_SPEC.md` 单一事实源 | 跨会话知识传承的载体 |
| ⑧ 提交 | 全部产物 | git commit + 分支 | 三段式 commit + AI 署名 + 代码生成率 |

后一阶段的输入严格等于前一阶段的产出，不允许跳过。

## 四大设计公理

整个 Skill 围绕一个目标：**让一个没参与过原始实现的 AI，在新会话里像"参与过的老同事"一样把活干完**。

| 公理 | 核心思想 | 详见 |
|------|---------|------|
| Ⅰ 每一步都在缩小范围 | [[code-knowledge-base-pyramid|五步定位法]]：从 ~10M+ token 压缩到 ~30K（300×） | 第五节 |
| Ⅱ 判断交给 LLM，数据交给脚本 | LLM 不擅长精确数值和幂等执行，全部下沉到脚本 | 第七节 |
| Ⅲ 红线前置 | [[requirement-semantic-translation#红线机制|YAML 红线系统]]，触发即停、模板化报告 | 第八节 |
| Ⅳ 跨会话知识传承 | `TECH_SPEC.md` 单一事实源 + 4 类入口自动分流 | 第十节 |

## 代码知识库：三级金字塔

参见 [[code-knowledge-base-pyramid]]。

### 三级结构

| 级别 | 文件 | 粒度 | 加载时机 |
|------|------|------|---------|
| L1 总览 | `overview.md` | 模块名 + 一句话职责 | 定位阶段默认 preload（< 5KB） |
| L2 模块 | `<module>.md` | 每个 .h/.mm 文件 + 功能说明 | 命中模块后按需加载 |
| L3 语义桥 | `figma_token_mapping.md` / `ui_components_wiki.md` | Figma Token → 工程 API 精确映射 | 实现·UI 阶段强制参考 |

### 自维护机制

核心脚本 `check_project_wiki_stale.py`：
- **SHA 基线缓存**：记录每个文件上次审阅时的 SHA，再次变化自动 flag
- **三色分诊清单**：新增/删除/大改三类信号分开列
- **pre-commit hook 阻断**：退出码 1 = 有 stale 信号 → 阻止提交

## 需求语义翻译

参见 [[requirement-semantic-translation]]。

五个确定性规则逐层堵住"产品语言 → 代码指令"的翻译翻车点：

1. **范围识别**：硬关键词表触发，不依赖 LLM 语义判断
2. **设计稿归宿**：每张图必须归到三类之一，禁止"参考图"万能垃圾桶
3. **拦截点清单**：禁止语义联想扩大范围，依据只接受设计稿标注/TAPD原文/用户原话
4. **领域联想**：[[requirement-semantic-translation#五维搜索矩阵|5 维搜索矩阵]]（iOS 事件方法 / 功能语义 / OC 命名习惯 / 协议代理 / 通知回调）
5. **结构化产出**：五列表格 + `subtasks.json` 机器可读台账

## 运行时验证

### 闸门一：编译验证

`bazel build` 退出码 0 是唯一判据。自修复硬上限 3 轮，超限强制停下。

A/B 分类：A 类可自修复（语法错误等）→ 直接修；B 类需用户介入（链接错误等）→ 立即停下。

### 闸门二：模拟器验证

五步流程：
1. **路径推导**：从 git diff 反推 UI 验证路径
2. **UI 路径预扫描**：6 步反向溯源（代码方法 → UI 控件）
3. **执行**：每步固定 5 个动作，实时汇报
4. **A/B/C 诊断**：A 真问题（回实现阶段）/ B 路径不通（修订验证计划）/ C 脚本时序（阶段内重试 ≤ 2 轮）
5. **视觉对齐核对**：逐项核对数值，未对齐 ≥ 1 → FAIL

## 提效效果

| 环节 | 传统方式 | Skill 方式 |
|------|---------|-----------|
| 需求拆解 | 1~2 小时 | 5~10 分钟 |
| 代码定位 | 30 分钟~2 小时 | 5~15 分钟 |
| 编译自查 | 手动来回 | 自动 3 轮 |
| UI 验证 | 手动装机点击 + 肉眼比对 | 自动装机 + 截图 + A/B/C 诊断 |
| Bug 修复接力 | 重新读代码 1+ 小时 | 5 分钟恢复现场 |
| 提交规范 | 手写 commit | 三段自动渲染 + 人工确认 |

最大隐性收益：**新人/AI 都能直接接手已有需求的迭代**，不再依赖"问原作者"。

## 关键启示

5 条普适原则：

1. **流水线化**：拆成语义化阶段，每阶段输入/输出/退出标准可机器校验
2. **脚本兜底**：LLM 负责"读判断"，精确数值/幂等执行下沉到脚本
3. **红线前置**：YAML + 分层加载，触发即停、模板化报告
4. **落盘判定**：任何长跑命令的成功证据都是"文件存在"，不依赖 stdout
5. **沉淀闭环**：每次需求产出一份 git-tracked 的 `TECH_SPEC.md`

## Skill 目录速览

```
skills/mailplugin-feature-dev/
├── SKILL.md / README.md / CHANGELOG.md     # 对外入口
├── setup/                                    # 安装与配置
├── tools/                                    # 自动化脚本（30+ 脚本）
│   ├── fetch_tapd_story.py                   # TAPD 收料
│   ├── fetch_figma_mcp.py                    # Figma MCP 数据落盘
│   ├── scan_figma_frames.py                  # 设计稿直方图筛选
│   ├── build_verify.sh                       # bazel 编译 + 报告
│   ├── check_project_wiki_stale.py           # 知识库时效性扫描
│   ├── install_to_simulator.sh               # 模拟器安装
│   └── render_tech_spec.py                   # TECH_SPEC 渲染
├── ref/                                      # 知识库 + 语义桥
│   ├── project_wiki/                         # L1/L2 代码地图
│   ├── glossary.md                           # 项目命名约定
│   ├── figma_token_mapping.md                # L3 语义桥
│   └── ui_components_wiki.md                 # UI 组件索引
├── red_lines/                                # 红线系统
│   ├── critical.yaml                         # Critical 红线（6 条）
│   └── standard.yaml                         # Standard 红线（30+ 条）
└── templates/                                # 产物模板
```

## 相关页面

- [[code-knowledge-base-pyramid]] — 三级金字塔代码知识库 + 五步定位法
- [[requirement-semantic-translation]] — 产品语言→代码指令的语义翻译体系
- [[claude-code-harness]] — Claude Code Harness 架构
- [[dw-harness-practice]] — 数仓 Harness 工程落地方案（类似的 Skill 流水线实践）
- [[agent-skills-system]] — Agent Skills 系统机制
- [[agent-hook-governance]] — Agent Hook 治理（红线机制的同类思路）
- [[superpowers-framework]] — Superpowers 工程纪律框架（类似的流水线化 + 红线设计）
- [[agent-memory-system]] — Agent 记忆系统（与跨会话知识传承相关）
