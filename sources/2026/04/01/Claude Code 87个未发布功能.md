---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-hidden-features]
---

## Claude Code 泄露代码隐藏的87个未发布功能 暴露其野心...

**文章信息**：
- **原文标题**：Claude Code 泄露代码隐藏的87个未发布功能 暴露其野心...
- **来源**：https://www.toutiao.com/article/7623628185803964954
- **作者**：小互AI
- **发表时间**：2026-04-01 11:31

---

**一段话总结**：

Claude Code CLI 的 npm 包意外泄露了完整 source maps（1884 文件，512,664 行 TypeScript，33MB），作者从中提取出 87 个编译时 feature flag，大部分处于关闭状态。这些 flag 指向一条进化路线：工具 → 常驻助手 → 团队协作 → 跨设备 → 多感官。核心功能包括 Kairos 主动助手模式（24小时在线、自主找活干）、Auto-Dream 记忆整理（后台定位→采集→整合→修剪四阶段）、Coordinator 多 Agent 编排系统（Research→Synthesis→Implementation→Verification 四阶段）、Teleport 跨设备会话迁移、Voice 语音输入、Buddy 虚拟宠物彩蛋等。代码已完成，非原型，随时可上线。

---

**作者判断**：

作者认为 Anthropic 在做减法——去掉 Agent 身上的每一个限制，直到它不再像一个工具。源码里的提示词已经不再叫它"tool"，而是"colleague"。结论：**Claude Code 现在的样子不是它最终的样子，它想变成的东西源码里已经写好了**。作者最直接的感受是："从'你问它答'到'它自己找活干'，再到'它按时间表自动干'——这不是增量升级，是换了个物种。"

---

**核心内容**：

**87 个 feature flag 总览**（通过 `grep -o "feature('[A-Z_]*')" | sort -u` 提取）：
ABULATION_BASELINE, AGENT_MEMORY_SNAPSHOT, AGENT_TRIGGERS, BUDDY, BUILDING_CLAUDE_APPS, COORDINATOR_MODE, DAEMON, DIRECT_CONNECT, EXTRACT_MEMORIES, KAIROS, KAIROS_BRIEF, KAIROS_CHANNELS, KAIROS_DREAM, KAIROS_GITHUB_WEBHOOKS, KAIROS_PUSH_NOTIFICATION, PROACTIVE, SSH_REMOTE, TEAMMEM, TELEPORT, UDS_INBOX, ULTRAPLAN, ULTRATHINK, VOICE_MODE, VERIFICATION_AGENT, WEB_BROWSER_TOOL, WORKFLOW_SCRIPTS 等。

**一、Kairos + Daemon：去掉"你不问它不动"**

- **Kairos**（内部名"assistant mode"）：激活后 Claude Code 变成 24 小时在线的主动助手。系统发周期性心跳 tick，每收到一个 tick 评估"有无可做之事"，有就做，没有就调 Sleep 工具休眠
- 系统提示词定位为"colleague"（同事），不是"tool"或"assistant"。原文提示："好的同事面对模糊性不会停下来——他们会调查、降低风险、加深理解"
- 检测终端是否聚焦：失焦=你不在=更大胆自主行动；聚焦=你在看=更多协商
- Kairos 专属工具：SleepTool（休眠）、SendUserMessage（主动发消息）、SendUserFile（发文件）、PushNotification（手机推送）、SubscribePR（订阅 GitHub PR webhook）、CronCreate（创建定时任务）
- 目前锁在 `tengu_kairos` gate 后面，仅内部可用，但代码已是成品
- **Daemon** 是 Kairos 底座：四种会话类型 interactive / bg / daemon / daemon-worker。Daemon 模式无 TUI 界面，纯 headless 常驻进程，内置 cron 调度器（标准 5 字段表达式、本地时区、jitter 防全球同时触发、持久化到 `.claude/scheduled_tasks.json`）
- 还有远程 scheduled agents——在 Anthropic 云端跑完全隔离的 Claude Code 实例，通过 `/schedule` 命令管理，最小间隔 1 小时

**二、Auto-Dream：去掉"下次要重新教"**

