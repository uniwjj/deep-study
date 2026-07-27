---
title: 代码知识库金字塔（三级 + 五步定位法）
description: 企业微信团队的三级金字塔代码知识库设计——L1 总览/L2 模块/L3 语义桥，配合五步定位法实现 300× token 压缩
aliases: [Code Knowledge Base Pyramid, 三级金字塔知识库, project_wiki]
tags: [ai-agent, practice, concept]
sources: [2026/07/20/AI代码生成率94%：我们用一个Skill跑通需求开发全流程.html]
created: 2026-07-21
updated: 2026-07-21
---

# 代码知识库金字塔

企业微信团队为让 AI 在 9000+ 源文件的大型 iOS/macOS 项目中精准定位改动点，构建了一套 **三级金字塔代码知识库** + **五步定位法**，将 token 消耗从 ~10M+ 压缩到 ~30K（**300× 压缩比**）。

## 三级金字塔

### L1：项目总览（`overview.md`）

AI 入场的"大堂导览"。一张表告诉 AI "项目有哪些模块、各自负责什么"。控制在 **5KB 以内**，可无负担地塞进每次定位上下文。

```markdown
| 模块    | 职责                           | 详细文档  |
|---------|-------------------------------|-----------|
| MList/  | 邮件列表展示、同步、过滤、多选编辑 | mlist.md  |
| RMail/  | 邮件正文渲染、附件预览、AI 总结   | rmail.md  |
| CMail/  | 邮件撰写、富文本编辑、附件上传    | cmail.md  |
| Model/  | 领域模型 + DB 持久化 + 业务管理器 | model.md  |
```

### L2：模块级（`<module>.md`）

文件粒度的"街道地图"。顶部有机器可读的元数据注释：

```markdown
<!-- module_id: mlist -->
<!-- root_dirs:
  - App/Mailbox/MList/ -->
<!-- desc: 邮件列表展示、同步、过滤、多选编辑 -->
```

按 Controller / ViewModel / View / Helper / Lab 分组，每个文件一行职责说明。一次读 ~70 行就能建立整个模块的拓扑认知。

### L3：领域语义桥

抹平"设计/协议"和"代码"的鸿沟。例如 Figma Token → 工程 API 的精确映射：

```objc
// ❌ 错误：目测字号 + 硬编码颜色
self.titleLabel.font = [UIFont systemFontOfSize:15];
self.titleLabel.textColor = [UIColor colorWithRed:0.1 green:0.1 blue:0.1 alpha:1.0];

// ✅ 正确：按映射规则翻译 Figma Token
self.titleLabel = [UILabel xyz_styledLabel:@"callout"];  // Mobile/callout
self.titleLabel.textColor = XYZColor(base_gray_100);    // Base/base_gray_100
```

映射表覆盖：文字样式（title_1 ~ caption_2）、颜色（自动响应 Dark Mode）、按钮组件、阴影/渐变/字体兜底等 **20+ 类规范**。

## 五步定位法

大模型不是搜索引擎，把整个项目 `find .` 丢给它毫无意义。五步漏斗式收敛：

| 步骤 | 给 LLM 的输入量 | 输出 | 工具 |
|------|----------------|------|------|
| 1. 意图消歧 | 项目概述 ~2K + 用户原话 | "目标可能对应 4 种技术解读" | LLM |
| 2. 模块定位 | 目录树 + 解读结果 | 2-3 个候选文件路径 | LLM |
| 3. 关键词搜索 | —（不进 LLM） | 函数声明 + 位置 | `rg` 脚本 |
| 4. 调用链追踪 | 单个文件相关片段 ~10K | 完整调用链 | LLM |
| 5. 验证确认 | 函数实现 ~5K | 最终改动点 + 理由 | LLM |

**关键设计**：前两步只看目录和文件名，第三步才让脚本 grep，第四步才真正读代码。模型从来不会被整个代码库淹死。

### 知识库 + 定位法的完整闭环

```
第 1 步：从 L1 总览 1 秒选出模块（不用 grep）
第 2 步：从 L2 模块 wiki 5 秒锁定文件（不用读源码）
第 3 步：进入文件后 rg 精准搜索（脚本而非 LLM）
第 4 步：只读相关片段（~10K token）
第 5 步：写代码前先查 L3 映射表（杜绝硬编码）
```

总 token 消耗从全项目灌入的 ~10M+ 降到 ~30K——**300× 压缩比**。

## 自维护：让知识库永不过时

核心脚本 `check_project_wiki_stale.py`：

| 机制 | 作用 |
|------|------|
| SHA 基线缓存（`.review_cache.json`） | 记录每个文件上次审阅时的 SHA，再次变化自动 flag "待复核" |
| 三色分诊清单 | 新增/删除/大改三类信号分开列，30 秒扫完 |
| pre-commit hook 阻断 | 退出码 1 = 有 stale 信号 → 阻止提交，强制开发者顺手维护 |
| 元数据驱动 | `overview.md` 和 `<module>.md` 顶部改 `desc`，索引自动跟随 |

效果：在半年净增 200+ 文件、改动 1000+ 处的迭代速度下，**模块 wiki 从未出现"地图和代码脱节"**。

## 副作用：AI 友好 = 新人友好

这套知识库对人类新人同样有用——新同学入职后，直接读 `overview.md` + 几份模块 wiki，半天就能上手改 bug。"AI 友好"和"新人友好"在这里完全统一。

## 相关页面

- [[mailplugin-feature-dev-skill]] — 邮件插件需求开发 Skill 总览
- [[requirement-semantic-translation]] — 产品语言→代码指令的语义翻译体系
- [[claude-code-large-codebase]] — Claude Code 大型代码库处理策略
- [[agent-skills-system]] — Agent Skills 系统机制
- [[dw-harness-practice]] — 数仓 Harness 工程落地方案（类似的知识库 + Skill 实践）
- [[google-context-engineering]] — Google 上下文工程实践
