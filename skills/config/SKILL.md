---
name: config
description: >
  Use to view or change yrsdd settings of the current project: document language, default level,
  review round limit, models per role, auto commit, test/lint/type commands. Triggers on "config",
  "settings", "/yrsdd:config", "настройки", "язык документов".
argument-hint: "[key=value ...]"
---

# yrsdd:config

Settings reference and defaults: [`../_shared/settings.md`](../_shared/settings.md). File:
`.claude/yrsdd.md` in the project.

## Protocol

1. If the file is missing, create it with the defaults (as `start` does) and say so.
2. **With arguments** (`artifact_language=en max_review_rounds=2`): validate each key and value
   against `settings.md`, patch only those keys, keep comments, order and unknown keys, show the
   changed lines.
3. **Without arguments:** print the current values in a short table, then ask with
   `AskUserQuestion` what to change (group: language and level / models / commands and commits).
   Apply only confirmed changes.
4. Remind: language applies to new documents only; existing ones keep theirs. Plugin
   instructions always stay English.
