# Разделение докера на среды

Должен ли ты разделять среды докера в несколько `docker-compose` файлов?
Определенно! В некоторых случаях невозможно разобраться что разработчики хотели сделать или почему ничего не работает. Настройка раздельных сред может стать настоязей мешанино. В этой статье я покажу как настроить `docker-compose` и избежать миллиона проблем.

Как мы вообще можем разделить среды для локальной разработки и продовые?
Отет прост: Требуется декомпозировать проект и создать отдельные файлы под каждую из сред или даже сервисов.

Это нормально, если у тебя будет больше 2-ч `docker-compose` файлов. 

Например:
```
deploy  
├── docker-compose.autotests.yml
├── docker-compose.db.yml  
├── docker-compose.dev.yml  
├── docker-compose.yml  
├── dockerfiles  
│   ├── api.Dockerfile  
│   └── front.Dockerfile  
└── scripts  
   ├── start-autotests.sh  
   ├── start-backend.sh
   └── start-migrations.sh
```

### Как это работает?
Докер умеет работать с множеством `docker-compose` файлов одновременно.И мы можем использовать это для разделения сред.

Выглядит это следующим образом.
```bash
docker-compose \
	-f "deploy/docker-compose.yml" \
	-f "deploy/docker-compose.db.yml" \
	-f "deploy/docker-compose.dev.yml" \
	up
```

В каждом из этих файлов определен какой-то кусок конфигурации, который не пересекается. Например в `docker-compose.yml` определено приложение и некоторые необходимые сервисы, а в остальных файлах добавляются сервисы или меняются значения предыдущих.

Наверное, тут проще на примере пояснить.

Допустим у нас есть проект, у которого поднимается бекенд с параметрами, которые отличаются на проде и локально.

Для простоты создадим простецикий проект со следующей структурой.

```
proj
├── deploy
│   ├── docker-compose.yml  
│   └── dockerfiles  
│       └── script.Dockerfile  
└── script.py
```

Для начала напишем скрипт, который будет в центре всего.
```python
# script.py
from sys import argv # это аргуметы переданные в скрипт

def main():
	print("; ".join(argv[1:])) # выводитна экран все аргументы программы

if __name__ == "__main__":
	main()
```


После того, как скрипт готов и отлажен давайте завернем его в докер,
создав `Dockerfile` в `deploy/dockerfiles/script.Dockerfile`.

```dockerfile
from python:3.8 # asd

WORKDIR /app  
COPY script.py /app/script.py  
CMD python /app/script.py
```

Как вы видите, Dockerfile очень прост. Теперь добавим главный `docker-compose.yml`.

```yaml
---
# deploy/docker-compose.yml
version: '3.7'  
  
services:  
  script:  
        build:  # Собираем приложение используя наш dockerfile.
            dockerfile: ./deploy/dockerfiles/script.Dockerfile  
            context: .  
        # Запускаем его с командой для продового запуска.
        command: python script.py this is prod
```

Теперь запустим всё это чудо.

```bash
d-test docker-compose \
        -f "./deploy/docker-compose.yml" \
        --project-directory "." \
        run --rm script
```

Вот что будет выведено на экран.
```log
Creating d-test_script_run ... done
this; is; prod
```

Как мы видим, на экран вывелось сообщение, которое мы указали в нашем `docker-compose.yml`

А теперь для локальной разработки мы не будем ничего менять в нашем файле композиции, а создадим новый рядом.

```yaml
---
# deploy/docker-compose.dev.yml
version: '3.7'  
  
services:  
  script:
    command: python script.py this is dev
```

Теперь добавим ещё один файл в нашу команду запуска.

```bash
d-test docker-compose \
        -f "deploy/docker-compose.yml" \
        -f "deploy/docker-compose.dev.yml" \
        --project-directory "." \
        run --rm script
```

Вот что будет выведено на экран:
```log
Creating d-test\_script\_run ... done  
this; is; dev
```

Как можно заметить, конфигурация запуска перезаписалась в порядке вызова.

То есть, каждый последующий файл композиции может добавлять сервисы и **частично** изменять конфигурацию предыдущих.

Итоговая структура проекта:
```
proj
├── deploy  
│   ├── docker-compose.dev.yml  
│   ├── docker-compose.yml  
│   └── dockerfiles  
│       └── script.Dockerfile  
└── script.py
```

### Где это применимо?
Ну, в любом проекте, сложнее того, который мы рассмотрели. Потому что в реальной жизни не всё так радужно и локальная версия приложения может отличаться не только параметрами запуска, но и целыми сервисами, которые требуются для локальной копии приложения.