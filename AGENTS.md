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
| **Change severity emoji mapping** | `Frigate Motion Alert to Telegram new 2.yaml` → `severity_map` |
| **Change zone display names** | `Frigate Motion Alert to Telegram new 2.yaml` → `zone_name` map |
| **Add review preview button** | `Frigate_telegram_run_command new.yaml` → add `/send_review_preview` trigger + branch |

## CONVENTIONS
- **Language:** All user-facing strings are Russian. Object/place names use Cyrillic.
- **Templates:** Heavy Jinja2 in `value_template` and inline `message_template`. Home Assistant template syntax.
- **Parse modes:** `markdownv2` for Telegram captions (requires MarkdownV2 escaping).
- **URLs:** `frigate_base_url` (local) used for preview.gif animations and event snapshots.

## ANTI-PATTERNS (THIS PROJECT)
- Do NOT use standard Markdown in `parse_mode: markdownv2` — Telegram MarkdownV2 requires escaping: `\(` `\)` `\.` `\!` etc.
- Do NOT assume `sub_labels` index aligns 1:1 with `objects` — check bounds (`loop.index0 < sub_labels | length`) **and** filter empty strings (`and sub_labels[loop.index0]`).
- Do NOT remove `verify_ssl: false` without ensuring the Frigate instance has a valid certificate.
- Do NOT omit `| default([])` on `objects`, `sub_labels`, `audio`, or `zones` — missing `data` fields will crash the template otherwise.
- Do NOT use `telegram_bot.edit_message` on media messages — use `edit_caption` for photos/videos/animations, `edit_replymarkup` for keyboards only.
- Do NOT try to edit a message with `message_id: "last"` from a different chat — store exact `chat_id:message_id` pairs per camera.

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
- **Message updates** (`edit_caption` + `edit_replymarkup`): `NEW` captures `response_variable` from `send_animation`, stores `chat_id:message_id` pairs in `input_text.frigate_tg_msg_{camera}`. `UPDATE` edits the same message caption with fresh objects/zones/duration. `END` edits caption to final duration and replaces keyboard with 📼/📸/🔴 buttons, then clears the helper.
- **Required HA helpers**: Create one `input_text` per camera named `input_text.frigate_tg_msg_cam_1` through `cam_6`. Max length should accommodate ~50 chars (e.g. `442672175:12345,2058806977:67890`).
- **Callback UX**: All callback branches end with `telegram_bot.answer_callback_query` to remove the "loading" spinner from the button press.
- `config_entry_id` is stored as a variable in the callback automation to avoid repetition across branches.
