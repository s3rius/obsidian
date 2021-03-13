# Makefiles for dummies
If we want to describe Makefiles in a nutshell, than we might say it's thing for describing command to ease interaction with the project.

Initially Makefiles were created to conveniently compile projects written in any language.

# How does it work
It works pretty simple.

Here's an example of a `Makefile`:
```Makefile
# Сгенерируем простой файл
test.gen:
	@echo 'echo "Hi!"' > "test.gen"

# Команда для запуска файла test.gen
.PHONY: run
run: test.gen
	sh "test.gen"
```

Before colons we define our targets. E.G in this example we have `test.gen` and `run` targets.

These targets can be triggered with command like `make ${target}`.

If we enter `make run` in terminal in a dir with `Makefile` we'll get the following:
```console
sh "test.gen"  
Hi!
```

As we can see in output, we successfully ran the `run` target which read the `test.gen` file, but we didn't run `make test.gen` target. What happened?

Let's dive into it.

## Targets dependencies

On the line where we define our `run` target, we can see the `test.gen`
target places right after the colon.

Это зависимость данного таргета и она будет вызвана до того, как выполнится скрипт описываемого таргета. Таких зависимостей может быть много, перечисляются они чере пробел.

Например:
```Makefile
.PHONY: target1
target1:
	echo "1"

.PHONY: target2
target2: target1
	echo "2"

.PHONY: target3
target3: target1 target2
	echo "memes"
```

При вызове `make target3` будет выведено:
```console
$ make target3
echo "1"  
1  
echo "2"  
2  
echo "memes"  
memes
```

Как можно видеть, он построил граф зависимостей и не выполнил `target1` дважды.

## Hiding commands output

В предыдущем примере можно заметить, что он написал все команды в терминал. Для того, чтобы этого избежать следует добавить "@" в начало команды и она не будет напечатана.

```Makefile
.PHONY: target1
target1:
	@echo "1"

.PHONY: target2
target2: target1
	@echo "2"

.PHONY: target3
target3: target1 target2
	@echo "memes"
```

Теперь при вызое `make target3` будет показано следующее:
```console
$ make target3
1
2
memes
```


## Generated files validation
Зачастую `Makefile` используют для компиляции С и зачастую требуется
собрать какую-либо часть проект и пропустить сборку этой части, если эта часть уже собрана.
Раскрою секрет, в Makefile это базовый функционал.

Давайте немного поменяем первый Makefile и запустим дважды.
```Makefile
# Сгенерируем простой файл
test.gen:
	echo 'echo "Hi!"' > "test.gen"

# Команда для запуска файла test.gen
.PHONY: run
run: test.gen
	sh "test.gen"
```

Теперь вызываемая команда таргета `test.gen` выводится на экран.

Позапускаем.

```console
$ make run  
echo 'echo "Hi!"' > "test.gen"  # Наша командабыла вызвана.
sh "test.gen"  
Hi!  
$ make run  
sh "test.gen"  # Наша команда не вызвана.
Hi!
```

Дело в том, что названия таргетов - названия файлов, которые должны сгенерировать эти самые таргеты.
То есть, в данном случае таргет `test.gen` должен сгенерировать файл `test.gen` по окончании выполнения. Если этот файл уже присутствует, то команда выполнена не будет. Именно поэтому у нас она не запустилась второй раз, так как в первый запуск был создан треубемый файл и его никто не удалял между запусками.

А что если я вот не хочу чтобы команда создавала и проверяла файлы?
Для этого пишут `.PHONY: ${target}`. Например у нас так объявлен таргет `run` и, даже если файл с названием `run` будет присутствовать в директории цель не будет выполняться.