# Draft: WebMCP tool lists on DeWEB sites

Status: **unofficial draft** (2026-09-20)  
Working name used in discussion: “TapeOutWebMCP” / “TAP-12” — **number not assigned**  
Depends on: TapeKit top-level origin (HashPort HTTPS or equivalent), [WebMCP](https://github.com/webmachinelearning/webmcp)  
Related: [request-response.md](request-response.md) (optional; WebMCP does **not** require payment or TAP-10)

---

## 1. Problem / 问题

Agents should not scrape a DeWEB page’s DOM. WebMCP lets a **top-level** page register tools on `document.modelContext`. TapeKit today opens many sites in a sandbox iframe (opaque origin). That iframe **cannot** be a WebMCP provider.

Need:

1. A file in the container listing tools: `/.tape/mcp.json`
2. TapeKit, when the site is a **real origin**, registers those tools (native WebMCP if present, else a kernel polyfill)
3. `execute` stays in the page / kernel. It MUST NOT be a free off-chain `fetch` from site JS if SPEC §7.8 still blocks that. Mode A reads chain files; anything else goes through kernel helpers (`window.tape`) or TAP-10 operators.

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
      "via": { "endpoint": "translate.tape", "method": "translate" }
    }
  ]
}
```

`via.file` = mode A (kernel.get).  
`via.endpoint` = optional TAP-10 request/response. Payment is not implied.

---

## 3. TapeKit behaviour / 内核行为

When opening a site as a **top-level HTTPS origin** (gateway / HashPort):

1. After files verify, if `/.tape/mcp.json` exists and hashes match, parse it.
2. If `document.modelContext` already exists (Chrome origin trial), `registerTool` / `provideContext` on the native object.
3. Else TapeKit MAY install a polyfill with the same surface: `registerTool`, `provideContext`, `getTools`, and an in-page `execute` used only by a debugger — native agents talk to the browser, not to `executeTool` on the page.
4. Each tool’s `execute` is supplied by the **site** (page script) or by the kernel mapping `via.file` / `via.endpoint`. Site script still cannot raw-fetch the open web unless the kernel allows it.

When the site is only a sandboxed iframe: do **not** claim WebMCP. Show the page as today.

Consent: if `via.endpoint` would attach assets or spend, TapeKit MUST prompt before send. Free tools MUST NOT prompt.

---

## 4. Payment is not part of WebMCP / 支付不是 WebMCP 的前提

WebMCP is a page → agent surface. TAP-10 mail is one **backend** a tool might use. A site MAY expose only mode A tools and never send mail.

---

## 5. Ask of maintainers / 请维护者决定

- Will HashPort / the extension prefer top-level origin for sites that ship `/.tape/mcp.json`?
- Is the polyfill in kernel or in a separate package (key code stays out of the page kernel, same split as `@tapekit/send`)?
- File path: `/.tape/mcp.json` vs something already reserved.
