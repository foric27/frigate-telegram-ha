# Frigate → Telegram Notifications

Home Assistant автоматизации и blueprints для отправки уведомлений о событиях Frigate NVR в Telegram.

## Возможности

- **Live-анимация**: отправка preview.gif при начале события
- **Обновление сообщений**: редактирование caption при обновлении и завершении события
- **Inline-кнопки**: быстрый доступ к видео, снимкам, live-view и тревогам
- **Обнаружение объектов**: поддержка person, car, dog, cat, package и многих других
- **Аудио-события**: определение лая, криков, сирен
- **Зоны**: отображение названий зон (двор, улица, гараж и т.д.)
- **Уровень тревожности**: различное оформление для alert vs detection
- **Несколько чатов**: одновременная отправка в несколько Telegram чатов

## Структура проекта

```
.
├── frigate_telegram_notifications.yaml        # Основная автоматизация (NEW/UPDATE/END)
├── frigate_telegram_callbacks.yaml             # Обработка inline-кнопок Telegram
├── frigate_telegram_notifications_blueprint.yaml  # Blueprint версия уведомлений
├── frigate_telegram_callbacks_blueprint.yaml      # Blueprint версия callback-команд
└── AGENTS.md                                    # Документация по проекту
```

## Требования

- Home Assistant с включённым MQTT брокером
- Интеграция [Frigate](https://docs.frigate.video/) (MQTT включён)
- Интеграция [Telegram Bot](https://www.home-assistant.io/integrations/telegram_bot/) в Home Assistant
- Созданные `input_text` helpers (см. ниже)

## Установка

### 1. Создание input_text helpers

Для каждой камеры из белого списка создайте `input_text` helper:
- Настройки → Устройства и службы → Помощники → Создать помощник → Text
- Название: `frigate_tg_msg_cam_1`, `frigate_tg_msg_cam_2`, и т.д.
- Максимальная длина: 255
- Icon: `mdi:message-text`

Эти helpers хранят пары `chat_id:message_id` для редактирования сообщений.

### 2. Получение Config Entry ID

1. Настройки → Устройства и службы → Telegram Bot
2. Откройте URL в браузере: `http://YOUR_HA:8123/config/integrations`
3. Найдите ID интеграции Telegram Bot (в URL или через инструменты разработчика браузера)
4. Либо используйте сервис `telegram_bot.send_message` в Automation с заполнением `config_entry_id` через UI

### 3. Установка через Blueprint (рекомендуется)

#### Уведомления (Frigate → Telegram)

1. Настройки → Автоматизации и сценарии → Blueprints → Импортировать blueprint
2. Вставьте URL или содержимое файла `frigate_telegram_notifications_blueprint.yaml`
3. Создайте автоматизацию из blueprint:
   - **Frigate URL**: `http://IP:5000`
   - **Telegram Config Entry ID**: полученный на шаге 2
   - **Chat IDs**: ID ваших Telegram чатов через запятую (например: `123456789,987654321`)
   - **Список камер**: камеры Frigate через запятую (например: `cam_1,cam_2`)
   - **Префикс input_text**: `frigate_tg_msg` (если helpers названы `frigate_tg_msg_cam_1`)

#### Callback-команды (Telegram → Frigate)

1. Импортируйте `frigate_telegram_callbacks_blueprint.yaml`
2. Заполните:
   - **Frigate URL**: тот же URL
   - **Telegram Config Entry ID**: тот же ID

### 4. Установка через YAML (альтернатива)

1. Скопируйте `frigate_telegram_notifications.yaml` в `configuration.yaml` или в папку `automations/`
2. Скопируйте `frigate_telegram_callbacks.yaml` аналогично
3. Отредактируйте переменные в начале каждой автоматизации:
   - `frigate_base_url`
   - `config_entry_id`
   - `chat_id` список
   - `camera_name` и другие маппинги при необходимости
4. Перезагрузите автоматизации: Developer Tools → YAML → Automations → Reload

### 5. Настройка Frigate

Убедитесь, что в `frigate.yml` включён MQTT:
```yaml
mqtt:
  enabled: true
  host: mqtt_broker_ip
```

И настроены камеры и зоны для детекции.

### 6. Тестирование

1. Вызовите событие в Frigate (пройдите перед камерой)
2. Проверьте появление сообщения в Telegram
3. Нажмите кнопки inline-клавиатуры для проверки callback-команд

## Настройка маппингов

В blueprints маппинги объектов, камер, зон и звуков заданы по умолчанию. Для их изменения:

1. **Blueprint**: после создания автоматизации перейдите в режим YAML редактора и измените словари `object_name`, `camera_name`, `zone_name`, `audio_object` в блоке `variables`
2. **YAML автоматизация**: отредактируйте переменные в начале файла

## Публикация Blueprint на GitHub

### Подготовка репозитория

1. Создайте репозиторий на GitHub (например: `myusername/frigate-telegram-ha`)
2. Скопируйте файлы в репозиторий:
   ```bash
   git clone https://github.com/myusername/frigate-telegram-ha.git
   cp frigate_telegram_notifications_blueprint.yaml frigate-telegram-ha/
   cp frigate_telegram_callbacks_blueprint.yaml frigate-telegram-ha/
   cp README.md frigate-telegram-ha/
   ```
3. Создайте тег `release` или `version` (опционально)

### Формат для Home Assistant Community

Для публикации в [Home Assistant Blueprint Exchange](https://community.home-assistant.io/c/blueprints-exchange/53):

1. Создайте тему в категории Blueprints Exchange
2. Заголовок: `[Blueprint] Frigate → Telegram Notifications`
3. Вставьте содержимое blueprint файла в блок code
4. Добавьте скриншоты примеров сообщений (опционально)
5. Укажите теги: `frigate`, `telegram`, `notifications`, `nvr`

### Импорт по URL

Home Assistant поддерживает импорт blueprints по прямой ссылке на raw YAML:

```
https://github.com/myusername/frigate-telegram-ha/raw/main/frigate_telegram_notifications_blueprint.yaml
```

Пользователи могут вставить этот URL в:
Настройки → Автоматизации и сценарии → Blueprints → Импортировать blueprint

## Лицензия

MIT License
