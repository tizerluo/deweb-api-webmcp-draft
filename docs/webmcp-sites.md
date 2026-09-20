# Draft: WebMCP tool lists on DeWEB sites

Status: **unofficial draft** (2026-09-20; rev. 2026-09-21 — pre-discussion corrections vs SPEC §7.8 and the current WebMCP surface)  
Working name used in discussion: “TapeOutWebMCP” / “TAP-12” — **number not assigned**  
Depends on: TapeKit top-level origin (an official gateway such as tapekit.org, a HashPort domain, or a self-hosted gateway — one HTTPS origin per site), [WebMCP](https://github.com/webmachinelearning/webmcp)  
Related: [request-response.md](request-response.md) (optional; WebMCP does **not** require payment or TAP-10)

---

## 1. Problem / 问题

Agents should not scrape a DeWEB page’s DOM. WebMCP lets a **top-level, secure** page register tools on `document.modelContext`. In gateway mode sites already open as real origins (`https://<#ID>-<cpu>.<gateway>/`); the sandbox viewer (preview mode, opaque origin) is the case that can **never** be a WebMCP provider.

Need:

1. A file in the container listing tools: `/.tape/mcp.json`
2. TapeKit, when the site is a **real origin**, registers those tools (native WebMCP if present, else a kernel polyfill)
3. `execute` stays in the page / kernel. It MUST NOT be a free off-chain `fetch` from site JS while SPEC §7.5/§7.8 block or gate those. Mode A reads chain files; anything else goes through kernel helpers (`window.tape`) or TAP-10 operators.

Agent 不该刮 DOM。WebMCP 要求真顶层源。沙盒 iframe 当不了提供方。

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
      "via": { "file": "/.tape/price.json" }
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

> **Reserved-path note.** `/.tape/` is currently gateway-reserved (SPEC §7.8: “files under those names in a site are never read”). This draft requests an explicit carve-out for `/.tape/mcp.json` (and `/.tape/api.json` in the request/response draft): site-declared files read by the kernel from the chain, while HTTP under `/.tape/` keeps going to the gateway. A non-reserved path is equally acceptable if maintainers prefer one. 中文：`/.tape/` 目前归网关保留；草案请求为站点声明的这两个文件开一条口子——由内核从链上读，HTTP 仍归网关；维护者想换路径也可以。

`via.file` = mode A (a chain read via the kernel).  
`via.endpoint` = optional TAP-10 request/response; payment is not implied. Endpoints use the TAP-10 display form `#<#ID>@<processor number>`; readable names such as `translate.tape` do not exist in the current naming rules (an earlier example used one as shorthand — corrected above).

---

## 3. TapeKit behaviour / 内核行为

When opening a site as a **top-level HTTPS origin** (official gateway / HashPort / self-hosted):

1. After files verify, if `/.tape/mcp.json` exists and hashes match, parse it.
2. If `document.modelContext` already exists (Chrome origin trial), register through the native object: `registerTool` (+ `unregisterTool` / `AbortSignal` for lifecycle) and `getTools`. The page must be a secure top-level context, and a `Permissions-Policy` that disallows `tools` disables registration — the gateway’s policy must allow it.
3. Else TapeKit MAY install a polyfill mirroring that surface: `registerTool` / `unregisterTool`, `getTools`, and a debug-only in-page `executeTool` — native agents talk to the browser, never to a page API.
4. Each tool’s `execute` is supplied by the **site** (page script) or by the kernel mapping `via.file` / `via.endpoint`. Site script still cannot raw-fetch the open web unless the kernel allows it.
5. Tools run in the site’s origin. The polyfill and any kernel helpers MUST NOT expose key material or signing to page script (TAP-10 §11: refuse raw-hash signing; never hand `tapesend:hub:` text to `personal_sign`; the official client is not served from a domain that serves user sites). Wallet actions, when a tool triggers them, go through the shell, with the user present.

When the site is only a sandboxed iframe: do **not** claim WebMCP. Show the page as today.

Consent: a `via.endpoint` call is a send — a wallet transaction from the sender’s wallet (TAP-10 §10) whose metadata is public and permanent (§11) — so it always involves the user, free or paid; “no price” is not “no interaction”. Anything that attaches assets still needs an explicit confirmation, and `via.file` tools never send, so no prompt. If relayed or sponsored sends are ever introduced, revisit this rule.

---

## 4. Payment is not part of WebMCP / 支付不是 WebMCP 的前提

WebMCP is a page → agent surface. TAP-10 mail is one **backend** a tool might use. A site MAY expose only mode A tools and never send mail. Note the send-side baseline: even a “free” TAP-10 call is a wallet transaction from its sender, with public, permanent metadata (TAP-10 §10–§11).

---

## 5. Ask of maintainers / 请维护者决定

- Will the official gateway (tapekit.org) and HashPort prefer top-level origin for sites that ship `/.tape/mcp.json`?
- Is the polyfill in kernel or in a separate package (key code stays out of the page kernel, same split as `@tapekit/send`)?
- File path: `/.tape/mcp.json` sits under the currently reserved `/.tape/` prefix — do you accept a §7.8 carve-out for site-declared files read by the kernel, or should we use another path?
