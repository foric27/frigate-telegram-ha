# PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-02

## OVERVIEW
Home Assistant YAML automations that bridge Frigate NVR events to Telegram notifications. Russian-language bot messages with emoji classification for detected objects.

## STRUCTURE
```
.
├── frigate_telegram.yaml   # Blueprint: уведомления (MQTT) + callback-команды (Telegram events) в одном файле
├── README.md               # Installation guide and publishing instructions
└── AGENTS.md               # This file — project knowledge base
```

## WHERE TO LOOK
| Task | Location |
|------|----------|
| Add new object type or emoji mapping | `frigate_telegram.yaml` → `object_name` map (variables block) |
| Add new camera alias | `frigate_telegram.yaml` → `camera_name` map |
| Change Telegram recipients | `frigate_telegram.yaml` → `chat_ids` input (blueprint) |
| Add audio detection type | `frigate_telegram.yaml` → `audio_object` map |
| Modify callback commands (video/snapshot/live/alarm) | `frigate_telegram.yaml` → callback branch (`send_video`, `send_snapshot`, etc.) |
| **Change camera whitelist** | `frigate_telegram.yaml` → top-level `conditions` template (blueprint: `camera_whitelist` input) |
| **Adjust preview GIF range** | `frigate_telegram.yaml` → `preview.gif` URL `start_time` / `end_time` params in NEW branch |
| **Change inline keyboard buttons** | `frigate_telegram.yaml` → `inline_keyboard` lists under NEW/END branches |
| **Add new callback handler** | `frigate_telegram.yaml` → add `telegram_callback` trigger + nested `choose` branch |
| **Change severity emoji mapping** | `frigate_telegram.yaml` → `severity_map` |
| **Change zone display names** | `frigate_telegram.yaml` → `zone_name` map |
| **Add review preview button** | `frigate_telegram.yaml` → add `/send_review_preview` trigger + callback branch |
| **Install via blueprint** | `frigate_telegram.yaml` (single blueprint) |
| **Installation instructions** | `README.md` |

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
- **Single blueprint**: `frigate_telegram.yaml` contains both MQTT triggers (`frigate/reviews`) and Telegram event triggers (`telegram_callback`). Uses `trigger.id` to branch between notification logic and callback logic.
- Trigger source: MQTT topic `frigate/reviews` (Frigate reviews API).
- Automation mode: `parallel` with `max: 20` and `max_exceeded: silent`. High burst of events possible.
- `config_entry_id` comes from blueprint input (`!input telegram_config_entry_id`).
- Camera whitelist filter lives in top-level `conditions` using `trigger.id` check for safety.
- `NEW` branch sends a live `preview.gif` (from `start_time` to `now()`); `END` branch sends the final `preview.gif` (from `start_time` to `end_time`).
- All Telegram send actions use `continue_on_error: true` so a failed send does not block the automation.
- `camera_name[camera] | default(camera)` is used so unknown cameras render gracefully.
- Inline keyboard uses YAML list syntax for multi-row layouts: each list item is a button row, buttons within a row are comma-separated (`Text:/command args`).
- NEW branch shows 🔴 Live + 🚨 Alarm buttons; END branch shows 📼 Video + 📸 Photo in first row and 🔴 Live in second row.
- **Message updates** (`edit_caption` + `edit_replymarkup`): `NEW` captures `response_variable` from `send_animation`, stores `chat_id:message_id` pairs in `input_text.frigate_tg_msg_{camera}`. `UPDATE` edits the same message caption with fresh objects/zones/duration. `END` edits caption to final duration and replaces keyboard with 📼/📸/🔴 buttons, then clears the helper.
- **Required HA helpers**: Create one `input_text` per camera named `input_text.frigate_tg_msg_cam_1` through `cam_6`. Max length should accommodate ~50 chars (e.g. `442672175:12345,2058806977:67890`).
- **Callback UX**: All callback branches end with `telegram_bot.answer_callback_query` to remove the "loading" spinner from the button press.
- `config_entry_id` is stored as a top-level variable to avoid repetition across branches.
