# Traefik - роутинг просто


![image](/images/traefik_imgs/logo.png){ align=right }

Сегодня я бы хотел рассказать о такой клёвой штуке, как [Traefik](https://traefik.io/traefik/). Не так давно я перевёл все сервисы своего сервера на traefik и это буквально сделало жизнь проще. Сегодня я расскажу вам зачем он нужен, как его настроить, и покажу на примере, как именно он помогает. По факту, это nginx на стероидах, в плане конфигурации, потому что в traefik за тебя сделано гораздо больше
# Что такое traefik

Traefik - это система, которая позволяет настроить маппинг между доменными именами и конкретными приложениями. Допустим, у вас есть контейнер frontend-приложения, которое вы хотели бы разместить на домене `myapp.com`, и также у вас есть контейнер backend-приложения, которое вы бы хотели разместить на домене `api.mayapp.com`. [Traefik](https://traefik.io/traefik/) поможет вам это сделать без лишних файлов конфигурации.

## Сравнение с другими инструментами

Если бы вы решили сделать это через [Nginx](https://www.nginx.com/), то вы бы создали новые файлы конфигураций под каждый домен, в папочке с конфигурациями для всех доменов, положили куда-нибудь сертификат от домена `*myapp.com` и подключали бы его вручную в каждом файле доменов. А если бы вам надо было увеличить количество контейнеров отдельного приложения, вы бы указывали сервера в директиве `upstream` и перечисляли там адреса инстансов приложения вручную.

Любители [Apache HTTP Server](https://httpd.apache.org/) на этом моменте обычно берут дробовик и разносят себе голову хорошим зарядом дроби.

А господа, которые используют [Traefik](https://traefik.io/traefik/), просто дописывают лейблы контейнеру в `docker-compose.yml` и идут дальше дегустировать вино.

!!! note ""
	Если же ваше приложение построено на вебсокетах, то тут уже фанаты Nginx тянутся за дробовиком. Ну а для ценителей traefik ничего не меняется, ведь в нём встроена поддержка HTTP/2.0.

![image](/images/traefik_imgs/nginx_comparasion.png){ align=center }

## Архитектура
В официальной документации зарисована следующая схема работы traefik:
![image](/images/traefik_imgs/traefik-architecture.png){ align=center }

Как можно видеть по данному изображению, есть 5 основных составляющих traefik, а именно:

* entrypoints
* routers
* rules (часть routers)
* middlewares (чать routers)
* services

### Entrypoint
Являются основными слушателями входящих соединений. Через них проходит весь трафик. Если попробовать объяснить в двух словах: "Порт, на который должно прийти входящее соединение", - вполне себе неплохое объяснение. В данном туториале я создам 2 `entrypoint`, которые будут слушать на http и https.

### Routers, Rules и Middlewares 
Роутер - связующее звено между `entrypoint` и сервисом, куда нужно направить трафик. Роутер хранит в себе информацию, куда направить входящий трафик, и 
правила, по которым можно определить, пускать ли трафик дальше.

Rules - это и есть те самые правила, которые определяют, через какой роутер пустить трафик.
Допустим, у вас в конфиге есть следующие правила:
```
Host(`myapp.com`) && (PathPrefix(`/api`) || PathPrefix(`/memes`))
```

Это можно читать как "Этот роутер пустит в моё приложение, если запрос пришёл на хост `myapp.com` и путь запроса начинается с `/api` или `/memes`"

Довольно просто, не так ли?

Ну а middlewares - это некоторая логика, что нужно выполнить с запросом до того, как он попадёт в ваше приложение. В этой статье я не буду рассказывать про существующие `middlewares`, тут потребуется самостоятельное ознакомление с [документацией](https://doc.traefik.io/traefik/middlewares/overview/).

# Конфигурация

!!! note ""
	В данной статье я буду использовать Docker для конфигурации traefik. 
	Вы, конечно же, можете использовать локально установленную версию, всё будет работать, как часы.

Для нашего основного traefik на сервере мы создадим небольшой docker-compose.yml, файл конфига и папочку с сертификатами.
Структура будет следующая:
```
.
├── certs
│   ├── local.key
│   └── local.pem
├── config.toml
└── docker-compose.yml
```
Теперь разберем каждый файл по порядку.
Вот основной docker-compose нашего traefik.

```yaml
# docker-compose.yml
---
version: '3.7'

services:
  traefik-proxy:
    # The official v2.0 Traefik docker image
    image: traefik:v2.4.8
    container_name: traefik_main
    command: 
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --providers.docker.network=traefik-shared
      - --providers.file.directory=/etc/traefik/dynamic
      - --providers.file.watch=true
      - --entrypoints.http.address=:80
      - --entrypoints.https.address=:443
    ports:
      # The HTTP port
      - "80:80"
      # The HTTPS port
      - "443:443"
    restart: unless-stopped
    networks:
      - traefik-shared
    environment:
      MAIN_HOST: 192.168.1.89
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
      - ./config.toml:/etc/traefik/dynamic/traefik.toml
      - ./certs:/etc/certs/

networks:
  traefik-shared:
    name: traefik-shared

```

Вот какие параметры я передаю в `traefik-cli`.

| Параметр                                        | Что делает                                                                       |
|-------------------------------------------------|----------------------------------------------------------------------------------|
| `providers.docker=true`                         | Включает прослушивание докера на новые события и следит за лейблами контейнеров. |
| `providers.docker.exposedbydefault=false`       | Отключает автоматическое создание роутеров ко всем контейнерам на сервере.       |
| `providers.docker.network=traefik-shared`       | Сеть докера, по которой будет выполнятся подключение к контейнерам.               |
| `providers.file.directory=/etc/traefik/dynamic` | Папка с конфигурационным файлом.                                                 |
| `providers.file.watch=true`                     | Включает отслеживание изменений файла конфигурации.                              |
| `entrypoints.http.address=:80`                  | Создаёт entrypoint с названием http и слушает 80 порт.                           |
| `entrypoints.https.address=:443`                | Создаёт entrypoint с названием https и слушает 443 порт.                         |

Конечно, вы всегда можете глянуть `traefik --help` и подобрать себе желаемые параметры.

Также из docker-compose файла видно, что я создал докер сеть `traefik-shared`, которую в дальнейшем буду использовать на всех контейнерах, которым требуется свой домен.

Далее следует папка с сертификатами. Для реальных доменов я использую cloudflare и скачиваю сертификаты и ключи с панели администратора. Также не забываю выставлять мод шифрования на strict.

![SSL/TLS encryption mode](/images/traefik_imgs/couldflare_certs.png)

Для генерации локальных сертификатов я использую тулу [mkcert](https://github.com/FiloSottile/mkcert).

Для любого локального домена я делаю что-то типа:
```bash
mkcert "*.local"
mv _wildcard.local-key.pem local.key
mv _wildcard.local.pem local.pem
```
И помещаю это в папочку `certs` рядом с `docker-compose.yml`.

После того как я создал все нужные сертефикаты для всех доменов, их надо указать в файле `config.toml` в папочке traefik.

Вот пример config.toml.
```toml
[tls.options]
  [tls.options.default]
  	# Эта опция вообще для strict cloudflare tls encryption,
	# Но я включаю её и на локальных доменах.
    sniStrict = true

[[tls.certificates]]
  # Я тут указываю /etc/certs, потому что в docker-compose
  # у нас volume на эту папку.
  certFile = "/etc/certs/local.pem" 
  keyFile = "/etc/certs/local.key"
```

!!! note ""
	Для добавления новых сертификатов в данный файл достаточно добавить:
```toml
[[tls.certificates]]
	certFile = "/etc/certs/<certFile>" 
	keyFile = "/etc/certs/<keyFile>"
```

После этого вы можете запускать traefik и наслаждаться доменами для ваших контейнеров.

# Запуск приложений
Теперь сконфигурируем приложение таким образом, чтобы к нему можно было обращаться через доменное имя.

Для примера возьмем мелкое приложение на nodejs со следующей структурой проекта:
```
.
├── docker-compose.yml
├── Dockerfile
├── .dockerignore
├── index.js
├── package.json
└── yarn.lock
```

```js
// index.js
const express = require('express')

let req_count = 0;
const hostname = process.env.HOSTNAME;
const PORT = process.env.PORT || 3000;

app = express();

app.get("/", (req, res) => {
  console.log(`GET / ${hostname}`)
  res.send({request_num: req_count++, host: hostname})
})

app.listen(PORT, () => {
  console.log(`Server is listening on port ${PORT}`);
});
```

```json
// package.json
{  
 "name": "express-test",  
 "version": "1.0.0",  
 "main": "index.js",  
 "author": "s3rius",  
 "license": "MIT",  
 "scripts": {  
 "runserver": "node index.js"  
 },  
 "dependencies": {  
 "express": "^4.17.1"  
 }  
}
```

```dockerfile
# Dockerfile
FROM node:16-alpine3.11

WORKDIR /app
COPY package.json yarn.lock /app/

RUN yarn install

COPY . /app/

ENTRYPOINT ["yarn", "run"]
```

```yaml
# docker-compose.yml
---
version: '3.7'


services:
  server:
    build: .
    labels:
      - traefik.enable=true
      - traefik.http.routers.test_node.rule=Host(`test_node.local`)
      - traefik.http.routers.test_node.entrypoints=http
      - traefik.http.routers.test_node.service=node_test
      - traefik.http.services.node_test.loadbalancer.server.port=3000
    command: runserver
    networks:
      - traefik-shared


networks:
  traefik-shared:
    name: traefik-shared
    external: true
```

!!! note ""
	Файл yarn.lock генерируется командой `yarn install`. Если у вас не установлен `yarn`, то ничего страшного. Просто это будет занимать чуть больше времени на сборку образа.
	
Как вы видите, приложение слушает на порт `3000` и отвечает свим hostname и количеством обработанных запросов. А в docker-compose.yml, в отличие от обычного проекта, появились labels.

| Лейбл                                                         | Что делает                                                                            |
|---------------------------------------------------------------|---------------------------------------------------------------------------------------|
| ``traefik.enable=true``                                           | Включить поддержку роутинга через traefik                                             |
| ``traefik.http.routers.<router>.rule=Host(`test_node.local`)``   | Поставить правило роутинга, если Host запроса равен test_node.local                   |
| ``traefik.http.routers.<router>.entrypoints=http``               | Слушать на entrypoint http (80 порт, это было объявлено в параметрах запуска traefik) |
| ``traefik.http.routers.<router>.service=<service>``              | Сервис, связанный с роутером test_node                                                 |
| ``traefik.http.services.<service>.loadbalancer.server.port=3000`` | Порт, куда направлять запросы в сервис node_test                                      |


!!! error "Обратите внимание"
	Роутеры, сервисы и хосты явно создавать нигде не нужно.
	Они сами создаются, когда вы их указываете.
	
	Название сервиса и роутера могут совпадать.
	
	Обратите внимание на косые кавычки при указании хоста! Это обязательно.
	
	В объявлении labels могут быть использованы переменные среды. Например: 
	``traefik.http.routers.test_node.rule=Host(`${APP_HOST}`)``

Также можно видеть, что я подключил контейнер к сети, которую мы указывали в контейнере traefik. Здесь она помечена как external.

Теперь мы можем спокойно запустить наш сервис. Для этого воспользуемся следующей командой:
```bash
docker-compose up --build --scale server=3
```

В данной команде мы указали, что хотим поднять 3 инстанса нашего сервиса. Остальным пусть занимается traefik.

Теперь попробуем некоторое количество раз выполнить запрос на наш сервис.

```console
$ curl -H "Host: test_node.local" "http://localhost"
{"request_num":0,"host":"7417ac8fda92"}
```

Результат должен быть примерно таким:

[![GIF](/images/traefik_imgs/curls.gif)](/images/traefik_imgs/curls.gif)

Как вы видите, traefik балансирует между контейнерами за нас. И я считаю, что это - победа.

## Подключение TLS и сертификатов
Тут всё не намного сложнее. Давайте немного поменяем лейблы нашего контейнера.

```yaml
services:
  server:
    labels:
      - traefik.enable=true
      - traefik.http.routers.test_node.rule=Host(`test_node.local`)
      - traefik.http.routers.test_node.entrypoints=https
      - traefik.http.routers.test_node.tls=true
      - traefik.http.routers.test_node.service=node_test
      - traefik.http.services.node_test.loadbalancer.server.port=3000
```

Как вы видите, я поменял entrypoints на `https`. Как вы можете помнить, это entrypoint, который слушает на 443 порт. И также я включил поддержку tls лейблом `traefik.http.routers.<router>.tls=true`

На данном этапе вам потребуется добавить свой хост в `/etc/hosts`, если вы используете нормальную систему. Но если вы всё же на windows, то вам потребуется добавить правило в `C:\Windows\System32\drivers\etc\hosts`.

И добавляем в конец файла запись:
```
127.0.0.1	test_node.local
```

И также вам потребуется cертификат на этот домен. 
Для этого:

1. Создадим сертификат через `mkcert`, как упоминалось ранее;
2. Поместим ключ и сертификат в папку certs;
3. Добавим ключ и сертификат для вашего домена в config.toml (Формат указан выше).

Теперь вы можете обращаться к вашему приложению напрямую через локальное доменное имя.

```console
$ curl --insecure https://test_node.local
{"request_num":0,"host":"7417ac8fda92"}
```


## Добавление локальных сервисов не из докера

Все те флаги, которые мы указываем в labels, вы также можете указать в файле конфигурации рядом с docker-compose.yml, указав конкретный ip адрес.

Например:
```toml
[http.routers]
# Define a connection between requests and services
  [http.routers.<router_name>]
    rule = "Host(`my_app.local`)"
    entrypoints = "https"
    service = "<service_name>"
    [http.routers.<router_name>.tls]

[http.services]
  # Define how to reach an existing service on our infrastructure
  [http.services.<service_name>.loadBalancer]
     [[http.services.<service_name>.loadBalancer.servers]]
       url = "http://192.168.1.89:8100"
```


# Создание локального DNS
В данном пункте я бы хотел рассказать, как настроить свой DNS-сервер, чтобы ваши домены были доступны всем устройствам в локальной сети. Для этого я буду использовать dnsmasq. Пользователям винды он недоступен, поэтому советую развернуть маленький домашний сервер на линуксе. 

Для этого установите `dnsmasq` и найдите и раскомментируйте, либо добавьте следующие строчки в файл `/etc/dnsmasq.conf`:

```conf
# Never forward plain names (without a dot or domain part)
domain-needed
# Never forward addresses in the non-routed address spaces.
bogus-priv
address=/.local/192.168.1.89
address=/.<other_domain>/<your_local_ip>
```

У меня traefik развернут на хосте `192.168.1.89`. У вас ip может отличаться.
Чтобы это узнать, посмотрите свой ip через роутер или выполните `ip addr`.

Вообще, `dnsmasq` парсит файл `/etc/hosts` и вы можете туда добавлять записи, типа:
```
192.168.1.1	mydomain.local
```

Но так как я указал `address`, то это необязательно. `dnsmasq` и без явного указания поддоменов должен будет работать отлично.

Запустите `dnsmasq` в режиме сревиса:
```bash
sudo systemctl enable dnsmasq.service
sudo systemctl start dnsmasq.service
```

Теперь пойдите в настройки вашего роутера и найдите DNS-сервера. Добавьте туда ваш ip. Также не забудьте сделать локальный ip вашего устройства статичным.

Для примера, в роутерах keenetic это можно сделать, зарегистрировав устройство в меню 'Список устройств' и нажав на галочку 'Постоянный IP-адрес'.

И добавьте свой DNS-сервер в список DNS серверов вашего роутера.
![Keenetic interface](/images/traefik_imgs/keenetic_dns.png)

Готово. Вы можете попробовать зайти на свой домен с другого устройства в локальной сети, и это должно работать.

# Мониторинг работы traefik

Вообще, traefik имеет WEB интерфейс для отслеживания его работы.
Для того чтобы подключить его, давайте поменяем `docker-compose.yml`.

```yaml
services:
  traefik-proxy:
    # The official v2.0 Traefik docker image
    image: traefik:v2.4.8
    container_name: traefik_main
    labels:
      - traefik.enable=true
      - traefik.http.routers.traefik_router.rule=Host(`traefik.local`)
      - traefik.http.routers.traefik_router.service=api@internal
      - traefik.http.routers.traefik_router.entrypoints=https
      - traefik.http.routers.traefik_router.tls=true

    command: 
      - --api.dashboard=true
	  ...
```

Я добавил ключ `--api.dashboard=true` и лейблы для роута до `traefik`.
Замечу, service определён заранее - это `api@internal`. Укажите его в `traefik.http.routers.<router_name>.service`.

Теперь вы можете зайти на `traefik.local` через свой браузер и увидеть конфигурацию traefik.

[![GIF](/images/traefik_imgs/traefik_web.png)](/images/traefik_imgs/traefik_web.png)
