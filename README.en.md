# K12 Space Automation

[中文](README.md) | English

K12 Space Automation is a local K12 workspace automation console for mailbox pool management, email OTP flows, K12 workspace join/switch tasks, Sub2API imports, access-token checks/repairs, and account JSON export.

This repository contains source code, documentation, configuration templates, and lock files only. It does not include real runtime configuration, tokens, cookies, mailbox refresh tokens, account JSON files, or task data by default. Do not commit real credentials or local runtime data, even when the repository is private.

## Features

- Mailbox pool management: import, select, delete, status marking, and retry.
- OTP handling: mailbox URL, manual OTP, SMSBower Gmail, and Emailnator Gmail.
- K12 flow: login, join or switch K12 workspace, and read K12-context access tokens.
- Sub2API: OAuth import, noRT import, account liveness check, and access-token repair.
- JSON output: SUB2API and CPA account JSON formats.
- Data migration: import/export local configuration, mailbox pool, tasks, and token data packages.
- Task management: batch start, cancel, retry, clear failed tasks, pagination, status, and logs.

## Architecture

- `src/`: Vue 3 web console entry and UI logic.
- `server/index.ts`: local HTTP API server for task scheduling, configuration persistence, mailbox state, K12 flows, Sub2API calls, and JSON output.
- `codex_register/`: lower-level automation toolkit for registration, OAuth, mailboxes, SMS, Sentinel, Sub2API, CPA, and standalone web tools.
- `codex_register/config.example.json`: committable configuration template. Copy it to `codex_register/config.json` before filling real values.
- `public/`, `index.html`, `vite.config.ts`: Vite frontend assets and build configuration.
- `data/`, `json/`, `pool_tokens.txt`, `config.json`: runtime data and local configuration. These paths are ignored and are not part of the repository payload.

## Requirements

- Node.js 20+, Node.js 22+ recommended.
- npm 10+.
- Network access to the services you configure.
- Optional HTTP or SOCKS proxy.

## Install and Run

Install dependencies:

```bash
npm install
```

Start the API server and web console in development mode:

```bash
npm run dev
```

Default URLs:

- Web console: `http://127.0.0.1:5174/`
- API server: `http://127.0.0.1:8796/`

Common scripts:

```bash
npm run server    # Start the local API server only
npm run frontend  # Start the Vite web console only
npm run build     # Type-check and build the frontend
npm run preview   # Preview the built frontend
npm run start     # Start the local API server
```

## Configuration

The main configuration is saved from the Settings page in the web console. Runtime writes:

- `data/config.json`: current console configuration.
- `config.json`: root-level compatibility configuration for legacy flows.

Standalone tools under `codex_register/` read `codex_register/config.json`. For first use:

```bash
cp codex_register/config.example.json codex_register/config.json
```

Common fields:

