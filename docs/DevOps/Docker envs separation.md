# Separating docker envs

Should we separate docker environments in several `docker-compose` files in your projects? Definitely! In some cases you can't figure out what developers wanted to do or why project doesn't work. Setting up separation between dev and prod environments can become a mishmash. In this article I'll show you the easiest way for setting up `docker-compose` for your project and how to avoid millon problems.

How can we separate environments for local development and production?
The answer is simple. We need to decompose our project and create separate files for environments or even a services.

It's totally ok if you have more than 2 `docker-compose` files.

For example:
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

### How does it work?

Docker can work with multiple `docker-compose` files simultaneously. And we can use it for environment separation. It looks like this:

```bash
docker-compose \
	-f "deploy/docker-compose.yml" \
	-f "deploy/docker-compose.db.yml" \
	-f "deploy/docker-compose.dev.yml" \
	up
```

In each `docker-compose` file we define a part of the configuration, which doesn't inersect with others. For example: `docker-compose.yml` defines our application and some required services. In other compose files we define services or override existing configurations defined in previous files.

I guess, it simpler to show it by example.

Let's imagine, that you have a project that can be started with different parameters, and they are different for local and production.

Let's create simple project with the following stucture:

```
proj
├── deploy
│   ├── docker-compose.yml  
│   └── dockerfiles  
│       └── script.Dockerfile  
└── script.py
```

Let's write a simple python script.
```python
# script.py
from sys import argv # это аргуметы переданные в скрипт

def main():
	print("; ".join(argv[1:])) # выводитна экран все аргументы программы

if __name__ == "__main__":
	main()
```

After that we need to wrap it into docker container by creating `Dockerfile` in `deploy/dockerfiles/script.Dockerfile`

```dockerfile
from python:3.8 # asd

WORKDIR /app  
COPY script.py /app/script.py  
CMD python /app/script.py
```

As you can see, Dockerifle is very simple. Now let's create the main `docker-compose.yml`.

```yaml
---
# deploy/docker-compose.yml
version: '3.7'  
  
services:  
  script:  
        build:  # build applicationo.
            dockerfile: ./deploy/dockerfiles/script.Dockerfile  
            context: .  
        # Запускаем его с командой для продового запуска.
        command: python script.py this is prod
```

Now we can start this project with that command:

```bash
d-test docker-compose \
        -f "./deploy/docker-compose.yml" \
        --project-directory "." \
        run --rm script
```

It will print:
```log
Creating d-test_script_run ... done
this; is; prod
```

As we can see it has printed a message we've passed in our `docker-compose.yml`.

Now we are going to configure environment for local development without changing the main `docker-compose.yml`. Let's create the `docker-compose.dev.yml`

```yaml
---
# deploy/docker-compose.dev.yml
version: '3.7'  
  
services:  
  script:
    command: python script.py this is dev
```

Now add this file in our startup script and run it.

```bash
d-test docker-compose \
        -f "deploy/docker-compose.yml" \
        -f "deploy/docker-compose.dev.yml" \
        --project-directory "." \
        run --rm script
```

It'll print the following:
```log
Creating d-test\_script\_run ... done  
this; is; dev
```

As you can see, deploy configuration was overwritten with our new file.

Any mentioned file in this command can add new services or *partially* override previously mentioned files.

Complete project structure:
```
proj
├── deploy  
│   ├── docker-compose.dev.yml  
│   ├── docker-compose.yml  
│   └── dockerfiles  
│       └── script.Dockerfile  
└── script.py
```

### Where can I use that?
In every project bigger than this one. Because the real life not so friendly and local environment of the project can be different to the production not only by configuration, but the whole services.