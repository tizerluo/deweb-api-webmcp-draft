# Draft: request/response content on TAP-10

Status: **unofficial draft** (2026-09-20)  
Working name used in discussion: “TapeAPI” / “TAP-11” — **number not assigned**  
Depends on: TAP-10 TapeSend ([TapeKit/send](https://github.com/TapeOutProtocol/TapeKit/tree/main/send))  
Does not change: TapeKit SPEC §15.1 promises (names, verification, node agreement)

English is the intended normative language if this is later merged. Chinese follows.

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

Clients resolve a vanity name the same way TapeKit already resolves `*.tape`, then `kernel.get("/.tape/api.json")`.

---

## 3. Three modes / 三种活不要混

| Mode | When | Mail? | Pay? |
|---|---|---|---|
| A `onchain-file` | quotes, configs | no | no — `kernel.get` a file |
| B `onchain-state` | scores, receipts | optional | optional — state lives in the container |
| C `hybrid` | translate, models | yes | optional — an operator watches the inbox |

Mode A MUST NOT require TAP-10. If the answer is a file, read the file.

---

## 4. Content kinds / 内容类型

These are **payload kinds inside TAP-10**, not a new hub opcode.

Working labels (replace if maintainers assign a TAP number):

```json
{
  "kind": "deweb.req/v0",
  "id": "0x…",
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

1. `id` is chosen by the caller; the response MUST echo the same `id`.
2. `id` SHOULD be `sha256("deweb.req/v0" ‖ from ‖ to ‖ nonce ‖ params)` or equivalent, so the caller can prove the request.
3. The TAP-10 envelope (endpoint, encryption, digest, inbox index) is unchanged. See TAP-10.
4. `deadline` is a block height on the sender’s chain. After it, a caller MUST treat silence as failure.
5. Assets MAY be TAP-10 attachments. This draft does **not** specify an escrow contract. Until one exists, payment is out of band or omitted.

`deweb.req/v0` / `deweb.res/v0` avoid claiming TAP-11. If this is accepted, maintainers may rename to `tap-NN/req`.

---

## 5. Client sketch / 客户端要点

1. Resolve name → container. Read `/.tape/api.json`.
2. If mode A (or method is free file read): `kernel.get`, return.
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

- Is a content-kind document the right home (`send/` next to TAP-10), or a separate TAP number?
- If a number is assigned, please assign it; this draft will not self-number.
- Should TapeKit later grow `window.tape.call` as a kernel helper that only sends these kinds?