| Field | Description |
| --- | --- |
| `port` | API server port, default `8796`. |
| `defaultProxyUrl` | Proxy for OpenAI/Auth requests. Supports `direct`, HTTP, and SOCKS. |
| `openaiProxyUrls` | Rotating proxy list for OpenAI/Auth requests. |
| `mailApiBaseUrl` | Base URL for four-part mailbox OTP APIs. |
| `workspaceIds` | K12 workspace ID list. |
| `route` | K12 workspace route, `request` or `accept`. |
| `taskConcurrency` | Task concurrency. |
| `runWorkspaceJoin` | Whether to run the K12 join/switch flow. |
| `runSub2Api` | Whether to import accounts into Sub2API. |
| `sub2apiNoRtMode` | Whether to use noRT import mode. |
| `sub2apiUrl` | Sub2API service URL. |
| `sub2apiEmail` | Sub2API admin email. |
| `sub2apiPassword` | Sub2API admin password. |
| `sub2apiGroupName` | Target Sub2API group. |
| `sub2apiProxyName` | Sub2API proxy name. |
| `sub2apiAccountPriority` | Sub2API account priority. |
| `sub2apiConcurrency` | Sub2API import concurrency. |
| `sub2apiAutoRefillEnabled` | Enable automatic Sub2API refill. |
| `sub2apiRefillGroupName` | Group checked by automatic refill. |
| `sub2apiRefillThreshold` | Automatic refill threshold. |
| `sub2apiRefillEmailCount` | Mailbox count used for automatic refill. |
| `sub2apiRefillIntervalMs` | Automatic refill interval. |
| `sub2apiRefillDeepCheckEnabled` | Enable deep liveness checks for automatic refill. |
| `gmailMailProvider` | Dynamic Gmail provider, `smsbower` or `emailnator`. |
| `smsBowerMailEnabled` | Enable SMSBower Gmail OTP. |
| `smsBowerApiKey` | SMSBower API key. |
| `smsBowerMailBaseUrl` | SMSBower mail API URL. |
| `smsBowerMailService` | SMSBower mail service name. |
| `smsBowerMailDomain` | SMSBower mail domain. |
| `smsBowerMailMaxPrice` | Maximum SMSBower mail price. |
| `smsBowerGmailFissionEnabled` | Enable SMSBower Gmail fission child-mailbox tasks. |
| `smsBowerGmailFissionCount` | Fission count per Gmail mailbox. |
| `emailnatorBaseUrl` | Emailnator service URL. |
| `emailnatorEmailType` | Emailnator mailbox type. |
| `requireChatgptAccountId` | Require ChatGPT account ID in access tokens. |
| `tokenOut` | Access-token output file, default `pool_tokens.txt`. |
| `jsonOutDir` | Account JSON output directory, default `json/`. |
| `jsonOutFormat` | JSON output format, `sub2api` or `cpa`. |

## Task Flow

1. Open the web console.
2. Fill Settings and confirm proxy, workspace, Sub2API, and OTP settings.
3. Import a mailbox pool or enable dynamic Gmail OTP.
4. Configure task count, concurrency, K12 workspace flow, Sub2API/noRT, and JSON output.
5. Start tasks and inspect status, logs, access-token summaries, and output paths.
6. Retry failed tasks after reading logs, or lower concurrency before rerunning.
7. Use data import/export for local state migration. Do not use Git for runtime data.

## Sensitive File Boundary

The following files or directories may contain passwords, API keys, mailbox refresh tokens, access tokens, cookies, OAuth data, mailbox pools, account JSON files, or task logs. They should not be committed, even to a private repository:

```text
config.json
codex_register/config.json
data/
json/
pool_tokens.txt
auth/
k12-basic-auth*
.env
.env.*
*.pem
*.key
*.crt
*.log
```

Before committing, check:

```bash
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

The second command should produce no output. `codex_register/config.example.json` is a template and may be committed.

## FAQ

### `EmailOtpValidate wrong_email_otp_code`

OpenAI rejected the submitted email OTP. Common causes are stale mailbox messages, ad emails containing six-digit numbers, or expired OTPs. Use another mailbox, clean old messages, or verify with manual OTP mode.

### Redirected to `accounts.google.com`

The mailbox was routed to Google OAuth instead of the normal email OTP flow. This tool does not automate Google account login. Use a mailbox that can proceed through email OTP.

### `CreateAccount HTTP 500 Request timeout`

This is usually caused by upstream instability, a slow proxy, request timeout, or high concurrency. Retry, change proxy, or lower concurrency.

### Cancel does not stop instantly

Tasks stop at the next cancellable boundary. If a network request is in progress, status updates may wait until that request returns or times out.

### Sub2API import fails

Verify `sub2apiUrl`, `sub2apiEmail`, `sub2apiPassword`, `sub2apiGroupName`, and proxy settings first. Then inspect the HTTP status and response summary in task logs.

### JSON output is missing

Verify `jsonOutDir`, `jsonOutFormat`, access-token availability, and write permission for the target directory.

## Build Verification

Run before committing:

```bash
npm run build
npx tsc --noEmit -p codex_register/tsconfig.json
git status --short --ignored
git ls-files | rg '(^|/)(data|json|auth|pool_tokens|config\.json|k12-basic-auth|\.env|.*\.pem|.*\.key)'
```

`npm run build` and `npx tsc` should pass. The sensitive-file check should produce no output.

## License

This project is licensed under the MIT License. See `LICENSE`.
