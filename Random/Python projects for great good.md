### Как стартовать проект

В целом, вариантов несколько. Например, можно просто хранить несколько скриптов на github и бть довольным, можно дописать к ним `setup.py` и выложиться на pypi. Но лучший способ - структурированный проект с `poetry`.

Почему?

Особенностей у poetry много. Сейчас расскажу про то, как использовать и что делать.

Для начала создадим проект.

```
➜ poetry new new_proj
Created package new_proj in new_proj
```

Данная команда сгенерирует следующую структуру:
```
new_proj
├── new_proj
│   └── __init__.py
├── pyproject.toml
├── README.rst
└── tests
    ├── __init__.py
    └── test_new_proj.py
```

Это наш новый проект с небольшими заготовками. В целом, тут нет ничего необычного. Весь код библиотеки/проекта будет в папке с названием проекта. `README.rst` - дефолтный README. `pyproject.toml` - данные о проекте для упаковщика.

### Что делать с poetry

Вся информация есть в оф. документации тулы. Тут я расскажу про основные моменты.

#### установка зависимостей

Чтобы добавить зависимость проекта достаточно выполнить
```bash
poetry add ${dependency}
```

Данная команда найдет последнюю нужную версию и запишет её в pyproject.toml и в poetry.lock.

Для того, чтобы установить зависимость для разработки (линтеры например), достаточно добавить флаг `--dev`.

```bash
poetry add ${dependency} --dev 
```

#### Выполнение команд

При использовании virtualenv для входа в оболочку всегда надо было вводить
```bash
source venv/bin/activate
```

Однако с poetry это излишне. Он сам создает и менеджит виртуальные среды.
Чтобы выполнить одиночную команду можно вызвать `run`. 

Например:
```bash
poetry install black --dev  # установим cli-программу
poetry run black # запустим её в папке
```

Для выполнения нескольких комманд подряд, можно войти в shell. Это аналог `source venv/bin/activate`.

```bash
poetry shell # зашли в оболочку
black # вызвали программу установленную в оболочку
```

#### Версионирование

Менять версии пакета вручную больше не нужно.

У poetry есть кое что для тебя.
```
➜ poetry version patch   
Bumping version from 0.1.0 to 0.1.1

➜ poetry version preminor
Bumping version from 0.1.1 to 0.2.0-alpha.0

➜ poetry version minor   
Bumping version from 0.2.0-alpha.0 to 0.2.0

➜ poetry version premajor
Bumping version from 0.2.0 to 1.0.0-alpha.0

➜ poetry version major   
Bumping version from 1.0.0-alpha.0 to 1.0.0
```

### pyproject.toml

Данный файл содержит в себе все нужные зависимости для проекта и всяческую мета-информацию.

Пример pyproject.tom
```toml
[tool.poetry]
name = "new_proj"
version = "0.1.0"
description = "Test library for example"
readme = "README.rst"
homepage = "https://test_url.com/"
repository = "https://github.meme/"
authors = ["Pavel Kirilin <win10@list.ru>"]

[tool.poetry.dependencies]
python = "^3.8"

[tool.poetry.dev-dependencies]
pytest = "^6.1"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

Для того чтобы устновить все зависимости требуется просто ввести
```
poetry install
```

Данная команда создаст виртуальную среду сама и установит **все** зависимости. Включая dev-зависимости. Чтобы установить зависимости только для приложения можно добавить `--no-dev` ключ. 

### Как паковать и публиковать на pypi
Теперь я вас удивлю. Всё что сейчас делалось, по большей чати нужно было для легкой упаковки. Сейчас поясню.

Давайте добавим какую-нибудь функцию в проект.

```python
# new_proj/main.py
def ab_problem(a: int, b: int) -> int:
    return a + b
```

```python
# new_proj/__init.py
from new_proj.main import ab_problem

__all__ = [
    'ab_problem'
]
```

Теперь соберем проект.

```bash
➜ poetry build
Building new_proj (0.1.0)
  - Building sdist
  - Built new_proj-0.1.0.tar.gz
  - Building wheel
  - Built new_proj-0.1.0-py3-none-any.whl
```

Та-да. Это готовый к публикации на pypi пакет. Лежит он в папке dist.

Давайте проверим, что всё работает корректно.

```bash
➜ pip install "./dist/new_proj-0.1.0-py3-none-any.whl"  
Processing ./dist/new_proj-0.1.0-py3-none-any.whl
Installing collected packages: new-proj
Successfully installed new-proj-0.1.0

