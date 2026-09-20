# Draft: WebMCP tool lists on DeWEB sites

Status: **unofficial draft** (2026-09-20; rev. 2026-09-21b — pre-discussion corrections + Round-1 review fixes vs the current WebMCP surface and SPEC §7.5/§7.8)  
Working name used in discussion: “TapeOutWebMCP” / “TAP-12” — **number not assigned**  
Depends on: a top-level HTTPS origin — this draft’s venue choice (an official gateway such as tapekit.org, a HashPort domain, or a self-hosted gateway; one origin per site) — and [WebMCP](https://github.com/webmachinelearning/webmcp)  
Related: [request-response.md](request-response.md) (optional; WebMCP does **not** require payment or TAP-10)

English is the normative text of this draft; the Chinese text is a non-normative summary.

---

## 1. Problem / 问题

Agents should not scrape a DeWEB page’s DOM. WebMCP registers tools on `document.modelContext`: by default it is available in top-level windows and same-origin iframes, and can be delegated to cross-origin iframes with the Permissions Policy `tools` (`allow="tools"`). **This draft’s venue is the top-level gateway origin** — a security/deployment choice, not a WebMCP requirement. In gateway mode sites already open as real origins (`https://<#ID>-<cpu>.<gateway>/`); the sandbox viewer (preview mode, opaque origin) is the case that can **never** be a WebMCP provider.

Need:

1. A file in the container listing tools: `/.tape/mcp.json`
2. TapeKit, when the site is a **real origin**, registers those tools (native WebMCP if present, else a kernel polyfill)
3. `execute` stays in the page / kernel. This draft adds no new network path beyond what a site already has — note the current split: SPEC §7.5 asks shells to block off-chain requests by default and §7.8 makes a Service Worker gateway block and record them, while the current gateway (0.3.x) allows and records non-script off-chain traffic. Which model to target is a maintainer question (§5). Mode A reads chain files; anything else goes through kernel helpers (`window.tape`) or TAP-10 operators.

Agent 不该刮 DOM。工具注册在 `document.modelContext`；本草案让网关的顶层源做注册点（部署选择，不是 WebMCP 的硬性要求）。沙盒 iframe 当不了提供方。

---

## 2. Discovery file / 发现文件

```json
{
  "webmcp": true,
  "tools": [
    {
      "name": "get_price",
      "description": "Read BEM/USDT from an on-chain file.",
      "inputSchema": { "type": "object", "properties": {} },
      "via": { "file": "/data/price.json" }
    },
    {
      "name": "translate_dialogue",
      "description": "Translate a line. Optional TAP-10 call.",
      "inputSchema": {
        "type": "object",
        "properties": {
          "text": { "type": "string" },
          "to": { "type": "string" }
        },
        "required": ["text", "to"]
      },
      "via": { "endpoint": "#9102@0", "method": "translate" }
    }
  ]
}
```

> **Reserved-path note.** `/sw.js` and the whole `/.tape/` prefix are currently gateway-reserved (SPEC §7.8: “files under those names in a site are never read”). This draft requests an explicit carve-out for `/.tape/mcp.json` (and `/.tape/api.json` in the request/response draft): site-declared files read by the kernel from the chain, while HTTP under `/.tape/` keeps going to the gateway. A non-reserved path is equally acceptable if maintainers prefer one. 中文：`/sw.js` 与整个 `/.tape/` 前缀目前归网关保留；草案请求为站点声明的这两个文件开一条口子——由内核从链上读，HTTP 仍归网关；维护者想换路径也可以。

`via.file` = mode A (a chain read via the kernel); data files live at ordinary site paths — outside `/sw.js` and the gateway-reserved `/.tape/` prefix (e.g. `/data/price.json`).  
`via.endpoint` = optional TAP-10 request/response; payment is not implied. Endpoints use the TAP-10 display form `#<#ID>@<processor number>`; readable names such as `translate.tape` do not exist in the current naming rules (an earlier example used one as shorthand — corrected above).

---

## 3. TapeKit behaviour / 内核行为

When opening a site as a **top-level HTTPS origin** (official gateway / HashPort / self-hosted):

1. After files verify, if `/.tape/mcp.json` exists and hashes match, parse it.
2. If `document.modelContext` already exists (Chrome origin trial), register through the native object: `registerTool({...}, {signal})`; a tool is unregistered by **aborting the `AbortSignal`** passed at registration — the surface has no `unregisterTool`. Read the registered set back with `getTools`, and follow changes through the `toolchange` event (`ontoolchange`). Registration is rejected (`NotAllowedError`) when the policy-controlled feature `tools` is disabled — via the `Permissions-Policy: tools=()` header or a cross-origin frame without `allow="tools"` — so the gateway’s policy must allow it.
3. Else TapeKit MAY install a polyfill mirroring that surface: `registerTool` (+ `AbortSignal`), `getTools`, `executeTool`, `ontoolchange`. `getTools` and `executeTool` are the standard discovery/execution API for **in-page agents** — browser-native agents use browser internals instead — and a page-level polyfill cannot make tools visible to a browser-native agent when the browser lacks WebMCP; it only serves page/in-page consumers that know the polyfill.
4. Each tool’s `execute` is supplied by the **site** (page script) or by the kernel mapping `via.file` / `via.endpoint`. Note what is already true today: the current gateway (0.3.x) allows and records non-script off-chain traffic — page script can `fetch` off-chain endpoints (`connect-src` allows `https:`/`wss:`), and only scripts stay on-chain — while SPEC §7.5 asks shells to block off-chain requests by default and §7.8 makes a Service Worker gateway block and record them. This draft claims no change to that state and asks maintainers which model to target (§5).
5. Tools run in the site’s origin. The polyfill and any kernel helpers MUST NOT expose key material or signing to page script (TAP-10 §11: refuse raw-hash signing; never hand `tapesend:hub:` text to `personal_sign`; the official client is not served from a domain that serves user sites). Wallet actions, when a tool triggers them, go through the shell, with the user present.

When the site is only a sandboxed iframe: do **not** claim WebMCP — an opaque-origin frame is neither top-level nor same-origin, so it can never be a tool provider. Show the page as today.

Consent: a `via.endpoint` call is a send — a wallet transaction from the sender’s wallet (TAP-10 §10) whose metadata is public and permanent (§11) — so it always involves the user, free or paid; “no price” is not “no interaction”. Anything that attaches assets still needs an explicit confirmation, and `via.file` tools never send, so no prompt. If relayed or sponsored sends are ever introduced, revisit this rule.

---

## 4. Payment is not part of WebMCP / 支付不是 WebMCP 的前提

WebMCP is a page → agent surface. TAP-10 mail is one **backend** a tool might use. A site MAY expose only mode A tools and never send mail. Note the send-side baseline: even a “free” TAP-10 call is a wallet transaction from its sender, with public, permanent metadata (TAP-10 §10–§11).

---

## 5. Ask of maintainers / 请维护者决定

- Will the official gateway (tapekit.org) and HashPort prefer top-level origin for sites that ship `/.tape/mcp.json`?
- Is the polyfill in kernel or in a separate package (key code stays out of the page kernel, same split as `@tapekit/send`)?
- File path: `/.tape/mcp.json` sits under the currently reserved `/.tape/` prefix — do you accept a §7.8 carve-out for site-declared files read by the kernel, or should we use another path?
- Off-chain model: SPEC §7.5/§7.8 says block and record by default; the current gateway (0.3.x) allows and records non-script off-chain traffic. Which model should this proposal target?
