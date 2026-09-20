# DeWEB request/response + WebMCP (unofficial draft)

**This is not an official TapeOut TAP.**  
TapeOut publishes **TAP-10 (TapeSend)** in [TapeOutProtocol/TapeKit](https://github.com/TapeOutProtocol/TapeKit) (the v1.0 DeWEB messaging spec is being rewritten and will be published separately). There is no TAP editor and no assigned TAP-11 / TAP-12. Working names in this repo are discussion labels only. Numbering belongs to TapeKit maintainers.

This repository is a **design draft + off-chain working case**:

1. Request/response **content** on top of TAP-10 (container → container mailbox).
2. A DeWEB site file `/.tape/mcp.json` so TapeKit can register [WebMCP](https://github.com/webmachinelearning/webmcp) tools when a site is opened as a **top-level origin**.

Status of a companion playground: chain identity, TAP-10 send, and BEM escrow were **simulated locally**. Translation/price used ordinary off-chain APIs. Nothing here is a mainnet client.

Spec text: CC0 (`LICENSE-SPEC`). Any later code: MIT (`LICENSE`).

## Why this exists

DeWEB sites are static files in a circuit container. SPEC §7.5 asks shells to block off-chain requests by default and §7.8 makes a Service Worker gateway block and record them — while the current gateway (0.3.x) allows and records non-script off-chain traffic instead (its CSP allows `https:`/`wss:`; scripts stay on-chain). Which model to target is one of the questions left to maintainers. TAP-10 already gives containers an encrypted inbox. Missing pieces for “this site is an API” and “this site is a page of tools for an agent”:

- A **content kind** so a TAP-10 payload is a call, not a chat letter.
- A **discovery file** `/.tape/api.json` so a container’s endpoint (`#<#ID>@<processor number>`) can list methods. (Both drafts flag two pre-existing constraints to maintainers: `/sw.js` and the whole `/.tape/` prefix are gateway-reserved — SPEC §7.8 — and readable vanity names don’t exist yet.)
- Optional **payment** as TAP-10 attachments / a future escrow — **not required** for WebMCP.
- A **tool list** `/.tape/mcp.json` so TapeKit can register WebMCP tools (`document.modelContext`, Chrome origin trial) on a real gateway origin (an official gateway such as tapekit.org, HashPort, or self-hosted). This draft’s venue is the top-level origin; WebMCP itself also covers same-origin iframes (cross-origin needs `Permissions-Policy: tools`), and the opaque-origin sandbox viewer can never be a provider.

## Documents

| File | What it proposes |
|---|---|
| [docs/request-response.md](docs/request-response.md) | TAP-10 content: req / res, `/.tape/api.json`, three modes |
| [docs/webmcp-sites.md](docs/webmcp-sites.md) | `/.tape/mcp.json`, TapeKit polyfill, no payment required |

Please discuss on TapeKit (open an issue there; Discussions are not enabled) before treating any number as assigned.

## What is already real

- [TAP-10 TapeSend](https://github.com/TapeOutProtocol/TapeKit/tree/main/send) — live hub on BNB Smart Chain
- DeWEB / TapeKit kernel — sites as chain files
- WebMCP — W3C WebML CG draft / Chrome origin trial (`document.modelContext`)

## What this draft does **not** claim

- An official TAP-11 or TAP-12
- A deployed escrow contract
- TapeKit already injecting `document.modelContext`
- Mainnet operators watching inboxes for `tap-11/req`
