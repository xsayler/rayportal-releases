# Rayportal

Десктопный VPN-клиент. TUN-туннель через [sing-box](https://sing-box.sagernet.org/),
поддерживает протокол VLESS (TCP / WS / gRPC / HTTP, TLS, Reality).

Работает на **macOS (Apple Silicon)**, **Linux (x86_64)** и **Windows 11 (x64 / ARM64)**.

В этом репозитории публикуются **только готовые сборки**.

## Скачать

Последняя версия — на странице [Releases](../../releases/latest).

| Платформа | Файл | Подпись |
|---|---|---|
| **macOS** (Apple Silicon: M1–M4) | `Rayportal_X.Y.Z_aarch64.dmg` | подписан + нотаризован Apple |
| **Windows x64** | `Rayportal_X.Y.Z_x64-setup.exe` | без подписи (SmartScreen предупредит) |
| **Windows ARM64** | `Rayportal_X.Y.Z_arm64-setup.exe` | без подписи (SmartScreen предупредит) |
| **Linux** (Debian/Ubuntu) | `Rayportal_X.Y.Z_amd64.deb` | без подписи |
| **Linux** (Fedora/RHEL) | `Rayportal-X.Y.Z-1.x86_64.rpm` | без подписи |
| **Linux** (универсально) | `Rayportal_X.Y.Z_amd64.AppImage` | без подписи |
| **Linux** (portable) | `Rayportal_linux-x86_64-glibc.tar.gz` | без подписи |

Сборка macOS для Intel не публикуется. Рядом с каждым файлом лежит
`SHA256SUMS-<платформа>.txt` для проверки контрольной суммы.

## Установка

### macOS

1. Открыть скачанный `.dmg`
2. Перетащить **Rayportal.app** в **Applications**
3. Запустить из Launchpad
4. **Один раз** ввести пароль администратора macOS — приложение установит
   привилегированный helper, который позволяет поднимать TUN без пароля
   при каждом подключении

### Linux

1. Установить пакет:
   - Debian/Ubuntu: `sudo apt install ./Rayportal_X.Y.Z_amd64.deb`
   - Fedora/RHEL: `sudo dnf install ./Rayportal-X.Y.Z-1.x86_64.rpm`
   - Либо сделать `.AppImage` исполняемым (`chmod +x`) и запустить, или
     распаковать `.tar.gz` и запустить `bin/rayportal`
2. Запустить приложение
3. При первом подключении **один раз** появится системный диалог PolicyKit
   (**pkexec**) — ввести пароль. Приложение установит привилегированный
   helper (sing-box + runner в `/usr/local/libexec/rayportal/`), дальше
   подключение работает без запросов пароля

Требуется glibc **2.35+** (Ubuntu 22.04+ / Fedora 36+ и новее).

### Windows

1. Запустить `Rayportal_X.Y.Z_x64-setup.exe` (или `arm64` для ARM-устройств)
2. Сборки **без подписи**, поэтому SmartScreen покажет «Windows protected
   your PC» → **More info** → **Run anyway**
3. Пройти установщик (ставит в `Program Files`)
4. При первом подключении **один раз** появится запрос UAC — приложение
   зарегистрирует системную службу `RayportalService` (LocalSystem), которая
   владеет туннелем. Дальше подключение работает без запросов

После установки на любой платформе подключение / отключение — в один клик
и без повторных запросов пароля.

## Что внутри

- **TUN-туннель** — весь системный трафик идёт через VPN
- **Импорт подписок** — base64-формат, или прямой импорт `vless://…` ссылок
- **Раздельный туннель (split tunneling):**
  - пресеты для российских сайтов и сайтов госорганов (gosuslugi.ru, mos.ru,
    vk.com, yandex.ru, sberbank.ru и др.) и свой список доменов
  - **BitTorrent в обход VPN** — трафик известных торрент-клиентов и
    распознанный по sniff протокол BitTorrent (μTP/DHT) идут напрямую
  - **свой список IP-адресов и подсетей** — IPv4/IPv6, одиночные адреса и
    CIDR-диапазоны идут напрямую, минуя туннель
- **Системный трей** — переключение профилей и Connect/Disconnect без
  открытия окна
- **Автозапуск** при входе в систему
- **Автообновление подписок** — раз в 30 мин / 1 час / 6 часов / сутки

## Безопасность

- **DNS без утечек** — DNS-запросы перехватываются на TUN-уровне и идут через
  DoH (Cloudflare 1.1.1.1) внутри туннеля
- Адрес VPN-сервера резолвится через системный DNS до поднятия туннеля,
  чтобы не было chicken-and-egg
- sing-box запускается с `auto_route` + `strict_route` — пакет не сможет
  случайно уйти мимо туннеля
- **Привилегии выдаются точечно:**
  - **macOS/Linux** — sudoers-правило **только** для конкретного пути
    `/usr/local/libexec/rayportal/run-sing-box`, никаких других команд
  - **Windows** — служба LocalSystem общается с приложением по named pipe
    `\\.\pipe\rayportal-v1` с явным DACL (только SYSTEM, Administrators и SID
    установившего пользователя), remote-клиенты запрещены; обрыв pipe
    останавливает туннель, так что падение GUI не оставляет orphan-процессов
- Установщик привилегированной части сверяет **SHA-256** bundled sing-box
  (и Wintun на Windows) перед копированием
- **macOS** — сборки подписаны Developer ID Application и нотаризованы,
  Gatekeeper пропускает без предупреждений. **Windows/Linux** сборки без
  подписи (на Windows будет SmartScreen-предупреждение при первом запуске)

## Системные требования

- **macOS** 12 (Monterey) или новее, Apple Silicon (arm64)
- **Linux** x86_64, glibc 2.35+ (Ubuntu 22.04+ / Fedora 36+)
- **Windows** 11, x64 или ARM64

## Удалить

**macOS:**

```bash
sudo rm -rf /usr/local/libexec/rayportal /etc/sudoers.d/rayportal
rm -rf ~/Library/Application\ Support/app.rayportal.Rayportal
rm -f ~/Library/LaunchAgents/Rayportal.plist
```

И перетащить `Rayportal.app` в корзину.

**Linux:**

```bash
# удалить пакет: sudo apt remove rayportal  /  sudo dnf remove rayportal
# (или удалить .AppImage / распакованный каталог)
sudo rm -f /etc/sudoers.d/rayportal
sudo rm -rf /usr/local/libexec/rayportal
rm -rf ~/.local/share/Rayportal
```

**Windows:**

Удалить через **Параметры → Приложения** (штатный деинсталлятор снимает и
службу `RayportalService`). Затем при желании удалить данные:
`%APPDATA%\rayportal\Rayportal` и `%ProgramData%\Rayportal`.

## Лицензия

Внутри сборки поставляется бинарник `sing-box` под лицензией
[GPL-3.0](https://github.com/SagerNet/sing-box/blob/main/LICENSE).
На Windows дополнительно поставляется подписанный
[Wintun](https://www.wintun.net/) DLL.
