---
title: Manus 上下文工程
description: Manus 团队六条上下文工程实践——KV 缓存优先、logit masking、文件系统为无限上下文、todo 注意力操控、保留错误历史、行动序列多样性
aliases: [Manus, 上下文工程, Context Engineering]
tags: [ai-agent, concept, practice]
sources: [2026/Harness/Manus上下文工程.md]
created: 2026-05-09
updated: 2026-05-09
---

# Manus 上下文工程

## 核心理念

四次框架重建、面向百万用户的真实迭代得出的六条原则。

## 原则 1：围绕 KV 缓存设计

- 输入/输出 token 比约 **100:1**，预填充远大于解码
- KV 缓存命中与未命中成本差距 **10 倍**
- 三条实践：稳定提示前缀（无时间戳）、上下文只追加、手动标记缓存断点

## 原则 2：Logit Masking 而非动态移除工具

移除工具破坏 KV 缓存。用 logit masking 遮蔽工具索引保持上下文结构不变。

## 原则 3：文件系统作为无限可恢复上下文

文件是外部持久记忆，通过读写边界管理而非全放进上下文。

## 原则 4：Todo 操控注意力

持续复述 todo 列表，将模型注意力引导到当前目标和进展。

## 原则 5：保留错误历史

不删除错误，让模型从失败中学习。"真正的代理行为最明显的指标是错误恢复能力。"

## 原则 6：行动序列多样性

打破少样本陷阱——模型容易机械重复之前的行动模式。

## 相关页面

- [[agent-harness-overview]] — Harness 综述
- [[claude-code-auto-dream]] — 记忆整合
- [[context-engineering]] — 上下文工程
