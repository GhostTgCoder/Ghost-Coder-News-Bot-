# Развёртывание и обновление

## Перед запуском

- Используйте отдельного системного пользователя.
- Храните `.env`, базу и Telegram-сессию вне публичного доступа.
- Ограничьте сетевые порты.
- Настройте резервное копирование и проверяйте восстановление.
- Сначала тестируйте на отдельном канале.

## systemd

Пример unit-файла, который нужно адаптировать к своему серверу:

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

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now ghost-coder-news-bot
sudo systemctl status ghost-coder-news-bot --no-pager
sudo journalctl -u ghost-coder-news-bot -n 100 --no-pager
```

Проверьте права пользователя сервиса и правильность путей.

## Резервное копирование

Сохраняйте базу данных, конфигурацию, ключ `AI_PROVIDER_ENCRYPTION_KEY`, а также MTProto-сессию, если она используется. Храните копии в защищённом месте отдельно от сервера. Не помещайте секреты в публичные архивы.

## Обновление

1. Прочитайте `CHANGELOG.md`.
2. Сделайте и проверьте резервную копию.
3. Остановите сервис, если обновление этого требует.
4. Установите новую версию и зависимости.
5. Запустите тесты.
6. Запустите сервис и проверьте логи, источники, ИИ и публикацию в тестовый канал.

## Откат и мониторинг

При ошибках сохраните логи, верните предыдущую версию и оценивайте необходимость восстановления базы отдельно. Откат базы может удалить новые данные.

Полезные команды:

```bash
systemctl status ghost-coder-news-bot --no-pager
journalctl -u ghost-coder-news-bot --since "1 hour ago" --no-pager
df -h
free -h
```

`0.9.0-rc1` — релиз-кандидат. До production выполните `RELEASE_CHECKLIST.md`.
