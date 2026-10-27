[![Deploy with Dokploy](https://img.shields.io/badge/Deploy_with-Dokploy-black)](https://dealenx.github.io/Dokployfile/dealenx/vpn-xray-dokploy)

# XRay VLESS VPN

Два варианта VPN для обхода блокировок:

| Вариант | Маскировка | Для чего |
|---------|------------|----------|
| **Cloudflare** | Обычный HTTPS | Основной, через Dokploy |
| **Reality** | Госуслуги | Backup, жёсткие белые списки |

---

## Вариант 1: Cloudflare + Dokploy

VPN через Traefik + Cloudflare CDN.

### Деплой одной кнопкой (Dokploy)

Шаблон: [`Dokployfile.yml`](Dokployfile.yml) — генерирует импорт-payload для Dokploy из этого репозитория.

Открой https://dealenx.github.io/Dokployfile/dealenx/vpn-xray-dokploy — получишь Base64-payload, вставь в Dokploy (**Templates** → **Import**). Файлы шаблона: `Dokployfile.yml`, `docker-compose.dokploy.yml`, `template.toml`, `config/` в `.dokploy/config.template.json`.

### Деплой (готовый образ из GitHub)

`docker-compose.yml` использует образ из ghcr.io, собранный Actions:

```yaml
image: ghcr.io/dealenx/vpn-xray-dokploy:latest
```

Либо для вставки в compose:

```yaml
services:
  xray:
    image: ghcr.io/dealenx/vpn-xray-dokploy:latest
    restart: unless-stopped
    environment:
      - UUID=${UUID}
      - WS_PATH=${WS_PATH:-/ws}
      - DOMAIN=${DOMAIN}
    volumes:
      - ./config/config.template.json:/etc/xray/config.template.json:ro
    networks:
      - dokploy-network
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.xray.rule=Host(`${DOMAIN}`)"
      - "traefik.http.routers.xray.entrypoints=websecure"
      - "traefik.http.routers.xray.tls=true"
      - "traefik.http.routers.xray.tls.certresolver=letsencrypt"
      - "traefik.http.services.xray.loadbalancer.server.port=8080"

networks:
  dokploy-network:
    external: true
```

### Деплой из исходников

1. В Dokploy: **Create** → **Compose** → укажи этот репозиторий
2. В **Environment** добавь:
   ```
   DOMAIN=vpn.твой-домен.com
   UUID=твой-uuid-сюда
   WS_PATH=/ws
   ```
   UUID сгенерировать: `uuidgen`
3. **Deploy**

Несколько пользователей — UUID через запятую: `UUID=uuid1,uuid2,uuid3`

### Подключение

```
vless://UUID@ДОМЕН:443?encryption=none&security=tls&type=ws&path=/ws#VPN
```

---

## Вариант 2: Reality (одна команда)

Маскировка под госуслуги. Не требует домена и SSL.

### Установка

```bash
curl -sL "https://raw.githubusercontent.com/Biaslan-git/vpn-xray-dokploy/refs/heads/master/reality-backup/install.sh" | bash
```

Скрипт автоматически:
- Установит XRay
- Сгенерирует ключи
- Создаст systemd сервис
- Выдаст ссылку для клиента

### Управление пользователями

```bash
cd /opt/xray-reality
./add-user.sh Имя       # добавить
./list-users.sh         # список со ссылками
./remove-user.sh UUID   # удалить
```

### Управление сервисом

```bash
systemctl status xray-reality    # статус
systemctl restart xray-reality   # перезапуск
journalctl -u xray-reality -f    # логи
```

---

## Клиенты

- **Android**: v2rayNG, NekoBox
- **iOS**: Shadowrocket, Streisand
- **Windows/Mac/Linux**: Nekoray
