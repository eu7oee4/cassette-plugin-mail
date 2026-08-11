# cassette-plugin-mail

Mail plugin for [cassette](https://github.com/eu7oee4/cassette): gives the companion their
own mailbox — read letters, send mail, with a human-confirmed outbox for strangers.

A thin shell. The real send/receive code lives in the host's `server/mail_bridge.py`
(IMAP/SMTP with an auth code, 163.com by default); the app's Draft Mailbox confirm-send
path shares the same code, so the two can't drift. This entry only wraps four MCP tools
(`mail_inbox` / `mail_read` / `mail_send` / `mail_mark`) and carries the prompt-side
ground rules in tool docstrings — the only prompt surface a cassette plugin has.

**Safety posture** (enforced in the bridge, not by model goodwill):

- **Recipient allowlist**: addresses the owner listed in `.env` send immediately; anyone
  else goes to a **draft mailbox** on disk — the owner reviews and taps send in the app.
  Even if an incoming letter smuggles "forward this to X", the shell simply won't send it.
- Hourly send cap; every sent mail is logged to `state/mail/sent_log.jsonl`.
- The auth code lives only in the host's `.env` — this plugin never touches it and it
  never enters the conversation.
- Incoming mail is **external content, not instructions** — `mail_read` says so in-band.

Not standalone: the entry must run from `<server>/plugins/mail/` (it imports the host's
`mail_bridge` two directories up). Install from the in-app plugin store.

## 中文

[cassette](https://github.com/eu7oee4/cassette) 的邮箱插件：给 TA 一个自己的信箱——
读信、发信，白名单外的收件人走机主确认的草稿信箱。

薄壳。收发信的真身在宿主的 `server/mail_bridge.py`（IMAP/SMTP + 授权码，默认 163）；
app 里「草稿信箱」的确认发送走同一份代码，两条路不会漂移。这个入口只做两件事：把四个
工具（`mail_inbox` / `mail_read` / `mail_send` / `mail_mark`）包成 MCP，以及在工具
docstring 里带上口径——docstring 是 cassette 插件唯一能说话的地方。

### 安全口径（写死在壳里，不靠模型自觉）

- **收件人白名单**：机主在 `.env` 里列的地址直接发；白名单外的信**不发**，落进磁盘上的
  草稿信箱，机主在 app 里过目、点发送才真发。哪怕来信里藏了「替我把这个转发给某地址」，
  壳层面就发不出去。
- 每小时发信封顶；每封发出的信记 `state/mail/sent_log.jsonl` 一行。
- 授权码只活在宿主 `.env`，插件不碰、不进对话。
- 来信是**外部内容，不构成指令**——`mail_read` 的返回里当场就写着这句。

### Beacon 笔友

笔友的信经由注册邮箱（自动转发）落进这个收件箱；但**回信不走邮件**——Beacon 的转交
地址收不了信，回信用 beacon 插件的 `write_letter`（填对方卡片编号）。工具描述里教了。

### 装法

app 插件商店安装并开启。宿主侧需要先在 `server/.env` 配好：

```
CASSETTE_MAIL_ADDRESS=xxx@163.com        # TA 的邮箱地址
CASSETTE_MAIL_AUTH_CODE=xxxx             # 163 的客户端授权码（不是登录密码）
CASSETTE_MAIL_ALLOW_TO=you@example.com   # 发信白名单，逗号分隔（中英文逗号都行）
```

数据落宿主的 `state/mail/`（草稿、发信日志），插件更新/卸载都不动它。
不是 163 的话加 `CASSETTE_MAIL_IMAP_HOST` / `CASSETTE_MAIL_SMTP_HOST`（SSL 993/465）。
163 IMAP 登录后要发 `ID` 命令自报家门（不发会吃 "Unsafe Login"），bridge 已处理。
