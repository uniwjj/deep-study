---
ingested: 2026-05-09
wiki_pages: [ai-agent/hermes-agent]
---

# 装完Hermes Agent玩了一圈，我觉得龙虾已死！

**来源**：正正AI杂说  
**时间**：2026-04-10  
**链接**：https://www.toutiao.com/article/7627053952227394057

## 一段话总结

Hermes Agent是Nous Research开源的AI智能体框架，核心创新在于让Agent自己写Skill、自己迭代Skill，实现了Harness Engineering的自动化。与OpenClaw（龙虾）相比，Hermes让Agent自主进化，而非依赖人工编写技能。安装简单，支持Telegram接入，能在VPS上24/7运行。

## 作者判断

作者认为Hermes的设计思路"蛮有意思"，核心卖点是Agent自己进化的能力。但客观指出：如果OpenClaw用得顺手没必要换；想要多渠道平台和现成技能市场，OpenClaw生态更成熟；想要Agent长期进化，Hermes更对口。最终结论是"OpenClaw让每个人都能养一只AI，Hermes想让AI自己养自己"。

## 核心内容

**Hermes vs OpenClaw 架构对比**：

![Hermes Agent与OpenClaw深度对比图](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/20f5c18ed98b4964bbbee415c2f2924f~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669377&x-signature=szO2I3pQYmYYtnBrguyD3oquNUo%3D)

- OpenClaw：人写Skill，社区贡献，Agent调用（ClawHub技能市场）
- Hermes：Agent写Skill，Agent用Skill，Agent改Skill（agentskills.io开放标准）

**安装命令**：
```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
hermes version
hermes doctor
hermes setup
```

![安装Hermes Agent过程截图](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/fb6c9fe126b04f958d6ff32c37194d05~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669377&x-signature=uC24O2n49QN8PW%2BP8VaBktqT4f0%3D)

**Hermes设计拆解**：
- Skills = Agent自己写的AGENTS.md
- 记忆 = Agent自己管的上下文
- 审批 = Agent自己的evaluator边界
- closed learning loop = Harness的自动化反馈循环

**Telegram接入**：搜@BotFather建bot拿token，然后`hermes gateway setup`选Telegram贴token，`hermes gateway`启动。

![Hermes Agent Setup Wizard终端界面](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/0be8959a69014ce09fbed0d57209f60f~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669377&x-signature=6sxblMZiaK2NojU1kTbk%2FwMw%2FJs%3D)

**24/7运行**：
```
hermes gateway install
systemctl status hermes-gateway
```

**结论**：Harness Engineering的终局，可能不是人写Harness，而是Harness自己写自己。

