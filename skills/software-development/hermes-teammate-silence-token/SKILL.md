---
name: hermes-teammate-silence-token
description: "Fix teammate silence-token mismatches in group chat rounds."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows]
metadata:
  hermes:
    tags: [hermes, teammate, bot-mode, silence-token, group-chat, troubleshooting]
---

## When to Use

A teammate bot posts successfully in a group chat but the Desktop App UI marks the turn "failed · unknown" instead of swallowing it silently. The bot's final response is a silence-token variation (e.g. "PASS", "SILENT\n", "SILENT.") that doesn't exactly match the parser's expected token.

---

# Hermes teammate silence-token alignment

Use when diagnosing why a Hermes teammate bot appears to "fail" in group chat rounds when it's actually responding successfully — the UI marks its silence token as "failed · unknown" because the token doesn't exactly match what the parser expects.

## Diagnosis procedure
1. Read the teammate's SOUL.md (~/.hermes/profiles/<name>/SOUL.md) — check whether it instructs exact silence-token usage when the bot has nothing to add.
2. Read the teammate's config.yaml and profile.yaml — confirm ui_meta.hermes-bots block exists (Bot Mode registered) and platform toolsets include teammates.
3. Read gateway/desktop logs (~/.hermes/logs/desktop.log, ~/.hermes/logs/gateway.log) — search for the bot's name near "failed · unknown" or silence-token mentions.
4. Compare the bot's actual response against the supported silence tokens: [SILENT], SILENT, NO_REPLY, NO REPLY. Whitespace and case are normalized by the spec, but the Desktop App UI does exact matching — a trailing newline, period, or "PASS" variation breaks the parser.
5. Patch the teammate's SOUL.md to add a GROUP CHAT SILENCE instruction: end the turn with exactly [SILENT] when nothing to add — no punctuation, no newline, no variation.

## Pitfalls
- Do not accept "SILENT\n", "SILENT.", "PASS", or lowercase variants as correct — the UI parser does exact whole-response matching; any deviation is delivered as a visible message or marked failed.
- The bot mode protocol section is injected only into canonical "Bot Chat" sessions; regular sessions and SOUL.md stay untouched, so SOUL.md is the right place to pin the exact token.
- Silence is a delivery decision only — the turn stays in the transcript, nothing is delivered to the chat, and the UI won't flag it as failed when the token matches exactly.

## Supported silence tokens (exact match required)
- [SILENT]
- SILENT
- NO_REPLY
- NO REPLY
