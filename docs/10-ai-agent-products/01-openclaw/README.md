# 3分钟搞懂OpenClaw

> ⭐⭐中级 · 阅读时间 3分钟 · 更新于 2026-11-17
>
> OpenClaw 是装在自己电脑上、从微信或飞书里派活的开源个人智能体，国内叫它「龙虾」。它本身不是模型，而是连接聊天软件、大模型和你电脑的网关。本文讲清它怎么工作、本地运行为什么不等于数据不出门、一年来踩过的安全坑，以及国内怎么用。

## 一句话定义

**OpenClaw** 是一个开源的**个人智能体**：你把它装在自己的电脑或一台云服务器上，它会一直在后台运行；之后你在微信、飞书、Telegram、Slack 这些聊天软件里给它发消息，它就调用大模型来理解你的意思，再动手操作这台机器，比如跑命令、读写文件、打开浏览器，把事情做完以后把结果发回给你。

它的吉祥物是一只龙虾，名字里的 Claw 就是「钳子」的意思，所以国内用户都叫它「龙虾」，自己装一个用起来叫「养龙虾」。它用 MIT 协议开源，GitHub 星标约 39 万，是 2026 年增长最快的开源项目之一。

举个例子，你可以在飞书里跟它说：「每天早上 8 点，把我今天的日程和昨晚没回的重要邮件整理一下发给我。」它会自己写好一个定时任务，第二天早上 8 点准时在后台醒来，读日历、翻邮箱、整理成一条消息推给你。整个过程你不用打开电脑，也不用打开任何一个 AI 应用。

## OpenClaw 本身不是 AI，而是一个「网关」

很多人第一次听说 OpenClaw，会以为它是一个和 ChatGPT、DeepSeek 同类的新模型。其实它**不训练模型，也不自带模型**。它做的事情是把三样东西连起来：你常用的聊天软件、你选的大模型，以及你的电脑。

负责连接的核心程序叫 **Gateway（网关）**，它在你的机器上常驻运行，一条消息进来以后大致会经过这样一个流程：

```text
你在微信 / 飞书 / Telegram 里发一条消息
        ↓
Gateway 网关 (常驻在你的电脑或服务器上)
  ├─ 读取工作区里的几个 Markdown 文件: 它是谁、你是谁、之前记住了什么
  ├─ 把你的消息和这些内容一起发给大模型 (Claude、GPT、DeepSeek、本地模型……)
  └─ 按模型的决定调用工具: 跑命令、读写文件、操作浏览器、使用技能
        ↓
结果从原来的聊天软件发回给你
```

所以 OpenClaw 好不好用，很大程度上取决于你接的是哪个模型。同样一句「帮我把下载文件夹按类型整理一下」，接一个强模型和接一个弱模型，执行起来的可靠程度会差很多。

![OpenClaw 的网关在中间：左边接微信、飞书、Telegram 的消息，右上把内容交给大模型理解，右下在你的电脑上动手，网关里放着 SOUL.md、USER.md、MEMORY.md 这些文件](images/04-framework-gateway-hub.jpg)

它的「性格」和「记忆」也没有藏在什么数据库里，而是放在工作区的几个普通文本文件里，你可以直接打开来看、来改：

| 文件 | 放什么 |
| --- | --- |
| `AGENTS.md` | 工作守则：遇到什么情况怎么做、记忆怎么用 |
| `SOUL.md` | 人设：说话的语气、做事的边界 |
| `USER.md` | 你的信息：偏好、沟通习惯、正在做的项目 |
| `MEMORY.md` | 长期记忆：值得一直记住的事实和决定 |
| `memory/日期.md` | 每天一份的工作日志，需要时再去翻 |

这种做法的好处是透明。你觉得它哪里做得不对，可以直接改文件里的规则；想换一台电脑，把整个工作区拷过去就行。坏处是这些文件会被模型读到，如果有人想办法往里面写进去一段恶意的指令，后面每一次对话都会受影响，这一点后面讲安全时会再提到。

