# Установка Ghost Coder News Bot

Инструкция для типового сервера Debian/Ubuntu. Версия `0.9.0-rc1` является релиз-кандидатом — сначала проверяйте работу на тестовом канале.

## Требования

- Linux-сервер и Python, совместимый с кодом проекта.
- Telegram-бот и токен от BotFather.
- Telegram ID администратора.
- Ключ ИИ-провайдера, если используется ИИ.
- Для чтения Telegram-источников — MTProto API ID, API Hash и сессия.
- HTTPS и контроль доступа, если веб-панель публикуется за пределами сервера.

## 1. Системные пакеты

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip git
```

Перейдите в каталог, куда распакован проект:

```bash
cd /opt/ghost-coder-news-bot
```

Это пример пути — замените его на фактический.

## 2. Виртуальное окружение и зависимости

```bash
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 3. Файл конфигурации

```bash
cp .env.example .env
nano .env
```

Заполните как минимум `BOT_TOKEN` и `ADMIN_IDS`. Проверьте `DATABASE_URL`, а также настройки ИИ и веб-панели, если они используются.

Сгенерируйте ключ Fernet для `AI_PROVIDER_ENCRYPTION_KEY`:

```bash
python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

Сохраните значение в `.env`. Не публикуйте этот файл, токен бота, API-ключи или файл Telegram-сессии.

Для MTProto получите `TELEGRAM_API_ID` и `TELEGRAM_API_HASH` на https://my.telegram.org. Пользовательская сессия даёт доступ к аккаунту и должна храниться в секрете.

## 4. Первый запуск

```bash
python main.py
```

Если запуск не удался, проверьте сообщение в терминале и руководство [Решение проблем](TROUBLESHOOTING.md).

## 5. Подключение канала

1. Создайте тестовый канал.
2. Добавьте бота в администраторы и разрешите публикации.
3. Добавьте канал в интерфейсе бота.
4. Подключите RSS/Atom-ленту или доступный Telegram-источник.
5. Настройте ИИ для канала или выберите **«ОРИГИНАЛ БЕЗ ИИ»**.
6. Проверьте очередь, теги, футер и модерацию.
7. Опубликуйте тестовый материал вручную.
8. Только после проверки включайте автоматическое расписание.

## 6. Запуск через systemd

Пример `/etc/systemd/system/ghost-coder-news-bot.service` — адаптируйте пути и пользователя:

```ini
[Unit]
Description=Ghost Coder News Bot
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=ghostbot
WorkingDirectory=/opt/ghost-coder-news-bot
EnvironmentFile=/opt/ghost-coder-news-bot/.env
ExecStart=/opt/ghost-coder-news-bot/venv/bin/python /opt/ghost-coder-news-bot/main.py
Restart=on-failure
RestartSec=5
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
```

Затем:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ghost-coder-news-bot
sudo systemctl status ghost-coder-news-bot --no-pager
sudo journalctl -u ghost-coder-news-bot -n 100 --no-pager
```

Проверьте права пользователя `ghostbot` на проект, базу и файлы сессии.

## 7. Веб-панель и безопасность

По умолчанию безопаснее привязать панель к `127.0.0.1`, а внешний доступ предоставлять через HTTPS reverse proxy. Не открывайте административный порт всему интернету без оценки рисков.

Перед рабочим запуском настройте резервное копирование, проверьте права файлов, тесты, источники, режим оригинала и восстановление после перезапуска.

**[🛒 Купить Ghost Coder News Bot на Kwork](https://kwork.ru/script-programming/55120212/telegram-bot-kotoriy-sam-sobiraet-i-publikuet-novosti)** — перед покупкой уточните, входит ли установка и настройка в стоимость.
