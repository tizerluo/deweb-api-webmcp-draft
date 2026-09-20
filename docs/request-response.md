# Draft: request/response content on TAP-10

Status: **unofficial draft** (2026-09-20; rev. 2026-09-21b — pre-discussion corrections + Round-1 review fixes vs TAP-10 v1.0 §1/§2.2/§6 and SPEC §7.5/§7.8)  
Working name used in discussion: “TapeAPI” / “TAP-11” — **number not assigned**  
Depends on: TAP-10 TapeSend / the DeWEB messaging layer ([TapeKit/send](https://github.com/TapeOutProtocol/TapeKit/tree/main/send); v1.0 rewrite in progress)  
Does not change: TapeKit SPEC §15.1 promises (names, verification, node agreement)

English is the normative text of this draft. The Chinese text in this document is a non-normative summary; the rules are defined in English only.

---

## 1. Problem / 问题

A DeWEB site can publish files. A container can send TAP-10 mail. There is no agreed **content type** that means “this mail is an API call” and “this mail is the matching result”.

Sites that need a backend (translate, rank, charge) currently either:

- leave DeWEB and run an ordinary server, or
- invent a private mail schema.

A small content convention on TAP-10 is enough. No new chain.

DeWEB 站点能发文件，容器能发 TAP-10 信。缺的是约定「这封信是一次调用」。不必新链。

---

## 2. Discovery file / 发现文件

A provider MAY publish `/.tape/api.json` in its site store (same SHA-256 rules as any DeWEB file).

> **Reserved-path note (please read first).** SPEC §7.8 today reserves `/sw.js` and the whole `/.tape/` prefix to the gateway — “files under those names in a site are never read”. In the current gateway (0.3.x) that namespace holds static resources served from the host (`boot.js`, `pages.js`, `config.json`, `blocklist.txt`, `policy.html`, `kernel/*`) and generated endpoints (`status`, `settings`, `csp-report`). This draft therefore **asks for an explicit carve-out**: `/.tape/api.json` (and `/.tape/mcp.json` in the WebMCP draft) are site-declared files read by the kernel **from the site store (chain)**, while HTTP requests under `/.tape/` keep going to the gateway. If maintainers prefer another, non-reserved path, that choice belongs in this discussion. 中文：`/sw.js` 与整个 `/.tape/` 前缀目前归网关保留（站点里这些名字的文件「永不读取」，SPEC §7.8）；现状里这一命名空间既有静态资源（`boot.js`、`pages.js`、`config.json`、`blocklist.txt`、`policy.html`、`kernel/*`）也有网关生成的端点（`status`、`settings`、`csp-report`）。草案请求明确开一条口子——这两个文件由内核从链上读，HTTP 命名空间仍归网关。若维护者想换非保留路径，也在这里讨论定。

```json
{
  "tap": 10,
  "name": "translate",
  "endpoint": "#9102@0",
  "mode": "hybrid",
  "methods": {
    "translate": {
      "price": { "token": "BEM", "amount": "0.02" },
      "timeoutBlocks": 20
    }
  }
}
```

- `tap: 10` means “this file describes a TAP-10 endpoint”, not a new TAP number.
- `mode`: `onchain-file` | `onchain-state` | `hybrid`
- `price.amount` `"0"` = free. Payment is **optional**.

Clients resolve the on-chain name exactly as TapeKit resolves `<#ID>.<processor number>.tape` today, then read the declaration file from the site store (`openSite()` → `site.get('/.tape/api.json')`). Readable vanity names such as `translate.tape` do not exist in the current naming rules and are **not** required by this draft; examples use the endpoint display form (`#<#ID>@<processor number>`).

---

## 3. Three modes / 三种活不要混

| Mode | When | Mail? | Pay? |
|---|---|---|---|
| A `onchain-file` | quotes, configs | no | no — read a chain file |
| B `onchain-state` | scores, receipts | optional | optional — state lives in the container |
| C `hybrid` | translate, models | yes | optional — an operator watches the inbox |

Mode A MUST NOT require TAP-10. If the answer is a file, read the file.

---

## 4. Content kinds / 内容类型

**How this sits on TAP-10 v1.0.** The v1.0 content rules recognize exactly one content kind: `kind: "message"` (§6). This draft proposes **adding content kinds** — `deweb.req/v0` / `deweb.res/v0` are working labels, not a new hub opcode. Under the current strict decode rules an unrecognized `kind` decodes as `unsupported` and **MUST NOT** be interpreted further, so on today’s clients a request degrades safely to “unsupported” instead of rendering as a broken message. Final kind strings and their versioning (a `v` field vs a `/vN` suffix) are the maintainers’ call.

```json
{
  "kind": "deweb.req/v0",
  "id": "0x…",
  "nonce": "0x…",
  "method": "translate",
  "params": { "text": "hello", "from": "en", "to": "zh" },
  "replyTo": "#8801@0",
  "deadline": 122900020
}
```

```json
{
  "kind": "deweb.res/v0",
  "id": "0x…",
  "ok": true,
  "result": { "text": "你好" }
}
```

Rules:

1. `id` is computed by the caller — a commitment to the request, not a random value — and the response MUST echo the same `id`.
2. `id` MUST be `sha256("deweb.req/v0" ‖ endpointID(from) ‖ endpointID(to) ‖ nonce ‖ utf8(JCS(params)))` — 32 bytes, written `0x`-prefixed lowercase hex — so both sides recompute the same bytes: the caller to derive `id`, the recipient to check the request it opened against it. `‖` is byte concatenation and ASCII labels are raw bytes, as in TAP-10 §1:
   - `endpointID(x)` is the 32-byte endpoint ID of TAP-10 §2.2 (`uint32(0) ‖ uint64(chainId) ‖ container`) — never the display form. `from` is the sending container and `to` the recipient container; the caller knows both, and the recipient reads both from the verified entry (TAP-10 §5.3).
   - `nonce` is a request field defined here: exactly 32 bytes, fresh for every request and never reused, carried as `0x`-prefixed lowercase hex (66 characters) and hashed as its 32 raw bytes.
   - `JCS(params)` is the RFC 8785 (JCS) serialization of `params`, hashed as its UTF-8 bytes; JCS is mandatory — with an “or equivalent” canonicalization the two sides could hash different bytes.
   The four leading fields are fixed-length (12 + 32 + 32 + 32 bytes) and `params` is the only variable-length field, last in the preimage, so every field boundary is unambiguous. `method`, `replyTo` and `deadline` stay outside the preimage: `id` commits to the endpoint pair, the `nonce` and `params`.
3. The TAP-10 envelope (payload format `0x02`, encryption, digest, inbox index, the 16,000-byte payload cap) is unchanged. See TAP-10 §5.
4. `deadline` is a block height on the chain the request was sent on (the sender’s chain). A caller MUST compare it against that chain’s finalized height (TAP-10 §7) and treat silence after it as failure.
5. Assets MAY be TAP-10 attachments (§6.1 formats). This draft does **not** specify an escrow contract; until one exists, payment is out of band or omitted. Baseline cost of any request: a send is a wallet transaction from the sender’s wallet (TAP-10 §10 — signature + gas; roughly 0.004 USD at typical BNB fee levels, §11), and the request metadata (who, when, size) is public and permanent. “Free” means no price attached, not no cost.
6. A response SHOULD carry, as the TAP-10 `ref` of its message, the request’s message ID (the existing reply mechanism — `ref` is public and unverified, §7), in addition to echoing `id`. Conversation grouping stays by endpoint pair, as §7 requires.

`deweb.req/v0` / `deweb.res/v0` avoid claiming TAP-11. If this is accepted, maintainers may rename to `tap-NN/req`.

---

## 5. Client sketch / 客户端要点

1. Resolve name → container. Read `/.tape/api.json` from the site store.
2. If mode A (or method is free file read): read the chain file (`site.get`), return.
3. Else seal a TAP-10 message to the provider endpoint with `deweb.req/v0`.
4. Watch own inbox for `deweb.res/v0` with the same `id`, opened under TAP-10 rules (digest, encryption, multi-node agreement).
5. If `deadline` passes with no matching res: fail. Do not invent a result from a single node.

TapeKit should keep key handling in `@tapekit/send`, not in the page kernel.

---

## 6. Out of scope / 先不做

- Hub contract changes
- Cross-chain bridging
- A required token or fee
- Kernel executing untrusted operator code
- Claiming a TAP number

---

## 7. Ask of maintainers / 请维护者决定

- Is a content-kind document the right home (`send/` next to TAP-10), or a separate TAP number — and should it fold into the TAP-10 v1.0 rewrite rather than a new number?
- If a number is assigned, please assign it; this draft will not self-number.
- May site-declared files live under `/.tape/` (a §7.8 carve-out), or should we pick a non-reserved path?
- Should TapeKit later grow `window.tape.call` as a kernel helper that only sends these kinds?