除了文件，它的能力还可以通过**技能（Skills）**来扩展。技能就是一份写着「某类任务该怎么做」的说明文档，加上可能用到的脚本，平时只占一行简介，用到时才加载全文。官方的技能市场叫 **ClawHub**，你也可以让它自己帮你写一个新技能。

## 你不找它，它也会来找你

ChatGPT、Claude 这类聊天应用，都是你打开它、问一句、它答一句。Claude Code、Cursor 这类编程 Agent 能干活，但也要你坐在电脑前把它叫起来。OpenClaw 让人觉得新鲜的地方，在于它会**主动干活**：

- **定时任务**：像前面「每天早上 8 点发日程」的例子，到点它就自己醒来去做；
- **主动巡检**：设置好以后，它会每隔一段时间（默认是半小时）自己检查一遍有没有需要提醒你的事，比如某封邮件需要你回复、某个服务挂了，没有事情就保持安静，不打扰你；
- **消息驱动**：你在地铁上用手机发一句话，家里的电脑就开始干活，干完再把结果发回到你的手机上。

把这几点合在一起，它更像一个住在你聊天列表里、7×24 小时值班的助理，而不是一个需要你主动打开的工具。这也是它在今年年初突然爆火的原因：技术本身并不神秘，大模型、工具调用、定时任务早就都有了，但把它们装进每个人每天都在用的聊天软件里，让 AI 从「你去找它」变成「它来找你」，OpenClaw 是第一个把这件事做到大家都能上手的开源项目。

之后国内外大厂推出的个人智能体，像 OpenAI 的 dots、腾讯的 WorkBuddy、开源的 Hermes Agent，基本上都沿着这条路在走。

## 装在本地，不等于数据不出门

「本地运行、隐私有保障」是很多介绍文章里最常见的说法，但这句话只说对了一半。OpenClaw 的程序确实跑在你自己的机器上，可它要想理解你的话，就得把内容发给大模型。只要你接的是云端模型，每一次对话的内容，以及它从你的文件、邮件里读出来的相关片段，都会发到模型服务商那里去处理。

| 数据 | 实际放在哪里 |
| --- | --- |
| 配置、记忆文件、技能、聊天记录 | 你自己的电脑或服务器 |
| 每次对话的内容，以及它读到的文件和邮件片段 | 发给你选的模型服务商 |
| 你和它之间来回的消息 | 经过微信、飞书、Telegram 这些聊天软件的服务器 |

所以更准确的说法是：OpenClaw 让你**自己决定数据交给谁**。如果你对隐私要求非常高，可以接 Ollama 这类工具在本地跑开源模型，让数据完全不离开你的机器，代价是本地模型的能力通常比顶尖的云端模型弱，复杂任务更容易出错。

模型的费用也要算进去。OpenClaw 本身免费，但它每次干活都要调用模型，跑定时任务和主动巡检时也会消耗 Token。从 2026 年 4 月起，Anthropic 不再让 Claude Pro、Max 订阅覆盖 OpenClaw 这类第三方工具的用量，想继续在 OpenClaw 里用 Claude，需要额外购买用量包或者改用 API Key。国内用户更常见的做法是直接接 DeepSeek、千问、智谱、Kimi、MiniMax 这些模型的 API，或者买各家的编程套餐（Coding Plan），成本会低不少。

## 从 Clawdbot 到基金会：这一年发生了什么？

OpenClaw 的作者是奥地利开发者 **Peter Steinberger**，他之前创办过 PDF 开发工具公司 PSPDFKit。这个项目最早只是他给自己写的一个助手，一年里换了好几次名字，也经历了不少风波：

