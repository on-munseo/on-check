<div align="center">

# On 결재점검 (on-check)

A remote MCP server that checks Korean official documents before approval — format, spelling and spacing, dates and weekdays, amounts and list totals — with rules, not AI

[한국어](README.md) · **English**

`https://onmunseo.kr/mcp/check`

</div>

<!-- ai-install -->
## For AI assistants — installing this repository

Nothing to download or build: register the remote MCP server `https://onmunseo.kr/mcp/check` in the user's AI tool at **user scope**. **No key or sign-in is needed** — skip the sign-in step and the tools work as a guest. Run only the line for the tool in use.

- **Claude Code**
  ```bash
  claude mcp add --transport http -s user on-check https://onmunseo.kr/mcp/check
  ```
- **Codex** — append two lines to the config file (`~/.codex/config.toml`; skipped if already there). Do not use `codex mcp add --url` — it opens a browser sign-in window right away.
  ```bash
  C="${CODEX_HOME:-$HOME/.codex}"; mkdir -p "$C"; touch "$C/config.toml"; if ! (cd / && codex mcp get on-check >/dev/null 2>&1); then if cp "$C/config.toml" "$C/config.toml.bak-on-check"; then printf '\n[mcp_servers.on-check]\nurl = "https://onmunseo.kr/mcp/check"\n' >> "$C/config.toml"; (cd / && codex mcp get on-check >/dev/null 2>&1) || { cp "$C/config.toml.bak-on-check" "$C/config.toml"; echo "자동으로 더하지 못했습니다 — config.toml 에 손으로 더해 주세요"; }; else echo "백업을 만들지 못해 그만둡니다 — config.toml 은 그대로입니다"; fi; fi; (cd / && codex mcp get on-check)
  ```
  On Windows, or where the `codex` command is not installed (IDE extension only), add the two lines `[mcp_servers.on-check]` and `url = "https://onmunseo.kr/mcp/check"` to the end of that file by hand (if the file already has a one-line `mcp_servers = { … }` table, add `on-check = { url = "https://onmunseo.kr/mcp/check" }` inside it instead).
