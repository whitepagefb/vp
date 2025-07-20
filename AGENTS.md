# AGENTS.md

## Общие цели

Этот файл описывает агентную логику для Codex / OpenAI DevDay Style Agentic Workflow, которая используется в проекте VPNBot (Outline + Hiddify).

## Задача

Цель: создать Telegram-бота, который:

1. Выдаёт Outline-ссылку в виде `ssconf://...`.
2. Одна и та же ссылка для всех стран, гео сменяется через nginx reverse proxy.
3. Дает 3 дня триала, затем ограничивает доступ.
4. Смена страны через кнопку в боте.

## Структура проекта

```
vpn-router/
├── nginx/
│   └── outline.conf.j2         # Jinja-шаблон конфига nginx
├── config/
│   └── current_country.txt     # Текущая страна (de, sg, etc)
├── switch_country.py              # Скрипт для смены IP в nginx
├── countries.json                 # Словарь: страна → IP
└── bot/
    └── bot.py                  # Telegram-бот
```

## Логика AGENTов

### Agent: TelegramBot

* Получает команды от пользователя ("/start", "Сменить гео")
* Показывает список доступных стран
* Вызывает `switch_country.py` с нужным кодом страны

### Agent: CountrySwitcher

* Загружает `countries.json`
* Записывает выбранную страну в `current_country.txt`
* Генерирует конфиг nginx из `outline.conf.j2`
* Перезапускает nginx

### Agent: AccessController (опционально)

* Проверяет срок действия доступа (3 дня trial)
* Если просрочен — отправляет сообщение и отключает доступ

## Поток действий

1. Пользователь запускает Telegram-бота
2. Получает ссылку ssconf://...
3. В панели — кнопка "Сменить страну"
4. При смене страны бот вызывает `switch_country.py` и nginx меняет reverse proxy
5. Пользователь продолжает использовать ту же ссылку — но уже с новым гео

## Что ещё важно:

* Outline Manager должен быть установлен на каждом VPS
* nginx (reverse proxy) — на отдельном сервере (или том же, что и бот)
* DNS (домен один): ssconf://yourdomain.com/... — меняется только IP

## Будущее апгрейды

* Добавить Hiddify
* UI-панель admin (Flask/FastAPI)
* Коллект статусов подключений
* Оплата (юкасса/сбер/юмоней)
* Резервные IP, failover
