# PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-02

## OVERVIEW
Home Assistant YAML automations that bridge Frigate NVR events to Telegram notifications. Russian-language bot messages with emoji classification for detected objects.

## STRUCTURE
```
.
├── Frigate Motion Alert to Telegram new 2.yaml   # Main motion → Telegram alert with inline keyboard
└── Frigate_telegram_run_command new.yaml          # Telegram callback → video / snapshot / live / alarm
```

## WHERE TO LOOK
| Task | Location |
|------|----------|
| Add new object type or emoji mapping | `Frigate Motion Alert to Telegram new 2.yaml` → `object_name` map |
| Add new camera alias | `Frigate Motion Alert to Telegram new 2.yaml` → `camera_name` map |
| Change Telegram recipients | Both files → `chat_id` lists |
| Add audio detection type | `Frigate Motion Alert to Telegram new 2.yaml` → `audio_object` map |
| Modify callback commands (video/snapshot/live/alarm) | `Frigate_telegram_run_command new.yaml` |
| **Change camera whitelist** | `Frigate Motion Alert to Telegram new 2.yaml` → top-level `conditions` template list |
| **Adjust preview GIF range** | `Frigate Motion Alert to Telegram new 2.yaml` → `preview.gif` URL `start_time` / `end_time` params |
| **Change inline keyboard buttons** | `Frigate Motion Alert to Telegram new 2.yaml` → `inline_keyboard` lists under NEW/END branches |
| **Add new callback handler** | `Frigate_telegram_run_command new.yaml` → add `telegram_callback` trigger + `choose` branch |

## CONVENTIONS
- **Language:** All user-facing strings are Russian. Object/place names use Cyrillic.
- **Templates:** Heavy Jinja2 in `value_template` and inline `message_template`. Home Assistant template syntax.
- **Parse modes:** `markdownv2` for Telegram captions (requires MarkdownV2 escaping).
- **URLs:** `frigate_base_url` (local) used for preview.gif animations and event snapshots.

## ANTI-PATTERNS (THIS PROJECT)
- Do NOT use standard Markdown in `parse_mode: markdownv2` — Telegram MarkdownV2 requires escaping: `\(` `\)` `\.` `\!` etc.
- Do NOT assume `sub_labels` index aligns 1:1 with `objects` — check bounds (`loop.index0 < sub_labels | length`) **and** filter empty strings (`and sub_labels[loop.index0]`).
- Do NOT remove `verify_ssl: false` without ensuring the Frigate instance has a valid certificate.
- Do NOT omit `| default([])` on `objects`, `sub_labels`, or `audio` — missing `data` fields will crash the template otherwise.

## NOTES
- Trigger source: MQTT topic `frigate/reviews` (Frigate reviews API).
- Automation modes: `parallel` with `max: 20` and `max_exceeded: silent`. High burst of events possible.
- `config_entry_id` is hardcoded — tied to a specific Telegram bot integration instance in HA.
- Camera whitelist filter lives in top-level `conditions`; edit the inline list to include/exclude cameras.
- `NEW` branch sends a live `preview.gif` (from `start_time` to `now()`); `END` branch sends the final `preview.gif` (from `start_time` to `end_time`).
- All Telegram send actions use `continue_on_error: true` so a failed send does not block the automation.
- `camera_name[camera] | default(camera)` is used so unknown cameras render gracefully.
- Inline keyboard uses YAML list syntax for multi-row layouts: each list item is a button row, buttons within a row are comma-separated (`Text:/command args`).
- NEW branch shows 🔴 Live + 🚨 Alarm buttons; END branch shows 📼 Video + 📸 Photo in first row and 🔴 Live in second row.
