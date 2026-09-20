# TapeKit issue draft — to post at https://github.com/TapeOutProtocol/TapeKit/issues/new

> Status: prepared 2026-09-21 by tizerluo (with Hermes). Not yet posted. Discussions are disabled on TapeKit, so this goes in as an issue per CONTRIBUTING.md.

**Suggested title:** Proposal (unofficial drafts): request/response content on TAP-10 + WebMCP tool lists for tape:// sites

---

## Body (English)

Hi — I've been building on TapeOut/DeWEB (a small NAND-circuit demo, TapeOutTank) and hit two gaps I think others will hit soon. I wrote two **unofficial drafts** to make the discussion concrete — they live in a separate repo and do **not** claim any TAP number:

- **Draft A — request/response content on TAP-10:** https://github.com/tizerluo/deweb-api-webmcp-draft/blob/main/docs/request-response.md
- **Draft B — WebMCP tool lists for DeWEB sites:** https://github.com/tizerluo/deweb-api-webmcp-draft/blob/main/docs/webmcp-sites.md

The problems, in one line each:

- **A:** there is no agreed content convention meaning "this mail is an API call" / "this mail is the matching result", so sites that need a backend (translate, rank, charge) must leave DeWEB or invent private schemas.
- **B:** agents should not scrape DeWEB pages' DOM; WebMCP is the standard tool surface for that, but it needs a top-level secure origin — the sandbox viewer can never provide one.

Both drafts were written against the current rules, and deliberately flag the friction points instead of glossing over them:

1. **Reserved paths (SPEC §7.8).** `/.tape/` is currently gateway-owned and "files under those names in a site are never read". Both drafts put discovery files under `/.tape/` (`api.json` / `mcp.json`) and therefore **ask for an explicit carve-out**: site-declared files read by the kernel from the site store, while HTTP under `/.tape/` stays with the gateway (or a different path, if you prefer).
2. **Content kinds (TAP-10 v1.0 §6).** §6 recognizes exactly one kind, `"message"`. Draft A proposes adding kinds (`deweb.req/v0` / `deweb.res/v0` are working labels). Under the current strict decode rules, old clients treat them as `unsupported` — safe degradation, by design.
3. **Send reality (§10/§11).** A send is a wallet transaction from the sender's wallet, with public, permanent metadata. Draft B therefore does not claim that "free" tools avoid user interaction: `via.file` tools never send; `via.endpoint` tools always involve the user.
4. **No vanity names.** Examples use the endpoint display form (`#<#ID>@<processor>`), since current naming is numeric only.

Neither draft changes anything in §15.1, requires a token or escrow, or self-numbers. Each doc ends with an "Ask of maintainers" section with the specific rulings I'd love to get. If the right home is the TAP-10 v1.0 rewrite rather than a separate number, I'm happy to adapt; if the direction is unwelcome, saying so is a perfectly good outcome for a draft. (Happy to split this into two issues if that's easier to track.)

---

## 中文摘要

两份**非官方草案**，求讨论与裁定（不抢 TAP 号）：

- **草案 A（TAP-10 上的请求/响应内容约定）**：补上"这封信是一次 API 调用"的约定，站点不必离开 DeWEB 自建后端。
  https://github.com/tizerluo/deweb-api-webmcp-draft/blob/main/docs/request-response.md
- **草案 B（DeWEB 站点的 WebMCP 工具清单）**：Agent 不刮 DOM；站点在真顶层源下注册工具，`/.tape/mcp.json` 声明。
  https://github.com/tizerluo/deweb-api-webmcp-draft/blob/main/docs/webmcp-sites.md

草案已按现状对表，并主动标注三条约束请维护者裁定：

1. `/.tape/` 是网关保留前缀（§7.8）——请求为站点声明文件开一条口子（内核从链上读、HTTP 仍归网关），或另选路径；
2. TAP-10 v1.0 §6 目前只认一种 `kind`（`"message"`）——草案 A 提议新增，旧客户端会安全降级为 `unsupported`；
3. 发送 = 发送方钱包交易、元数据公开永久（§10/§11）——草案 B 不声称"免费 = 无交互"。

两份草案都不动 §15.1、不要求代币/托管、不自编号；如更适合并入 v1.0 重写，愿意配合。
