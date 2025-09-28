# О репозитории

- GUI поддерживает только английский и русский (en/ru).
- Адрес панели/API должен иметь префикс пути: `http://<domain>:<port>/api`.

[![build](https://github.com/lejianwen/rustdesk-server/actions/workflows/build.yaml/badge.svg)](https://github.com/lejianwen/rustdesk-server/actions/workflows/build.yaml)

- Исправлена проблема таймаута соединения при входе клиента под `API`-аккаунтом
- В s6-образ добавлена поддержка `API` (исходники: https://github.com/lejianwen/rustdesk-api)
- Требовать ли вход для соединения: `MUST_LOGIN` по умолчанию `N`, установите `Y` для обязательной авторизации
- `RUSTDESK_API_JWT_KEY` — при наличии выполняется валидация токена через `JWT`
- Поддерживается client websocket (client >= 1.4.1)

## Docker (s6-образ)

```yaml
networks:
  rustdesk-net:
    external: false
services:
  rustdesk:
    ports:
      - 21114:21114
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21117:21117
      - 21118:21118
      - 21119:21119
    image: lejianwen/rustdesk-server-s6:latest
    environment:
      - RELAY=<relay_server[:port]>
      - ENCRYPTED_ONLY=1
      - MUST_LOGIN=N
      - TZ=Asia/Shanghai
      - RUSTDESK_API_RUSTDESK_ID_SERVER=<id_server[:21116]>
      - RUSTDESK_API_RUSTDESK_RELAY_SERVER=<relay_server[:21117]>
      - RUSTDESK_API_RUSTDESK_API_SERVER=http://<api_server[:21114]>/api
      - RUSTDESK_API_KEY_FILE=/data/id_ed25519.pub
      - RUSTDESK_API_JWT_KEY=xxxxxx # jwt key
    volumes:
      - /data/rustdesk/server:/data
      - /data/rustdesk/api:/app/data # база API
    networks:
      - rustdesk-net
    restart: unless-stopped
```

## Сборка вручную

```bash
cargo build --release
```

Собираются 3 исполняемых файла в `target/release`:

- `hbbs` — ID/Rendezvous сервер
- `hbbr` — Relay сервер
- `rustdesk-utils` — CLI утилиты

Актуальные бинарники: [Releases](https://github.com/rustdesk/rustdesk-server/releases)

## Переменные окружения (ENV)

| Переменная | Бинарь | Описание |
| --- | --- | --- |
| RELAY | hbbs | IP/DNS машины с hbbr (через запятую для нескольких) |
| ENCRYPTED_ONLY | hbbs | Если `"1"`, незашифрованные подключения запрещены |
| PORT | hbbs/hbbr | Порт прослушивания (hbbs: 21116, hbbr: 21117) |
| RUST_LOG | all | Уровень логирования (error|warn|info|debug|trace) |

Дополнительно для API контейнера:

| Переменная | Описание |
| --- | --- |
| RUSTDESK_API_RUSTDESK_API_SERVER | Базовый URL панели/API, используйте формат `http://<domain>:<port>/api` |
| RUSTDESK_API_RUSTDESK_ID_SERVER | Адрес ID/Rendezvous сервера |
| RUSTDESK_API_RUSTDESK_RELAY_SERVER | Адрес Relay сервера |
| RUSTDESK_API_JWT_KEY | Ключ для проверки JWT |

## Полезные ссылки

- API репозиторий: https://github.com/lejianwen/rustdesk-api
- Документация (EN): https://rustdesk.com/docs/en/self-host/
- Документация (ZH): https://rustdesk.com/docs/zh-cn/self-host/
