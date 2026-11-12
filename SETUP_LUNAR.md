# 月壤热传导论文每日推送

本分支基于 fangzhigang32/PaperReader，增加 src/lunar_digest.py 作为当前工作流入口。
原 src/main.py 等文件保留用于追溯，当前定时任务不再调用它们。

## 已设置的流程

- GitHub Actions：Lunar Paper Digest。
- 每天 UTC 00:00，即北京时间 08:00 触发；排队及检索/摘要/发送需要时间，不承诺 08:00 准时到信。
- 每次最多 5 篇，回查近 7 天的 arXiv 更新与 Crossref 新收录记录。
- 重点：月壤热物性、热传导模型、分层、横向传热/三维、地形与辐射边界、Diviner 与参数约束。
- 关键词筛选是初筛，不能保证每篇都相关，也不是所有期刊的完整覆盖。
- 每篇显示题目、来源、发表日期、数据库记录日期、摘要和原文链接。
- 没有摘要会明确注明；没有合适新论文会发送说明，不拿旧论文或无关论文凑数。
- Crossref 新收录记录可能是旧论文；arXiv 更新可能只是修订。邮件分别标注，不能将其当作“昨日发表”。
- arXiv 查询上限 100 条；Crossref 每个主题查询 60 条。没有无限翻页。
- 仅在 SMTP 接受邮件后记录推送历史。历史存放 GitHub Actions cache；缓存被清除/过期可能造成重复。
- 若只有一个数据源可用，邮件提示检索不完整；两个数据源都失败时工作流失败，不发送“没有新论文”的误导消息。
- 运行结果保存在 Actions 的 lunar-digest 附件内（14天）；不再把日志自动提交进公开仓库。
- 原仓库继承的 papers/、runlog/ 是上游历史，不是本账号的推送记录。

## 必须配置的 3 个 Repository Secrets

仓库 Settings → Secrets and variables → Actions → New repository secret：

| 名称 | 内容 |
|---|---|
| SENDER_EMAIL | 你确认使用的 163 发件邮箱 |
| RECEIVER_EMAIL | 你确认的 QQ 收件邮箱 |
| SENDER_PASS | 163 邮箱的客户端授权码，不是邮箱登录密码 |

邮件服务器已设为 smtp.163.com，SSL 端口 465。
在 163 网页邮箱进入“设置 → POP3/SMTP/IMAP”，开启 SMTP 相关服务，按页面要求完成身份验证并获取客户端授权码。
由账号本人完成身份验证与授权码生成，将授权码直接保存到 SENDER_PASS；不要放进代码或聊天。
更换发件邮箱后，要同时更新 SENDER_EMAIL 和 SENDER_PASS。收件邮箱仍使用已确认的 QQ 邮箱。
邮箱地址和授权码只放 Secrets，不填入代码、截图、聊天或 README。
没有配置齐这三项，定时发送会明确失败，不会假装已启用。

## 可选：中文 AI 速览

不配置这部分也能检索和发送，保留原始标题、摘要和中文分类标签。
如需中文译名和阅读提示，添加以下 3 个 Secrets：

| 名称 | 内容 |
|---|---|
| OPENAI_API_KEY | 自己模型服务的 API 密钥 |
| OPENAI_API_BASE | 支持 Chat Completions 的 HTTPS 基础地址，通常以 /v1 结尾 |
| MODEL | 该密钥实际可调用的模型 ID |

可填写完整 /chat/completions 地址；只支持非流式 Chat Completions 响应格式。
Responses-only 的模型或端点不适用。不要把服务商显示名称当作模型 ID。
本实现不需要 MODEL_PROVIDER、SEARCH_TEXT、ARXIV_COUNT 或 CROSSREF_COUNT Secrets。
模型请求会将候选的公开标题和摘要发到你配置的 API 服务，最多每篇一次。
摘要最多传入12000个字符。AI 提示会标注“未读全文”，不代替原文。
AI 超时/响应错误会保留原始记录并明确提示，不会把全部候选隐藏。
AI 调用可能产生服务商费用；不配置密钥就不发生 AI 调用。

## 启用和首次运行

1. 在 Actions 页面，如出现 fork 的工作流提示，选择启用工作流。
2. 按上表添加 3 个邮件 Secrets。
3. Actions → Lunar Paper Digest → Run workflow。
4. mode=preview：只生成 digest.html / digest.json，不发送，也不更新已推送历史。
5. mode=send：实际发送一次邮件，随后记录已推送论文。
6. 到 QQ 收件箱和垃圾邮件检查是否收到。工作流显示 SMTP 接受并不等于已确认进收件箱。
7. 定时工作流从默认分支执行。公开仓库长时间无活动时定时任务可能被 GitHub 禁用，应检查 Actions。
8. GitHub 的排队延迟、缓存策略和邮件反垃圾规则仍会影响实际送达。

## 当前实现与检索范围

工作流：.github/workflows/main.yml
程序：src/lunar_digest.py
依赖：requirements-digest.txt（替代原项目的大型浏览器/LLM依赖链，仅安装本流程所需的包）

目前仅元数据检索，不绕过付费墙，不下载全文，不承诺每天都有高度相关新论文。
如果要调整研究方向，修改 classify() 的关键词与权重，以及 arxiv_papers()/crossref_papers() 的查询。
先根据实际收到的前几期记录检查漏检和误检，再扩大关键词。
