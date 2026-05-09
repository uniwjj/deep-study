---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-sourcemap-reverse]
---

## 终于把Claude Code本地跑起来了，看看缺少哪些文件，如何修复

**文章信息**：
- **原文标题**：终于把Claude Code本地跑起来了，看看缺少哪些文件，如何修复
- **来源**：https://www.toutiao.com/article/7623429393402102278
- **作者**：VibeCoder
- **发表时间**：2026-03-31 22:41

---

**一段话总结**：

作者通过解析 Anthropic 发布 npm 包时意外携带的 sourcemap 文件（cli.js.map），还原了 Claude Code 几乎全部 1,884 个 TypeScript 源文件（约 51 万行代码）。还原后的代码面临构建配置缺失、编译时 API 替换、内部包不公开、React 版本兼容等 7 类核心问题。作者采用三层降级策略——完整恢复（直接复制 sourcemap 中包含的 SDK 源码）、运行时 Stub（为空缺模块创建最小空实现）、Feature Flag 禁用（用运行时 Set 替代 78 个编译时 feature flag）——最终实现约 95% 的代码还原。缺失的 5% 主要是上下文管理服务（Context Collapse、Snip Compact、Cached Microcompact）和沙箱隔离，导致长对话（50 轮以上）体验退化，且所有 shell 命令直接在宿主机执行。最大的坑是 React 版本——必须用 React 19 canary 才能正确渲染 Ink TUI 界面。开源仓库：https://github.com/zxdxjtu/claude-code-sourcemap

---

**作者判断**：

作者认为 sourcemap 还原是一个可行的技术路线，95% 还原率已足够覆盖核心功能。缺失部分对短对话基本无感，但长对话和安全性有明显退化。作者坦诚花费最多时间在 React 版本兼容上，因为"编译通过、程序启动、进程在跑，就是屏幕什么都不显示"这一表现极具迷惑性。

---

**核心内容**：

**Sourcemap 是怎么回事**：Anthropic 发布 npm 包时把 cli.js.map 一起带上了。用 Node.js 的 source-map 库解析后，1,884 个原始 TypeScript 源文件全部还原，覆盖 Claude Code 几乎所有核心模块：对话引擎、45 个以上的工具实现、权限安全系统、MCP 集成、多 Agent 协作、React 终端 UI。

**还原后面临的 7 类核心问题**：

1. **没有构建配置**：sourcemap 只还原 .ts 源文件，不带 package.json 或 tsconfig.json。需手动分析所有 import 语句找出 84 个 npm 依赖。
2. **编译时 API 运行时不存在**：Claude Code 用 Bun 的 `bun:bundle` 模块提供 `feature()` 函数，在编译阶段判断 78 个功能开关并做 Dead Code Elimination。还原后这些函数在运行时不存在。
3. **321 个文件的 import 路径断裂**：代码用 `from 'src/services/...'` 绝对路径导入，还原后目录结构变了，路径全部对不上。
4. **25 个内部源文件缺失**：部分被 DCE 移除，部分是 Anthropic 内部模块，不在 sourcemap 映射范围内。
5. **8 个 Anthropic 内部 npm 包不公开**：包括 `@anthropic-ai/sandbox-runtime`（沙箱运行时）、`@ant/claude-for-chrome-mcp`（Chrome 浏览器集成）、`@ant/computer-use-mcp`（Computer Use 控制）等。
6. **2 个 C++ 原生模块**：`color-diff-napi`（颜色差异计算）和 `modifiers-napi`（键盘修饰键检测）的源码不在 JavaScript sourcemap 中。
7. **React 版本问题**：代码使用 React 19 canary 的 API，包括 React Compiler 运行时（499 处调用）和 `useEffectEvent`（仅 canary 版本有）。

**三层降级策略**：

- **第一层·完整恢复**：sourcemap 中包含 `@anthropic-ai/bedrock-sdk`、`@anthropic-ai/vertex-sdk`、`@anthropic-ai/foundry-sdk` 三个 SDK 的源码，直接复制到 node_modules 即可。
- **第二层·运行时 Stub**：为 25 个缺失源文件创建最小空实现——工具缺源码则 `isEnabled()` 返回 false；上下文压缩服务缺源码则处理函数原样返回消息；UI 组件缺源码则在 `useEffect` 里立即调用 `onCancel()`。8 个内部 npm 包统一重定向到 `native-stubs.ts`，`SandboxManager.isSupportedPlatform()` 返回 false 走非沙箱分支。
- **第三层·Feature Flag 禁用**：用运行时 `Set.has()` 替代 78 个编译时 feature flag，约 50 个核心功能启用（`FORK_SUBAGENT`、`MCP_TOOL`、`PROMPT_CACHE`），约 28 个内部/实验性功能禁用（`KAIROS`、`COORDINATOR_MODE`、`DAEMON`、`BUDDY`）。

构建系统用 4 个 Bun Bundler 插件实现替换（拦截 `bun:bundle` 导入、拦截 react/compiler-runtime、拦截 8 个内部包、把 .md/.txt 文件作为字符串导入），最终输出约 21.6 MB 的单文件 `dist/cli.js`。

**跑不了的功能**：

- **Context Collapse（上下文折叠）**：旧对话段无法自动折叠为摘要释放 token 空间。
- **Snip Compact（历史裁剪）**：对话历史无法自动裁剪旧消息，token 累积更快，更早触发上下文窗口溢出。
- **Cached Microcompact（缓存感知微压缩）**：Anthropic API 的 prompt cache 编辑优化不可用，每次压缩后缓存命中率下降。
- 三个服务一起失效 → 长对话（50 轮以上）体验退化明显，短对话基本无感。
- **沙箱隔离完全缺失**：正式版中 shell 命令在沙箱内运行限制文件系统和网络访问，还原版所有命令直接在宿主机执行，权限审批系统在但缺少底层强制执行层。
- Coordinator Mode（多 Worker 协调模式）源码完整（370 行）但通过 feature flag 禁用。

**React 版本：最大的坑**：

构建成功后程序"卡住"——屏幕空白无输出。排查发现执行到了 `createRoot`，Ink 框架初始化了但 React 组件渲染结果为空。React 18 + reconciler 0.29 报错 `ReactSharedInternals.S is undefined`；React 19 正式版报错 `useEffectEvent is not a function`。最终换成 React 19 canary（19.3.0-canary-74568e86-20260328）+ reconciler canary 版本才正确渲染。这件事花了最多时间，因为表现极具迷惑性：编译通过、程序启动、进程在跑，就是屏幕什么都不显示。

**还原率**：整体约 95%。1,884 个文件、51 万行代码、84 个外部依赖包全部到位。对原始源码只做了 2 处修改：Commander.js 短选项格式修复、非 TTY 环境检测逻辑增加环境变量绕过。核心对话引擎、45+ 工具系统、Ink TUI 框架、权限系统、MCP 集成、Agent/Swarm 协作、记忆系统、配置管理、提示词系统全部完整可用。