- 后台自动整理记忆，源码里真的叫"dream"
- 触发条件：距上次整理 >24 小时 + 至少 5 个新会话 + 无其他进程在整理（PID 锁防并发）
- 四阶段：Orient（读 memory 目录索引）→ Gather recent signal（从日志/现有记忆/session transcripts 收集）→ Consolidate（写入/更新、合并重复、日期转绝对、删除已推翻事实）→ Prune and index（更新 MEMORY.md，保持 200 行 / 25KB 以内）
- 安全限制：Dreaming Agent 的 Bash 只能执行只读命令（ls/find/grep/cat），Edit 和 Write 只能操作 memory 目录内文件
- 已在灰度，设置里有 Auto-dream: on/off 开关
- Kairos 模式下 auto-dream 被禁用，改用更精细的 cron 定时 dream。普通模式用 auto-dream

**三、Coordinator + UDS Inbox + Ultrareview：去掉"只能一个人干"**

- **Coordinator 模式**：完整的多 Agent 编排系统。协调者只有三个权限：派活（Agent 工具）、传话（SendMessage）、叫停（TaskStop）。四阶段工作流：Research（Workers 并行调查）→ Synthesis（协调者自己消化制定方案）→ Implementation（Workers 按方案修改）→ Verification（Workers 测试验证）
- 反偷懒设计：禁止写"based on your findings"——协调者必须自己消化信息，给 worker 精确到行号的指令
- **UDS Inbox**：同机多个 Claude Code 实例通过 Unix Domain Socket 互相通信。启动时创建 socket 写入 `~/.claude/sessions/`，支持按名字互发消息和广播（`to: "*"`）
- **Ultrareview**：在 Anthropic 云端启动 5-20 个 Agent（默认 5，最多 20），每个从不同角度审查代码。总耗时 ≤22 分钟（预留 3 分钟综合），单 Agent ≤600 秒。两种模式：PR 模式（`/ultrareview 123`）和分支模式（打包本地工作区上传）

**四、Teleport + Ultraplan：去掉"绑定在一台机器"**

- **Teleport**：`--remote` 把本地会话传到云端（检测 Git 仓库直接 clone 或打包 git bundle ≤100MB 上传），`--teleport` 从云端拉回（拉对话记录、replay、checkout 同一分支、自动 stash 脏工作区）
- **Ultraplan**：本地输入 `/ultraplan`，系统在云端启动远程 Claude Code（Opus 4.6 模型），最多 30 分钟探索项目、分析可行性、规划步骤。"ultraplan" 这个词会触发彩虹色高亮，但出现在代码块/引号/文件路径里不触发

**五、Voice + Chrome 控制：去掉"只能打字"**

- **Voice**：按住说话，实时语音转文字。三层录音后端降级：原生 NAPI 模块 → arecord（Linux ALSA）→ SoX rec。16kHz/16 位/单声道 PCM。通过 WebSocket 流到 Anthropic STT（Deepgram Nova 3），支持 20 种语言（暂无中文）。自动提取项目领域词汇（Git 分支名、文件名、技术术语）作为 keyterms 提高识别准确率，最多 50 个
- **Chrome 浏览器控制**：通过 Chrome 扩展在用户现有 Chrome 中操作（非无头浏览器），能导航/点击/填表/截图/录 GIF/读 console log/network 请求

**六、Buddy：彩蛋——去掉无聊**

- 虚拟宠物，住在终端里，有 ASCII 精灵图和语音气泡
- 18 种物种（duck, axolotl, capybara, dragon, chonk, goose, blob, cat, octopus, owl, penguin, turtle, snail, ghost, cactus, robot, rabbit, mushroom）
- 五档稀有度：common 60%、uncommon 25%、rare 10%、epic 4%、legendary 1%。1% 概率闪光版
- 5 项属性：DEBUGGING, PATIENCE, CHAOS, WISDOM, SNARK
- 反作弊：由用户 ID + 盐值 `friend-2026-401`（愚人节彩蛋）做 FNV-1a 哈希 → Mulberry32 伪随机，结果完全确定
- 只有"灵魂"存在配置文件，"骨骼"从哈希重算，无法伪造
- 发布时间窗：2026 年 4 月 1-7 日，24 小时滚动覆盖全球时区