➜  python
Python 3.9.1 (default, Feb  1 2021, 04:02:33) 
[GCC 10.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from new_proj import ab_problem
>>> ab_problem(1,33)
34
```

Как можно видеть всё работает корректно и теперь мы можем использовать наш пакет.

Для публикации следует использовать:
```bash
poetry publish -u "user" -p "password"
```

Подробнее можно почитать [тут](https://python-poetry.org/docs/cli/#publish).

### Конфигурация проекта
Конечно, такого рода конфигурация проекта всё равно никуда не годиться.
Давайте настроим автоматический линтинг.

```bash
poetry add \
	flake8 \
	black \
	isort \
	mypy \
	pre-commit \
	yesqa \
	autoflake \
	wemake-python-styleguide --dev
```

Теперь добавим конфигурационных файлов в корень проекта.

`.mypy.ini` для настройки валидации типов.
```ini
[mypy]
strict = True
ignore_missing_imports=True
allow_subclassing_any=True
allow_untyped_calls=True
pretty=True
show_error_codes=True
implicit_reexport=True
allow_untyped_decorators=True
```

`.isort.cfg` для конфигурации сортировки импортов.
```ini
[isort]
multi_line_output = 3
include_trailing_comma = true
use_parentheses = true
```

`.flake8` - конфигурация линтинга. Тут довольно много. Это игнорирование ненужных кодов ошибок, которые не особо-то и ошибки.
```ini
[flake8]
max-complexity = 6
inline-quotes = double
max-line-length = 88
extend-ignore = E203
docstring_style=sphinx

ignore =
  ; Found `f` string
  WPS305,
  ; Missing docstring in public module
  D100,
  ; Missing docstring in magic method
  D105,
  ; Missing docstring in __init__
  D107,
  ; Found class without a base class
  WPS306,
  ; Missing docstring in public nested class
  D106,
  ; First line should be in imperative mood
  D401,
  ; Found `__init__.py` module with logic
  WPS412,
  ; Found implicit string concatenation
  WPS326,
  ; Found string constant over-use
  WPS226,
  ; Found upper-case constant in a class
  WPS115,
  ; Found nested function
  WPS430,
  ; Found using `@staticmethod`
  WPS602,
  ; Found method without arguments
  WPS605,
  ; Found overused expression
  WPS204,
  ; Found too many module members
  WPS202,
  ; Found too high module cognitive complexity
  WPS232,
  ; line break before binary operator
  W503,
  ; Found module with too many imports
  WPS201,
  ; Found vague import that may cause confusion: X
  WPS347,
  ; Inline strong start-string without end-string.
  RST210,
  ; subprocess call with shell=True seems safe, but may be changed in the future.
  S602,
  ; Starting a process with a partial executable path.
  S607,
  ; Consider possible security implications associated with subprocess module.
  S404,
  ; Found nested class
  WPS431,
  ; Found wrong module name
  WPS100,
  ; Found too many methods
  WPS214,
  ; Found too long ``try`` body
  WPS229,
  ; Found function with too much cognitive complexity
  WPS231,
  
  ; all init files
  __init__.py:
  ; ignore not used imports
  F401,
  ; ignore import with wildcard
  F403,
  ; Found wrong metadata variable
  WPS410,

per-file-ignores =
  ; all tests
  test_*.py,tests.py,tests_*.py,*/tests/*:
  ; Use of assert detected
  S101,

exclude =
  ./.git,
  ./venv,
  ./cached_venv,
  ./var,
```

`.pre-commit-config.yaml` - конфигураци хуков для запуска всех линтеров перед коммитом.
```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.4.0
    hooks:
      - id: check-ast
      - id: trailing-whitespace
      - id: check-toml
      - id: end-of-file-fixer

  - repo: https://github.com/asottile/add-trailing-comma
    rev: v2.1.0
    hooks:
    -   id: add-trailing-comma

  - repo: local
    hooks:
      - id: black
        name: Format with Black
        entry: black
        language: system
        types: [python]

      - id: autoflake
        name: autoflake
        entry: autoflake
        language: system
        types: [ python ]
        args: [ --in-place, --remove-all-unused-imports, --remove-duplicate-keys ]

      - id: isort
        name: isort
        entry: isort
        language: system
        types: [ python ]

      - id: flake8
        name: Check with Flake8
        entry: flake8
        language: system
        pass_filenames: false
        types: [ python ]
        args: [--count, .]

      - id: mypy
        name: Validate types with MyPy
        entry: mypy
        language: system
        types: [ python ]

      - id: yesqa
        name: Remove usless noqa
        entry: yesqa
        language: system
        types: [ python ]

      - id: pytest
        name: pytest
        entry: pytest
        language: system
        pass_filenames: false
        types: [ python ]
```

И не забываем про `.gitignore`. Его можно найти [тут](https://github.com/github/gitignore/blob/master/Python.gitignore).

ОН НЕОБХОДИМ ЧТОБЫ pre-commit РАБОТЛ МАКСИМАЛЬНО КОРРЕКТНО.

Теперь установим хуки в репозиторий.
```bash
➜ git init
Initialized empty Git repository in .git/

➜ poetry shell

➜ pre-commit install
pre-commit installed at .git/hooks/pre-commit

➜ git commit
... # Упадет с кучей ошибок
```

Теперь поправим все возникшие проблемы.

Во первых исправим тесты.

В `tests/test_new_proj.py` напишем следующее:
```python
from new_proj import ab_problem


def test_ab() -> None:
    """AB problecm success case."""
    assert ab_problem(1, 2) == 3
```

Добавим описания в `__init__` файлы.

`tests/__init__.py`:
```python
"""Tests for new_proj."""
```

`new_proj/__init__.py`:
```python
"""Project for solving ab problem."""
from new_proj.main import ab_problem

__all__ = [
    "ab_problem",
]
```

Пофиксим основной файл проекта.

`new_proj/main.py`:
```python
def ab_problem(first: int, second: int) -> int:
    """
    Solve AB problem.

    The function sums two integers.

    :param first: a argument.
    :param second: b argument.
    :returns: sum.
    """
    return first + second
```

Теперь вы можете сделать свой первый коммит.

```bash
(new-proj-I04oatEq-py3.9) ➜ git commit     
Check python ast.........................................................Passed
Trim Trailing Whitespace.................................................Passed
Check Toml...............................................................Passed
Fix End of Files.........................................................Passed
Add trailing commas......................................................Passed
Format with Black........................................................Passed
autoflake................................................................Passed
isort....................................................................Passed
Check with Flake8........................................................Passed
Validate types with MyPy.................................................Passed
Remove usless noqa.......................................................Passed
pytest...................................................................Passed
```

## Создание CLI-приложения
А что если я хочу cli-приложение?
Ты не представляешь насколько это просто.

Пойдем модифицируем наш основной файл.

```python
import argparse


def parse_args() -> argparse.Namespace:
    """
    Parse CLI arguments.

    :returns: parsed namespace.
    """
    parser = argparse.ArgumentParser()

    parser.add_argument("a", type=int)
    parser.add_argument("b", type=int)

    return parser.parse_args()


def ab_problem(first: int, second: int) -> int:
    """
    Solve AB problem.

    The function sums two integers.

    :param first: a argument.
    :param second: b argument.
    :returns: sum.
    """
    return first + second


def main() -> None:
    """Main function."""
    args = parse_args()
    print(ab_problem(args.a, args.b))  # noqa: WPS421

```

Теперь поправим pyproject.toml таким образом чтобы он создал cli для нашей функции.

Добавим следующую секцию куда-нибудь в `pyproject.toml`:
```toml
[tool.poetry.scripts]
ab_solver = "new_proj.main:main"
```

Теперь нам доступна программа `ab_solver` внутри shell.

```bash
➜  poetry install

➜ poetry shell      

(new-proj-I04oatEq-py3.9) ➜ ab_solver 1 2
3
```

Хочешь установить? Пожалуйста.
```bash
➜ poetry build
Building new_proj (0.1.0)
  - Building sdist
  - Built new_proj-0.1.0.tar.gz
  - Building wheel
  - Built new_proj-0.1.0-py3-none-any.whl

➜ pip install "./dist/new_proj-0.1.0-py3-none-any.whl"
Processing ./dist/new_proj-0.1.0-py3-none-any.whl
Installing collected packages: new-proj
Successfully installed new-proj-0.1.0

➜ ab_solver 1 2                                     
3
```

Если запаблишить проект, то у пользователя тоже установится ваша cli-программа.