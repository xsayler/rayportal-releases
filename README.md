# Rayportal

Десктопный VPN-клиент. TUN-туннель через [sing-box](https://sing-box.sagernet.org/),
поддерживает протокол VLESS (TCP / WS / gRPC / HTTP, TLS, Reality).

В этом репозитории публикуются **только готовые сборки**.

## Скачать

Последняя версия — на странице [Releases](../../releases/latest).

Файл `Rayportal_X.Y.Z_aarch64.dmg` — для Mac на Apple Silicon (M1 / M2 / M3 / M4).
Сборка для Intel пока не публикуется.

## Установка

1. Открыть скачанный `.dmg`
2. Перетащить **Rayportal.app** в **Applications**
3. Запустить из Launchpad
4. **Один раз** ввести пароль администратора macOS — приложение установит
   привилегированный helper, который позволяет поднимать TUN без пароля
   при каждом подключении

После этого подключение / отключение работает в один клик и без запросов пароля.

## Что внутри

- **TUN-туннель** — весь системный трафик идёт через VPN
- **Импорт подписок** — base64-формат, или прямой импорт `vless://…` ссылок
- **Раздельный туннель (split tunneling)** — пресеты для российских сайтов
  и сайтов госорганов (gosuslugi.ru, mos.ru, vk.com, yandex.ru, sberbank.ru
  и др.) и свой список доменов
- **Системный трей** — переключение профилей и Connect/Disconnect без
  открытия окна
- **Автозапуск** при входе в систему
- **Автообновление подписок** — раз в 30 мин / 1 час / 6 часов / сутки

## Безопасность

- Сборки **подписаны Developer ID Application** и **нотаризованы** Apple —
  Gatekeeper пропускает без предупреждений «приложение не идентифицировано»
- DNS-запросы перехватываются на TUN-уровне и идут через DoH
  (Cloudflare 1.1.1.1) внутри туннеля — нет утечки DNS
- Адрес VPN-сервера резолвится через системный DNS до поднятия туннеля,
  чтобы не было chicken-and-egg
- sing-box запускается с `auto_route` + `strict_route` — пакет не сможет
  случайно уйти мимо туннеля
- `sudoers`-правило выдаётся **только** для конкретного пути
  `/usr/local/libexec/rayportal/sing-box`, никаких других команд

## Системные требования

- macOS 12 (Monterey) или новее
- Apple Silicon (arm64)

## Удалить

```bash
sudo rm -rf /usr/local/libexec/rayportal /etc/sudoers.d/rayportal
rm -rf ~/Library/Application\ Support/app.rayportal.Rayportal
rm -f ~/Library/LaunchAgents/Rayportal.plist
```

И перетащить `Rayportal.app` в корзину.

## Лицензия

Внутри `.app` поставляется бинарник `sing-box` под лицензией
[GPL-3.0](https://github.com/SagerNet/sing-box/blob/main/LICENSE).
