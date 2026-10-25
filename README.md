# qulogi-catalog

Публичный каталог релеев сообщества для [Qulogi](https://github.com/mel0k1/Qulogi) — инструмента обхода «белых списков».

Клиенты Qulogi забирают этот каталог с зеркал, проверяют подпись Ed25519 и выбирают самый быстрый живой транспорт:

- `https://raw.githubusercontent.com/mel0k1/qulogi-catalog/main/catalog.json`
- `https://cdn.jsdelivr.net/gh/mel0k1/qulogi-catalog@main/catalog.json`

The community catalog of Qulogi relays. Clients race the mirrors, verify the Ed25519 signature against the pinned key and pick the fastest live transport. Pull requests adding relays are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

## Формат

`catalog.json` — подписанный документ формата v1 (см. `docs/RELAY_DISCOVERY.md` в основном репозитории):

```json
{
  "version": 1,
  "updated": "2026-10-25T21:00:16Z",
  "relays": [
    {
      "id": "owl-eu-1",
      "region": "eu-central",
      "weight": 60,
      "transports": [
        { "type": "qulogi-tls", "addr": "r1.example.net:443", "sni": "vkvideo.ru" }
      ]
    }
  ],
  "key_id": "qulogi-dir-2026q4",
  "sig": "…"
}
```

Типы транспортов: `qulogi-tls` / `qulogi-tcp` (свой `qulogid`), `storage-webdav` (кадры туннеля через белое облако), `cdn-front` и `bot-longpoll` (планы). Операторские поля: `region` — география, `weight` 0–100 — подсказка выбора, `comment` — примечание для ревьюеров.

## Проверка подписи

Пиновый публичный ключ (зашит в клиенты):

```
3waP7rtqdJzSUWXV3HO1QXcs0kpP4SGYOWektwMsJsM=
```

Проверить каталог локально:

```bash
go build github.com/mel0k1/Qulogi/cmd/qulogicat
qulogicat -verify -in catalog.json -pub 3waP7rtqdJzSUWXV3HO1QXcs0kpP4SGYOWektwMsJsM=
```

CI этого репозитория делает то же самое на каждый PR: каталог без корректной подписи не попадёт в `main`.

## Зеркала и DNS TXT

Зеркалом может быть любой статический хостинг (GitHub Pages, jsDelivr, белый CDN). DNS TXT публикуется либо одной строкой-указателем (`https://…/catalog.json`), либо base64-кусками по 200 символов с префиксом порядка:

```bash
qulogicat -txt -url https://raw.githubusercontent.com/mel0k1/qulogi-catalog/main/catalog.json
qulogicat -key <seed> -in relays.json -txt   # куски каталога целиком
```

TXT-имя для своей зоны: `_qulogi.<ваш-домен>`; в клиенте оно добавляется как зеркало типа `dns-txt` или `doh`.

## Безопасность

- Зеркало не может подменить релеи: подпись ставится ключом сопровождающих, у клиента пиновый публичный ключ.
- Устаревший каталог — максимум «старый список релеев», не атака на клиента.
- Приватный seed хранится у сопровождающих; ротация — через новое `key_id` и пиннинг второго ключа в клиентах.

Данные каталога — [CC0-1.0](LICENSE).
