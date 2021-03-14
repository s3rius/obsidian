# Makefiles for dummies
If we want to describe Makefiles in a nutshell, than we might say it's thing for describing command to ease interaction with the project.

Initially Makefiles were created to conveniently compile projects written in any language.

# How does it work
It works pretty simple.

Here's an example of a `Makefile`:
```Makefile
# Generate bash file.
test.gen:
	@echo 'echo "Hi!"' > "test.gen"

# Run generated file test.gen
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
target places right after the colon. This is a dependency of the `run` target, it fires right before executing the `run` target. Each target can have multiple dependencies, they are spearated with space.

E.G.:
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

Let's call the `target3` target and see the output.
```console
$ make target3
echo "1"  
1  
echo "2"  
2  
echo "memes"  
memes
```

As you can see, Makefile has built the target dependency tree and didn't call the `target1` second time. 

## Hiding commands output

In previous example you can notice that make command printed out all shell commands for every target. You can hide the commands from the output by adding "@" at the start of a command.

Like this:
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

Now you can call targets and it won't print the actual commands. Like this:
```console
$ make target3
1
2
memes
```


## Generated files validation
Make files are often used for compiling C programs and sometimes you need to compile a part of a project and skip building this part if executable is already present. It's a basic functionality of Makefiles.

Let's change the first `Makefile` and call the `run` target twice.
```Makefile
# Generate bash file.
test.gen:
	echo 'echo "Hi!"' > "test.gen"

# Run generated file test.gen
.PHONY: run
run: test.gen
	sh "test.gen"
```

Now we can see which commands Makefile executes.

Let's run it.
```console
$ make run  
echo 'echo "Hi!"' > "test.gen"
sh "test.gen"  
Hi!  
$ make run  
sh "test.gen"
Hi!
```

As you can see, second time it has skipped the run of a `test.gen` target.
It's because name of a target is a filename and it will never execute the target if file with name equals to target name exists. That is, in that particular case the `test.gen` target have to generate the `test.gen` file during the execution, and if it's already present it doesn't update it. That's why it didn't run it for the second time.  

If you don't want to have that feature you can disable it by adding `.PHONY: ${target}` somewhere.
Like this:
```Makefile
.PHONY: test.gen
# Generate bash file
test.gen:
    echo 'echo "Hi!"' > "test.gen"

# Run generated file test.gen
.PHONY: run
run: test.gen
    sh "test.gen"
```

Now it executes `test.gen` every time it's called.
```console
$ make run
echo 'echo "Hi!"' > "test.gen"
sh "test.gen"
Hi!
$ make run
echo 'echo "Hi!"' > "test.gen"
sh "test.gen"
Hi!
```