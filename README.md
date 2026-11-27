# air-scope.ru

Посадочная страница платформы AirScope (модули AirBIM, AirInfra, AirHeritage). Разработка ИКИТ СФУ.

Сайт статический: всё, что публикуется, лежит в папке `site/`.

## Как работает выкладка

Сайт отдается с сервера «Практики полета» в Рег.облаке (80.78.248.203, Caddy). Схема «сервер тянет»:

1. `git push` в ветку `main`.
2. Не позже чем через 3 минуты таймер `airscope-deploy.timer` на сервере забирает новый коммит и выкладывает `site/` в `/srv/www/air-scope.ru` (`rsync --delete`).
3. Caddy отдает папку по адресу https://air-scope.ru; `www.air-scope.ru` переадресуется на основной адрес. Сертификат Let's Encrypt Caddy выпускает и продлевает сам.

Репозиторий открытый, поэтому серверу не нужен ключ доступа к GitHub.

На сервере:

| Путь | Что |
|---|---|
| `/srv/src/airscope-site` | клон этого репозитория |
| `/srv/infra/deploy-airscope.sh` | скрипт выкладки: fetch → reset → rsync `site/` |
| `/etc/systemd/system/airscope-deploy.{service,timer}` | запуск скрипта от пользователя `deploy` каждые 3 минуты |
| `/srv/src/.deployed-rev-airscope` | последний выложенный коммит |
| `/srv/www/air-scope.ru` | выложенный сайт |
| `/srv/infra/Caddyfile` | блоки `air-scope.ru` и `www.air-scope.ru` в конце файла |

Команды на сервере:

```bash
journalctl -u airscope-deploy -f                  # журнал выкладки
sudo systemctl start airscope-deploy              # выложить сейчас, не дожидаясь таймера
sudo -u deploy FORCE=1 /srv/infra/deploy-airscope.sh   # выложить заново без нового коммита
```

## DNS

Домен `air-scope.ru` зарегистрирован в REG.RU. Записи A для `@` и `www` указывают на 80.78.248.203.

## Локальный просмотр

Открыть `site/index.html` в браузере.