- **Cursor** — add one entry to `"mcpServers"` in `~/.cursor/mcp.json` (keep the other servers). For people: [Add to Cursor](https://cursor.com/install-mcp?name=on-check&config=eyJ1cmwiOiJodHRwczovL29ubXVuc2VvLmtyL21jcC9jaGVjayJ9).
  ```json
  "on-check": { "url": "https://onmunseo.kr/mcp/check" }
  ```
- **VS Code** — for people: [Add to VS Code](https://insiders.vscode.dev/redirect/mcp/install?name=on-check&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fonmunseo.kr%2Fmcp%2Fcheck%22%7D).
  ```bash
  code --add-mcp '{"name":"on-check","type":"http","url":"https://onmunseo.kr/mcp/check"}'
  ```
- **Claude Desktop · claude.ai** — done by the person in the app: Settings → Connectors → Add custom connector → name `On 결재점검`, URL `https://onmunseo.kr/mcp/check` → Connect.

The tools do not attach to the conversation that registered them. When done, tell the user: "Restart the tool or open a new conversation to see the On 결재점검 tools." If `claude mcp list` (Codex: `codex mcp list`) lists `on-check`, it is registered. Full steps (Korean): [llms-install.md](llms-install.md)

<!-- ai-install -->

Send a document (HWP/HWPX, PDF, Word) or pasted text (`text`). It is checked for official-document format (date and time notation, attachments, the closing mark), spelling and spacing, dates and weekdays, amounts, list totals and numbering order; each finding comes back with the paragraph number, what to change into what, and why. The checks are rule-based (no AI), and the document itself is not changed.

This repository holds only the setup guide and configuration files (there is no web page here — to use a screen, open the [On 문서 web editor](https://onmunseo.kr/app/)). The engine and server code of the service are not public. The service and its tools speak Korean.

## Comparing with other tools

- **Rules, not AI:** unlike an AI spell checker, the same text always gives the same result, and every finding comes with the paragraph number, what → what, and the reason. No fix is made up.
- **On one server:** it works together with text extraction, document writing and statute citation checks (On 근거) on the same On 문서 server.
- **Where documents go:** documents sent to this address are processed only in memory on the On 문서 server (a data centre in Korea) and discarded afterwards; they are not sent to any outside AI company. Send only public documents or your own to this server-side address; handle internal documents of your organisation with the browser-only features of the [web editor](https://onmunseo.kr/app/) (open, edit and save HWP/HWPX, the pre-approval check, masking personal data) — attachment text extraction, OCR and HWPX creation in the web editor still go through the server.

**Jump to:** [Comparing](#comparing-with-other-tools) · [Why](#in-short--why-use-it) · [Tools](#what-it-does) · [Connect](#connect) · [Examples](#examples) · [Measured results](#measured-results) · [Guest and key](#guest-and-key) · [Safety](#safety) · [Good to know](#good-to-know)

## In short — why use it

| Common pain | With On 결재점검 |
|---|---|
| You open a calendar to check dates and weekdays | Impossible dates and wrong weekdays are flagged (e.g. “2026. 11. 31.” → November has 30 days) |
| You add up amounts and list totals again by hand | Amount notation and list totals are checked (table totals are re-calculated in the web editor) |
| You look up official notation (date dots, attachments, the closing “끝.”) every time | Format rules flag them and suggest the fix |
| An AI chat gives a different review each time you ask | The checks are rules, so the same text always gives the same result |

**When you do not need it:** For one or two lines, reading it yourself may be faster. The checks miss things (see Measured results), so they do not replace a final human review.

## What it does

| Tool | What it does | Guest |
|---|---|---|
| `on_check_text` | Checks format, spelling and spacing, dates and weekdays, amounts, list totals and numbering, and returns each finding (paragraph, from → to, reason) by category. Text via `text` (up to 300,000 characters); files via `file_base64` or `file_url` | yes |

- The guest tool was checked on 2026-11-26 by calling `tools/list` on the live server as a guest.
- Items with `check_only: true` are places where the rules do not propose a fix directly (a person should look). Re-calculating table totals and cross-checking related documents are done in the On 문서 web editor (the tool result says so).

## Connect

**Claude Code** — one line in the terminal (`-s user` makes it available in every folder):

```bash
claude mcp add --transport http -s user on-check https://onmunseo.kr/mcp/check
```

**Claude Code plugin** — one line (Claude Code 2.1.275 or later):

```bash
claude plugin install on-check --marketplace on-munseo/on-check
```

In every app, the conversation in which you register the server does not get the tools yet. Restart the app once (Claude Code: `/exit`, then `claude`; VS Code: a new chat) or open a new conversation, and the tools are there from then on. Without a key, skip the sign-in step and use it as a guest.

> To use all document tools and the law tools (On 근거) together, add `https://onmunseo.kr/mcp` instead of this bundle (recommended — [On 문서](https://github.com/on-munseo/on-munseo.github.io)). If you add both, the same tools appear twice.

### Claude Code — more

- With a key: in Claude Code open `/mcp` → `on-check` → Authenticate, and the On 문서 sign-in page opens. Enter your key there.
- To pass the key as a header: `claude mcp add --transport http -s user on-check https://onmunseo.kr/mcp/check --header "Authorization: Bearer <key>"`. This stores the key in `~/.claude.json`, so the sign-in page is the better way.
- Two-step plugin install (older versions): `claude plugin marketplace add on-munseo/on-check`, then `claude plugin install on-check@on-check`. In a Claude Code chat: `/plugin marketplace add on-munseo/on-check` → `/plugin install on-check@on-check`.
- Check: `claude mcp list` · remove: `claude mcp remove on-check`.

### claude.ai · Claude Desktop (connectors)

Settings → Connectors → “Add custom connector”: name `On 결재점검`, URL `https://onmunseo.kr/mcp/check`, then Connect. With a key, enter it on the On 문서 sign-in page that opens; without one you are a guest. Menu names differ by plan and version, and some plans may not have this menu.

To add it to Claude Desktop through the config file, use the `mcp-remote` bridge (needs Node.js). macOS `~/Library/Application Support/Claude/claude_desktop_config.json` · Windows `%APPDATA%\Claude\claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "on-check": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://onmunseo.kr/mcp/check"
      ]
    }
  }
}
```

To send a key as a header, append `"--header", "Authorization:${ON_AUTH}"` to `"args"` and add `"env": { "ON_AUTH": "Bearer <key>" }` (no space after the colon, so Windows does not split the argument). Restart Claude Desktop after saving.

### ChatGPT

1. Settings → Apps & Connectors → Advanced settings: turn on Developer mode (wording may differ by version).
2. Create a custom MCP server: name `On 결재점검`, URL `https://onmunseo.kr/mcp/check`, authentication “OAuth” if you have a key, otherwise “None”.
3. Enter your key on the On 문서 sign-in page, then turn the connector on from the tools (+) menu of a chat.

- ChatGPT receives a separate pass instead of your key (same permissions and daily quota). Revoking the key blocks the pass at once.
- Passing a file attached in the chat is not supported yet. Upload the file on On 문서 “자료 올리기” ([onmunseo.kr/start](https://onmunseo.kr/start)) and pass the 15-minute link; paste text directly.

### Codex

Without a key, add two lines to `~/.codex/config.toml`:

```toml
[mcp_servers.on-check]
url = "https://onmunseo.kr/mcp/check"
```

- `codex mcp add` opens the On 문서 sign-in window right away. With a key, enter it there; without one, close it (Ctrl-C) and use the two lines above.

### Cursor

`~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` in a project:

```json
{
  "mcpServers": {
    "on-check": {
      "url": "https://onmunseo.kr/mcp/check"
    }
  }
}
```

If your Cursor version cannot sign in to remote MCP servers (OAuth), add `"headers": { "Authorization": "Bearer <key>" }` (the key is then stored in the file).

### VS Code

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "on-check": {
      "type": "http",
      "url": "https://onmunseo.kr/mcp/check"
    }
  }
}
```

To send a key as a header without writing it in the file, let VS Code ask for it on first start:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "on-key",
      "description": "On 문서 key",
      "password": true
    }
  ],
  "servers": {
    "on-check": {
      "type": "http",
      "url": "https://onmunseo.kr/mcp/check",
      "headers": {
        "Authorization": "Bearer ${input:on-key}"
      }
    }
  }
}
```

### Try it with curl

List the tools (add `-H "Authorization: Bearer <key>"` if you have a key):

```bash
curl -s https://onmunseo.kr/mcp/check \
  -H "Content-Type: application/json" -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

Call `on_check_text` — sending the request from a file keeps quotes and line breaks intact:

```bash
cat > call.json <<'EOF'
{"jsonrpc": "2.0", "id": 1, "method": "tools/call", "params": {"name": "on_check_text", "arguments": {"text": "2026. 11. 31.(월) 14:00 ○○ 회의실에서 회의를 개최함."}}}
EOF
curl -s https://onmunseo.kr/mcp/check \
  -H "Content-Type: application/json" -H "Accept: application/json, text/event-stream" \
  --data-binary @call.json
```

One “날짜·요일” (date/weekday) finding came back: November 2026 has only 30 days (`check_only`) (2026-11-26, guest). The answer is the text in `result.content`. The server is stateless (Streamable HTTP, POST), so a single `tools/call` works without `initialize`. Opening the URL in a browser shows a short guide page.

## Examples

| If you say | This tool |
|---|---|
| “(attaching a draft) Check this before I submit it for approval” | `on_check_text` (file) |
| “Check whether the dates and weekdays below are right” | `on_check_text` (`text`) |
| “Fix only the findings that have a definite correction” | The AI applies `to` from items that are not `check_only` |

The AI decides which tool to call; these are typical cases.

## Measured results

| What | Test set | Result | Build · date · machine |
|---|---|---|---|
| Format checks | Test part of a self-made set of 120 official-format cases | recall 89.9% · precision 100% | 2026-11-11 · Mac M5 Pro |
| All pre-approval checks | Same set, test part | recall 56.0% · precision 100% | 2026-11-11 · Mac M5 Pro |
| Clean documents | Same set | 0 of 24 wrongly flagged (format and all checks) | 2026-11-11 |
| Spelling | Test part of 879 self-made sentences | recall 68.8% · precision 100% · 0 of 161 correct sentences flagged | 2026-11-11 · Mac M5 Pro |
| Warnings on real public documents | Warnings raised on 19,643 public documents, sampled and judged one by one (test part, weighted by warning volume) | share of correct warnings — format 91.8% · spelling 89.6% | 2026-11-13 · Windows i7-14700 |
| Security markings (all of On 문서) | 19,643 public documents · 40 lines with hidden markings (self-made) | ordinary documents wrongly blocked as confidential 0 · hidden markings found 90% · ordinary lines wrongly blocked 10% | 2026-11-13 · 2026-11-11 |

<sub>All values were measured by us, with the builds of 2026-11-11 and 11-13, and have not been re-run on the live build. The 120 cases, 879 sentences and 40 lines are our own tests and may be easier than real documents. The warnings on the 19,643 documents were judged by a single judge (an AI), on public documents only, with no real internal drafts. Recall = share of the places that should be found that were found; precision = share of findings that were right.</sub>

Weak spots: recall across all checks is 56.0%, so 44% of the places that should be found are missed (precision in the same test is 100%). Spelling recall is 68.8%. Treat results as a reference; a person makes the final call.

## Guest and key

|  | Guest (no key) | Key |
|---|---|---|
| How to connect | Add the URL with no authentication | Enter the key on the On 문서 sign-in page (OAuth), or send `Authorization: Bearer <key>` |
| Tools at this endpoint | 1 | 1 |
| Daily quota | 50 calls a day and 6 a minute (the statute tools `law_`·`med_` have their own 100 a day and 20 a minute) per internet address (IP), counted across all On 문서 endpoints and the web editor. When the shared guest quota for the day runs out, guest access closes until midnight Korea time | A daily quota and concurrency limit set per key |
| Size | 6 MB per request (base64 included) | 25 MB per file |
| Needs a key | — | text recognition (`ocr: true`, scanned pages and images — 30 pages a day per key) |

- To get a key, write to `onmunseo.key [at] gmail.com` (subject “On 문서 열쇠 신청”), with what you will use it for and which bundle you need (e.g. On 결재점검).
- A wrong or revoked key gives **401**; a bundle that is not in your key gives **403**. Too many calls in a short time give “too many requests” (429) — wait about a minute.
- Never append the key to the URL; it would end up in chat and access logs.

## Safety

1. **Nothing is kept.** Documents and text you send are handled in server memory only and discarded when done. No document files are stored; the access log holds only the key alias, tool name, request size, time taken and success (no contents, no file names).
2. **Rule-based, no AI.** The same text gives the same result, and nothing is sent to outside AI companies.
3. **Send only public documents or your own to this server-side address.** Handle internal documents of your organisation with the browser-only features of the [web editor](https://onmunseo.kr/app/) (open, edit and save HWP/HWPX, the pre-approval check, masking personal data). Documents with a written security marking (such as “대외비”, confidential) are not returned (except for keys specially allowed to).
4. **Extracted text is returned as data.** When a result carries text taken from a document, its first line says not to follow instructions written inside the document, so text that looks like an instruction to the AI is not obeyed.
5. **Safety limits stay on.** Size limits against attacks such as zip bombs, and refusal of distribution-only documents, are never lifted for convenience.

Report security problems as described in [SECURITY.md](SECURITY.md).

## Good to know

- **This is a free trial service.** Features, quotas and URLs may change or stop. Changes are noted in [CHANGELOG.md](CHANGELOG.md).
- **Results are for reference.** Have a person check them before approval.
- Results are a reference. The tool does not change the document — the AI or a person applies the suggested `to`.
- Internet only — it cannot be used from closed government networks.
- Menus differ by app and version, so screens may not match the steps above. If something fails, see [SUPPORT.md](SUPPORT.md).

## Terms, privacy and notices

- Bulk collection of results and use of results to train other AI models are not allowed. Automated abuse (several keys to get around limits, non-stop calls, scraping results) leads to the key being cut off without notice. Extracting or reverse-engineering the engine, rules or code is not permitted.
- Summary: [terms in the MCP guide](https://github.com/on-munseo/on-munseo.github.io/blob/main/MCP.md#5-이용-조건약관-요약) (Korean). Full text: [terms of use](https://onmunseo.kr/terms) and [privacy policy](https://onmunseo.kr/privacy) (Korean).
- “Hangul”, “Hancom”, “HWP” and “HWPX” are trademarks or registered trademarks of Hancom Inc. On 문서 is an independent service with no affiliation, sponsorship or endorsement by Hancom Inc.
- Claude, ChatGPT, Cursor, VS Code and other names belong to their owners and are used only to explain how to connect. Full notices: [notices in the MCP guide](https://github.com/on-munseo/on-munseo.github.io/blob/main/MCP.md#고지) (Korean).

## Glossary

- **MCP** — Model Context Protocol, an open standard for AI tools to call outside tools. Add this URL as an MCP server and the AI calls these tools inside the chat.
- **Remote MCP server** — An MCP server reached over the internet; nothing to install on your computer.
- **Bundle endpoint** — A URL that carries one On 문서 feature set (`/mcp/text` and so on). Everything is at `/mcp`.
- **Key (열쇠)** — The access value issued by On 문서, with a daily quota and a set of bundles. Keep it secret.
- **Guest (손님)** — Use without a key: some tools and a small daily quota, right away.
- **OAuth (sign-in page)** — You enter the key on the On 문서 sign-in page when connecting; the app receives a separate pass instead of the key.
- **HWPX** — The XML format of Korean Hangul documents (KS X 6101 OWPML); .hwp is the older format.
- **base64** — A way of writing a file as text, used to put a file inside JSON.
- **Upload link (15 minutes)** — Uploading a file on the On 문서 web ([onmunseo.kr/start](https://onmunseo.kr/start)) gives a link that works for 15 minutes; pass it as `file_url` from AI chats that cannot send files. AI tools that can run commands (Claude Code and others) can upload with one line, `curl -s --data-binary @file -H "x-file-name: name" https://onmunseo.kr/api/drop`, and get the same link (`url`) — faster than base64, and uploads do not count against the daily quota.
- **Recall · precision** — Recall: share of the places that should be found that were found. Precision: share of findings that were right.
- **check_only** — A finding with no direct fix from the rules; a person decides.

## Other On 문서 bundles

| Product | Repository | Endpoint (after `https://onmunseo.kr`) |
|---|---|---|
| **On 문서 (all)** | [on-munseo.github.io](https://github.com/on-munseo/on-munseo.github.io) — web editor and every MCP tool | `/mcp` |
| On 글꺼내기 | [on-text](https://github.com/on-munseo/on-text) | `/mcp/text` |
| On 결재점검 | this repository | `/mcp/check` |
| On 개인정보 가리기 | [on-mask](https://github.com/on-munseo/on-mask) | `/mcp/mask` |
| On HWPX 만들기 | [on-hwpx](https://github.com/on-munseo/on-hwpx) | `/mcp/hwpx` |
| On 공문서 쓰기 | [on-gov](https://github.com/on-munseo/on-gov) | `/mcp/gov` |
| On 글 다듬기 | [on-tidy](https://github.com/on-munseo/on-tidy) | `/mcp/tidy` |
| On 근거 | [on-geungeo](https://github.com/on-munseo/on-geungeo) | `/mcp/law` |
| On폭 글꼴 (fonts) | [napurrin/onpok-fonts](https://github.com/napurrin/onpok-fonts) — free fonts (SIL OFL 1.1) | — |

## License

Text is [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); configuration files (`server.json`, `.claude-plugin/`, `plugin/`) are MIT. The names “On 문서” and “On 결재점검” and the logo are not covered by these licenses. The engine and server code of the service are not in this repository and are not public. See [LICENSE](LICENSE).

---

Made by [napurrin](https://github.com/napurrin), who builds and maintains it while learning from Coco.
