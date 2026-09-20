# DeWEB request/response + WebMCP (unofficial draft)

**This is not an official TapeOut TAP.**  
TapeOut currently publishes **TAP-10 (TapeSend)** in [TapeOutProtocol/TapeKit](https://github.com/TapeOutProtocol/TapeKit). There is no TAP editor and no assigned TAP-11 / TAP-12. Working names in this repo are discussion labels only. Numbering belongs to TapeKit maintainers.

This repository is a **design draft + off-chain working case**:

1. Request/response **content** on top of TAP-10 (container → container mailbox).
2. A DeWEB site file `/.tape/mcp.json` so TapeKit can register [WebMCP](https://github.com/webmachinelearning/webmcp) tools when a site is opened as a **top-level origin**.

Status of a companion playground: chain identity, TAP-10 send, and BEM escrow were **simulated locally**. Translation/price used ordinary off-chain APIs. Nothing here is a mainnet client.

Spec text: CC0 (`LICENSE-SPEC`). Any later code: MIT (`LICENSE`).

## Why this exists

DeWEB sites are static files in a circuit container. Off-chain `fetch` is blocked by default. TAP-10 already gives containers an encrypted inbox. Missing pieces for “this site is an API” and “this site is a page of tools for an agent”:

- A **content kind** so a TAP-10 payload is a call, not a chat letter.
- A **discovery file** `/.tape/api.json` so a name like `translate.tape` lists methods.
- Optional **payment** as TAP-10 attachments / a future escrow — **not required** for WebMCP.
- A **tool list** `/.tape/mcp.json` so TapeKit can call `document.modelContext.registerTool` on a real origin (HashPort HTTPS). Sandbox iframes cannot be WebMCP providers.

## Documents

| File | What it proposes |
|---|---|
| [docs/request-response.md](docs/request-response.md) | TAP-10 content: req / res, `/.tape/api.json`, three modes |
| [docs/webmcp-sites.md](docs/webmcp-sites.md) | `/.tape/mcp.json`, TapeKit polyfill, no payment required |

Please discuss on TapeKit before treating any number as assigned.

## What is already real

- [TAP-10 TapeSend](https://github.com/TapeOutProtocol/TapeKit/tree/main/send) — live hub on BNB Smart Chain
- DeWEB / TapeKit kernel — sites as chain files
- WebMCP — W3C / Chrome origin trial (`document.modelContext`)

## What this draft does **not** claim

- An official TAP-11 or TAP-12
- A deployed escrow contract
- TapeKit already injecting `document.modelContext`
- Mainnet operators watching inboxes for `tap-11/req`