| 时间 | 事件 |
| --- | --- |
| 2025 年 11 月 | 第一次提交代码，早期是在开源编程 Agent Pi 的基础上搭起来的 |
| 2026 年 1 月 | 以 Clawdbot 的名字爆火；因为名字太像 Claude，Anthropic 提出商标异议，先改名 Moltbot，三天后定名 OpenClaw |
| 2026 年 1 月底 | 专门给 AI Agent 用的社交网络 Moltbook 上线，大量 OpenClaw 在上面发帖，进一步带火了这个项目 |
| 2026 年 2 月 | Peter Steinberger 宣布加入 OpenAI，OpenClaw 转为由独立的基金会维护 |
| 2026 年 3 月 | 国内掀起「养龙虾」热潮：腾讯云在深圳腾讯大厦楼下办免费安装活动，近千人排队；腾讯、字节、智谱、月之暗面、MiniMax 等陆续推出自己的「龙虾」产品，微信也上线了官方插件 |
| 2026 年 4 月 | Anthropic 停止用 Claude 订阅额度支持 OpenClaw 等第三方工具 |
| 2026 年 7 月 | OpenClaw 基金会正式成立，是美国的 501(c)(3) 非营利组织，有了全职团队，OpenAI 是捐赠方之一 |
| 2026 年 8 月 | 发布 OpenClaw 2.0：安装更简单，网页端重做，支持多人共享同一个会话，任务也可以放到云端的临时机器上跑 |

![OpenClaw 的 GitHub 星标增长曲线：2026 年初从接近 0 陡峭上涨，到 4 月已经超过 30 万，之后增速放缓，到 11 月接近 40 万](images/01-star-history-openclaw.jpg)
*图注：star-history.com 根据 GitHub 数据生成的 openclaw/openclaw 星标历史。从曲线看，OpenClaw 的星标绝大部分是在 1 月到 3 月这三个月里涨起来的，正好对应改名风波和国内的「养龙虾」热潮；4 月以后增速明显放缓，截至写这篇时约 39 万。*

现在 OpenClaw 的版本号直接用日期命名，比如 `v2026.11.4`，几乎每隔几天就会发一个新版本。早期依赖的 Pi，也已经被换成了 OpenClaw 自己写的 Agent 核心。

## 权限越大，出事越大

OpenClaw 好用，是因为它能碰到你的文件、邮箱、浏览器和命令行；它危险，也是因为这些。

AI 研究者 Simon Willison 提出过一个判断 Agent 风险的说法，叫**「致命三要素」**：能接触你的私密数据、会读取外部的不可信内容、能往外发消息。三样同时具备，攻击者就可能在一封邮件或一个网页里藏一段写给 AI 看的指令（也就是**提示词注入**），让它把你的数据偷偷发出去。OpenClaw 的典型用法，恰好三样全占了。

![Simon Willison 画的「致命三要素」示意图：三个圆分别是接触私密数据、接触不可信内容、能对外通信](images/02-lethal-trifecta.jpg)
*图注：Simon Willison 博客文章《The lethal trifecta for AI agents》里的原图。上面的圆是 Access to Private Data（能接触私密数据），左下是 Ability to Externally Communicate（能对外通信），右下是 Exposure to Untrusted Content（会接触不可信的内容）。他的建议是，三样里至少要拿掉一样，才能从根本上防住这类数据泄露。*

这一年里，它在安全上出过的问题，大致可以分成四类：

- **网关被暴露在公网上**：很多人把 OpenClaw 装在云服务器上，又没有设置好访问权限。安全公司 Censys 在 1 月底就扫描到两万多个能从公网直接访问的 OpenClaw 实例，其中不少是早期没有强制登录验证的版本。
- **程序本身的漏洞**：比较严重的一个是 CVE-2026-25253，评分 8.8，只要你点开一个恶意网页，攻击者就能拿到网关的令牌，进而远程控制你的电脑，即使网关只在本机监听也躲不过。这个漏洞在 1 月底的版本里已经修复。
- **技能市场里的恶意技能**：安全公司 Koi Security 在 2 月审查了 ClawHub 上的 2857 个技能，发现 341 个是恶意的，大部分来自同一个团伙，装上以后会诱导你执行命令，在 Mac 上植入窃取密码和钱包的木马；随着技能数量增长，他们之后发现的恶意技能涨到了 800 多个。
- **AI 自己执行跑偏**：Meta 的一位 AI 对齐负责人 Summer Yue 曾让 OpenClaw 帮她整理邮箱，明确交代「动手之前先跟我确认」。结果邮箱内容太多，触发了上下文压缩，这条交代在压缩时被丢掉了，它开始批量删邮件，她在手机上叫停也没用，最后只能跑回电脑前把进程结束掉。

