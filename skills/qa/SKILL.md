---
name: qa
description: E2E acceptance criteria verification using the agent-browser CLI (vercel-labs/agent-browser). Runs after the tdd skill to confirm implemented behavior works in the browser. Use when user wants to QA a ticket, verify acceptance criteria end-to-end, run browser tests, or says "qa this", "verify e2e", "check acceptance criteria", "run qa", or after TDD is complete.
---

# QA — E2E Acceptance Criteria Verification

Runs after `tdd` to confirm the implemented feature satisfies every acceptance criterion in the browser, not just in unit tests. Uses the [`agent-browser`](https://github.com/vercel-labs/agent-browser) CLI — a fast native Rust browser automation tool.

## Quick Start

```bash
agent-browser open http://localhost:3000/login
agent-browser snapshot -i
agent-browser fill @<email-ref> "$USER_EMAIL"
agent-browser fill @<pw-ref>    "$USER_PW"
agent-browser click @<submit-ref>
agent-browser wait --load networkidle
```

---

## Workflow

### 1. Install agent-browser (if missing)

```bash
which agent-browser || npm install -g agent-browser && agent-browser install
```

`agent-browser install` downloads Chrome for Testing on first run.

### 2. Fetch Acceptance Criteria

Load the Jira ticket for the current task (`mcp_mcp-atlassian_jira_get_issue`). Extract every acceptance criterion. If no ticket is available, ask the user to list criteria before continuing.

### 3. Read Credentials from .env.local

```bash
source .env.local   # exposes $user and $pw
```

Credentials are stored in `.env.local` under the keys `user` (email) and `pw` (password). Never hardcode them in commands.

### 4. Login

```bash
agent-browser open http://localhost:3000/login
agent-browser wait --load networkidle
agent-browser snapshot -i   # discover email/password/submit refs
# then fill and submit using the refs from snapshot:
agent-browser fill @eN "$user"
agent-browser fill @eM "$pw"
agent-browser click @eK    # submit / login button
agent-browser wait --load networkidle
```

If the login page redirects to a different route, wait for it:
```bash
agent-browser wait --url "**/*"
```

### 5. Start Video Recording

Before testing, start a video recording so the entire QA session is captured:

```bash
mkdir -p docs/tickets/<TICKET-ID>
agent-browser record start docs/tickets/<TICKET-ID>/<TICKET-ID>-qa.webm
```

The recording will be stopped and saved at the end (step 8).

### 6. E2E Verification Loop

Before starting, print the **full list of acceptance criteria** as bullet points so the user can see what will be tested:

```
Acceptance Criteria for <TICKET-ID>:
- AC1: <full text of criterion 1>
- AC2: <full text of criterion 2>
- AC3: <full text of criterion 3>
...
```

Then, for each criterion, announce it and show its result immediately:

```
▶ AC1: <full text of criterion>
  → navigating to <route>...
  → performing action...
  ✅ PASS — <what was observed>

▶ AC2: <full text of criterion>
  → navigating to <route>...
  → checking visibility...
  ❌ FAIL — expected: <X>, observed: <Y>
     Screenshot: /tmp/qa-<ticket-id>-2.png
```

Steps per criterion:
```
agent-browser open <route>
agent-browser wait --load networkidle
agent-browser snapshot -i          # discover refs
agent-browser click/fill/select    # user actions using refs
agent-browser get text / is visible / screenshot  # assert outcome
```

**Recon before acting** — always run `snapshot -i` after navigation to discover refs before interacting.

### 7. Manual Testing Report

After the automated checks, navigate agent-browser to the page that contains the new feature/fix and produce a **Manual Testing Guide** so the user can verify it themselves:

```bash
agent-browser open <route-of-new-feature>
agent-browser wait --load networkidle
agent-browser screenshot /tmp/qa-manual-<ticket-id>.png
```

Then output:

```
## Manual Testing Guide — <ticket id>

Logged-in as: <user from .env.local>
Page: <URL>
Screenshot: /tmp/qa-manual-<ticket-id>.png

### Steps to verify

1. Go to <URL>
2. <Action> — expected: <outcome>
3. <Action> — expected: <outcome>
...

### Edge cases to try

- <edge case 1>
- <edge case 2>
```

The guide should map 1-to-1 to the acceptance criteria from the ticket, written as human-readable steps a QA engineer can follow without any code knowledge.

### 8. Stop Video Recording & Save

Stop the recording and save the WebM video to the ticket docs folder:

```bash
agent-browser record stop
```

The video is saved at `docs/tickets/<TICKET-ID>/<TICKET-ID>-qa.webm`. It can be opened in any browser or video player.

---

## Command Reference

| Goal | Command |
|------|---------|
| Start video recording | `agent-browser record start <path.webm>` |
| Stop video recording | `agent-browser record stop` |
| Navigate | `agent-browser open <url>` |
| Wait for load | `agent-browser wait --load networkidle` |
| Accessibility tree | `agent-browser snapshot -i` |
| Screenshot | `agent-browser screenshot /tmp/qa.png` |
| Annotated screenshot | `agent-browser screenshot --annotate` |
| Click | `agent-browser click @eN` |
| Fill input | `agent-browser fill @eN "value"` |
| Press key | `agent-browser press Enter` |
| Select dropdown | `agent-browser select @eN "option"` |
| Get text | `agent-browser get text @eN` |
| Check visibility | `agent-browser is visible @eN` |
| Console logs | `agent-browser console` |
| Page errors | `agent-browser errors` |
| Close browser | `agent-browser close` |

---

## Checklist

- [ ] `agent-browser` installed and Chrome ready
- [ ] Jira ticket fetched; acceptance criteria listed
- [ ] Dev server reachable at `http://localhost:3000`
- [ ] Credentials read from `.env.local` (`$user`, `$pw`)
- [ ] Logged in successfully
- [ ] Video recording started
- [ ] Full AC list printed as bullet points before verification
- [ ] Each criterion exercised with live PASS/FAIL output
- [ ] Screenshots captured for failures
- [ ] Manual Testing Guide produced
- [ ] Video saved to `docs/tickets/<TICKET-ID>/<TICKET-ID>-qa.webm`