![Summer Yue 在 X 上发的帖子：配图是她和 OpenClaw 的聊天记录，OpenClaw 一条接一条地执行删除邮件的命令，她连续发了「Do not do that」「Stop don't do anything」「STOP OPENCLAW」都没能叫停](images/03-summer-yue-tweet.jpg)
*图注：Summer Yue 2026 年 2 月 22 日在 X 上发的原帖（邮箱地址是她自己打的码）。正文大意是：「我让 OpenClaw『动手前先确认』，然后眼看着它飞快地删我的邮箱。我在手机上根本停不下来，只能像拆炸弹一样冲回我的 Mac mini 前面。」左边的聊天记录里，OpenClaw 打算把 2 月 15 日以前不在保留名单里的邮件全部删掉，她连发三次「别做」「停下」都没用；右下角是事后 OpenClaw 承认「我记得这条要求，但我违反了它」。*

最后一个例子最值得记住：**在聊天里口头交代的规则，不一定靠得住**。上下文一长，它就可能被压缩掉。真正要守住的边界，应该写进配置和权限里，而不是只在对话里说一句。

![动画：红便签「先确认」贴在纸带最前面，邮件把纸带撑满后，最老的一段连同便签被压成「摘要」，AI 连盖三个「删除」；第二次便签钉在「配置」板上，压缩后还在，AI 先在手机上问「删 3 封？」，你按下确认](images/01-compressed-sticky-note.gif)

官方这一年也补了不少防线：陌生人私信它时，默认要你批准配对才能对话；ClawHub 上的技能会经过 VirusTotal 的病毒扫描，并且每天重新扫一遍。但官方也说得很清楚，扫描拦不住所有的提示词注入，而且直到 2.0 版本，**沙箱默认还是关闭的**，主会话里的命令会直接在你的电脑上执行。中国有关部门在 3 月也提醒过 OpenClaw 的安全风险，并限制国有企业和政府机关在办公电脑上使用。

如果你打算自己养一只龙虾，下面几条建议可以大大降低风险：

1. **别装在主力电脑上**：用一台单独的旧电脑、虚拟机或云服务器，或者打开沙箱；
2. **别把网关直接暴露到公网**：需要远程访问时，用官方文档推荐的方式（比如 Tailscale 组网）；
3. **给它单独的账号**：先用一个不重要的邮箱和网盘试，确认靠谱以后再慢慢放权；
4. **技能只装信得过的**：装之前先看一眼技能文档里让它执行了什么命令；
5. **删除、发送、付款这类操作，要求它先问你**，并且把这条规则写进配置文件里。

## 在国内怎么用？

OpenClaw 原版支持的聊天软件有二十多个，Telegram、WhatsApp、Slack、Discord、iMessage 都在里面。国内用户最关心的微信和飞书，现在也都能接：微信在 3 月上线了官方的 ClawBot 插件，装上以后就能在微信里直接和你的 OpenClaw 对话；飞书也有官方插件；钉钉、企业微信、QQ 则主要靠社区开发的插件来接入。

具体怎么用，可以按你愿意折腾的程度选一条路：

| 路线 | 代表产品 | 适合谁 |
| --- | --- | --- |
| 自己部署原版 | OpenClaw 官方版，也可以用腾讯云轻量应用服务器的一键部署 | 懂命令行，想完全自己掌控，愿意自己负责安全 |
| 本地一键安装版 | 腾讯 QClaw、智谱 AutoClaw（澳龙） | 想装在自己电脑上，但不想碰命令行 |
| 云端托管版 | 火山引擎 ArkClaw、月之暗面 Kimi Claw、MiniMax MaxClaw | 不想让电脑一直开着，能接受数据放在厂商的云上 |
| 自研、兼容龙虾技能 | 腾讯 WorkBuddy | 主要处理办公文档，看重企业级的安全管控 |

前三类都是基于 OpenClaw 开发的，区别主要在装在哪里、用谁家的模型、接哪些聊天软件：QClaw 主打微信、QQ 直连；ArkClaw 和飞书结合得比较深；Kimi Claw、MaxClaw 分别默认用自家的 Kimi、MiniMax 模型。这些产品大多要订阅会员或者按用量付费，价格调整得也很频繁，用之前建议先看一下官网上的最新价格。

如果你决定自己部署原版，官方推荐用安装脚本，它会帮你装好需要的环境：

```bash
# macOS / Linux / Windows 的 WSL2
curl -fsSL https://openclaw.ai/install.sh | bash

# Windows PowerShell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

装好以后会进入初始设置，引导你选模型、填 API Key、连接聊天软件。整个过程比年初简单了很多，但遇到报错时还是需要你能看懂命令行，这也是很多人愿意花几百块请人上门代装、或者直接选一键版和托管版的原因。

另外，如果你希望 Agent 能自己总结经验、越用越顺手，也可以看看同样开源的 **Hermes Agent**，它可以直接导入 OpenClaw 的配置，也能安装 ClawHub 上的技能。

## 时效性提醒

OpenClaw 的版本更新非常频繁，下面这些信息最容易变：

- 支持的聊天软件，尤其是微信插件的功能范围，比如上线时还不支持群聊；
- 默认的安全设置，比如沙箱是否默认打开；
- 国内各家龙虾产品的价格、免费额度和可用的模型；
- 各家模型厂商对第三方工具的订阅政策。

动手之前，建议先看一眼 GitHub 上的 Releases 页面和官方文档。

💬 **互动话题**：如果让你养一只龙虾，你最想先把哪件每天重复做的事交给它？你会放心让它碰你的主力邮箱和电脑吗？

📖 **相关阅读**：

- [3分钟搞懂AI-Agent](../../02-core-capabilities/05-ai-agent/README.md) - OpenClaw 背后的 Agent 是怎么一步步把事情做完的
- [3分钟搞懂Hermes-Agent](../03-hermes-agent/README.md) - 同样是开源个人智能体，多了一套自己总结经验的学习循环
- [3分钟搞懂WorkBuddy](../02-workbuddy/README.md) - 腾讯自研的桌面办公智能体，兼容龙虾的技能
- [3分钟搞懂Pi](../../07-dev-tools/09-pi/README.md) - OpenClaw 早期就是在这个极简编程 Agent 上搭起来的
- [3分钟搞懂Skills](../../09-agent-engineering/04-skills/README.md) - ClawHub 上的技能用的就是这套格式
- [3分钟搞懂Agent记忆系统](../../09-agent-engineering/06-agent-memory/README.md) - MEMORY.md 和每日日志分别解决什么问题
- [3分钟搞懂Agent沙箱与权限](../../09-agent-engineering/08-sandbox-and-permissions/README.md) - 为什么建议把龙虾关进沙箱里养
- [3分钟搞懂Prompt注入攻击](../../04-advanced/13-prompt-injection/README.md) - 能读邮件、能发消息的 Agent 为什么最怕被注入
- [3分钟搞懂Ollama](../../07-dev-tools/11-ollama/README.md) - 在本地跑模型，让数据完全不出你的电脑

**资料来源**：

- OpenClaw GitHub 仓库（MIT 协议、星标数、安装方式、支持的聊天软件、私信配对和沙箱的默认设置）：https://github.com/openclaw/openclaw
- OpenClaw GitHub Releases（v2026.11.4，2026-11-16）：https://github.com/openclaw/openclaw/releases
- OpenClaw 官方文档，Agent workspace（AGENTS.md、SOUL.md、USER.md、MEMORY.md 和每日日志）：https://docs.openclaw.ai/concepts/agent-workspace
- OpenClaw 官方文档，Heartbeat（主动巡检改为普通定时任务，默认每 30 分钟一次）：https://docs.openclaw.ai/gateway/heartbeat
- OpenClaw 官方博客，Introducing the OpenClaw Foundation（2026-07-08，501(c)(3) 非营利组织、捐赠方、全职团队）：https://openclaw.ai/blog/introducing-openclaw-foundation
- OpenClaw 官方博客，VirusTotal 合作（2026-02-07，ClawHub 技能扫描与每日复扫）：https://www.openclaw.ai/blog/virustotal-partnership
- Wikipedia，OpenClaw 词条（改名经过、Moltbook、国内限制、2.0 发布时间）：https://en.wikipedia.org/wiki/OpenClaw
- Help Net Security，OpenClaw creator Peter Steinberger joins OpenAI（2026-02-16）：https://www.helpnetsecurity.com/2026/02/16/openclaw-creator-peter-steinberger-joins-openai/
- The Decoder，OpenClaw 2.0 brings simplified setup, a rebuilt browser app and multiplayer sessions（2026-08-31）：https://the-decoder.com/openclaw-2-0-brings-simplified-setup-a-rebuilt-browser-app-and-multiplayer-sessions/
- The Decoder，Anthropic 停止用 Claude 订阅支持第三方工具（2026-04）：https://the-decoder.com/?p=33980
- Censys，OpenClaw in the Wild（截至 2026-01-31 发现 21639 个公网可访问实例）：https://censys.com/?p=3946
- The Hacker News，OpenClaw Bug Enables One-Click Remote Code Execution（CVE-2026-25253，CVSS 8.8，v2026.1.29 修复）：https://thehackernews.com/2026/02/openclaw-bug-enables-one-click-remote.html
- Koi Security，ClawHavoc: 341 malicious skills（2026-02，后续更新为 824 个）：https://koi.ai/blog/clawhavoc-341-malicious-clawedbot-skills-found-by-the-bot-they-were-targeting
- Windows Central，Meta 的 Summer Yue 邮箱被 OpenClaw 批量删除（2026-02）：https://www.windowscentral.com/artificial-intelligence/meta-summer-yue-director-openclaw-ai-email-deletion
- Simon Willison，The lethal trifecta for AI agents（2025-06-16，「致命三要素」；文中示意图出处）：https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- Summer Yue 在 X 上的原帖（2026-02-22，文中截图出处）：https://x.com/summeryue0/status/2025774069124399363
- star-history.com，openclaw/openclaw 星标历史（文中星标曲线图出处）：https://www.star-history.com/#openclaw/openclaw&Date
- 联合新闻网（转引中新社），微信推出 ClawBot 插件支持接入 OpenClaw（2026-03-22）：https://udn.com/news/story/7333/9395441
- IT之家，腾讯 QClaw 开启全量公测（2026-03-20）：https://www.ithome.com/0/930/937.htm
- IT之家，智谱 AutoClaw（澳龙）正式上线（2026-03-10）：https://www.ithome.com/0/927/423.htm
- IT之家，火山引擎上线 ArkClaw（2026-03-09）：https://www.ithome.com/0/927/235.htm
- 虎嗅，月之暗面推出 Kimi Claw：https://www.huxiu.com/article/4838318.html
- 华尔街见闻，MiniMax 推出 MaxClaw（2026-02）：https://wallstreetcn.com/articles/3766231
- 扬子晚报，深圳腾讯大厦免费安装 OpenClaw 近千人排队（2026-03-06）：https://www.yzwb.net/news/ch/202603/t20260306_328547.html

---

[📚 返回目录](../README.md) ｜ 下一篇：[3分钟搞懂WorkBuddy](../02-workbuddy/README.md) ➡️
