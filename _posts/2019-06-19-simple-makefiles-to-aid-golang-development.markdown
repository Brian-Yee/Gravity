A Meta-Commentary on Learning Languages
---------------------------------------

I've recently been learning golang on the side given my full-time job uses Python as I think there
is a high amount of benefit to be derived from programming in different languages in your free time
for two very close but still distinct principal components<sup>[1](#pca)</sup> of reasoning:

1. Different languages encourage different ways of solving similar problems. Once understood they
   are added to your toolbox of knowledge for _groking_ solution patterns to various problems.
2. The things that are closest to us are also the most opaque. In a philosophical Heideggerian
   sense the benefits of a language can be forgotten because, in a way, languages are written to
   focus on their content and not the syntax expressing them.

The successive interplay of these two principal components allow us to grow by learning what is
truly a universal solution pattern and what are _handy_ pieces of notation that come along with the
ecosystem of a language that you may wish to leverage in other languages. It can feel especially
rewarding to learn a language immediately applicable without the need of additional module imports
for any language -- I think this is the allure of the command line and why people are very
passionate about their dotfiles. But even then, there exists a variety of languages for your
dotfiles (`bash`, `sh`, `zsh`, `fish`, ...) each with their own notations bringing us back to the
original two bullet points raised above! This is perhaps why customizable text editors that have
been around forever without as-easily hackable alternatives (yeah, yeah, bring on the pitch forks)
like `vi` or `emacs` can invoke an almost irrational passion similar to that of dotfiles but
[_turned up to 11_](https://vimeo.com/59380615).

The Tip of the Iceberg
----------------------

For the reasons mentioned in the prior section, I find the lack of love I see the `make` language
recieve rather surprising. For the uninitiated `make` is a language that you will find
on pretty much any environment you are developing on (similar to `vi`) that you can use to execute
commands in a repository. For example a sample file written in `make` (called a `Makefile`) is

```makefile
run:
   go run main.go
```

where now to run your `main.go` program you simply type `make run`. Related to the first initial
bullet points this language helps you _build_ the things closest to you so you can forget about the
syntax and worry about the implementation details. It's mainly seen in aiding compiling languages
such as C or C++ tt keep track of the state of many compiling files. To aid this use-case if you
write the "target" (the string left of the colon; in the case above `run`) as a file name it will
only execute it if the file _does not exist_. You can try this out with the following `Makefile`:


```makefile
my-file.txt:
   echo "Hello" >> my-file.txt
```

if you run it twice you will find that it only wrote "Hello" to `my-file.txt` once. In fact `make`
thinks every target is a file unless otherwise specified and will run it only if it cannot find the
file. To ensure it runs every time even if a file exists with the same name in your repo you add
what's called a phony argument:

```makefile
.PHONY: run
run:
   go run main.go
```

This only _scratches_ the surface of what one can do with `make` and in a future post I will
elaborate further. However I have introduced all the notation required for the core purpose of this
post which is to present:

**An introductory, simple, _non-intimidating_ Makefile for writing golang**

hopefully after some initial use the dev's interest will be piqued and they can read on to find
additional resources on the subject.

A Simple Go Makefile
--------------------

The language golang is highly opinionated including its own: formatter, testing suite, doc
generator and execution command. Each of these can be written as a _recipe_ (commands to execute)
for `make` target that's easily memorable for the developer.


```makefile
run:
	go run main.go

clean:
	go fmt

test:
	go test -v

race:
	go test -race

bench:
	go test -bench=.

coverage:
	go test -cover

docs:
	open "http://localhost:6060/pkg/"
	godoc -http=:6060
```

Now for example if you want to check for race conditions you simply type `make race`. It's
understandable if you might have a folder in your repository called `docs` for one reason or
another and if you do you would see the `docs` target fail as alluded to in the prior section. Go
ahead try it!

``` shellsession
> mkdir docs
> make docs
make: `docs' is up to date.
```

Because the `Makefile` looks to see if a file exists, it falsely believes that the purpose of the
`docs` target is to make a `docs` folder -- and since a `docs` folder exists there is nothing to be
done. To remedy this we simply throw a `.PHONY` attribute at the start and all the targets to say
that they should run regardless of the names of what is saved in the directory.

```makefile
.PHONY: test race clean bench coverage docs
...
```

now when we call the target we see the recipe executed as expected

``` shellsession
> mkdir docs
> make docs
open "http://localhost:6060/pkg/"
godoc -http=:6060
```

Sometimes it can be hard to read so many targets attached to a single phony target so many will opt
to put a `.PHONY` target above all targets that the phony references. Additionally, it can be nice
to give a little help to indicate what our recipes do in plain English for the developer who may
uses the `Makefile`. Incorporating both practices mentioned in this paragraph we will write a
`help` target as

```makefile
.PHONY: help
help:
   @echo "help            Display this message."
   @echo "clean           Format Go files."
   @echo "test            Run unit tests."
   @echo "race            Search for possible race conditions."
   @echo "bench           Run benchmark tests"
   @echo "coverage        Calculate code coverage."
   @echo "docs            Generate docs in default browser"

...
```

The `@echo` line is used to avoid printing out to `stdin` the commands the recipe is executing, so
now when we run the `help` target we print out a nice clean summary of what the Makefile can do.

```shellsession
> make help
help        Display this message.
clean       Format Go files.
test        Run unit tests.
race        Search for possible race conditions.
bench       Run benchmark tests
coverage    Calculate code coverage.
docs        Generate docs in default browser
```

More generally if you just type `make` it will show this same output as `make` will run the recipe
at the top if not passed any targets. One last thing to add is that after writing the initial
script I realized I constantly ran two targets in succession:

```
> make clean
go fmt
> make test
go test -v make run
```

while this isn't a deal-breaker, in my `Makefile` I can say that `test` is dependent on `clean` so
that every time the `test` target is run the `clean` target is run first. This can be expressed in
the following syntax

```makefile
...

test: clean
	go fmt

...
```

There is no limit to how many targets I can make dependent and we've actually seen multiple
dependencies before when assigning all targets to a `.PHONY` target. Now whenever I run `test` it
will auto-format my code and then run the unit-tests.

```shellsession
> make test
go fmt
go test -v
```

Wrapping it Up
--------------

In summary, `make` is a useful language which requires very little knowledge to start leveraging in
your work-flow. Of course the complexities and powers of `make` extends much farther than this post
but it is the intent of this post solely to whet the reader's appetite for learning more about how
they can improve their development work-flow with it. It has been aimed at an audience of golang
developers but the language can be used for any general purpose ranging from compiling C code,
running Python scripts or passing off a `Dockerfile` that requires fancy mounting to someone who
may not be familiar with Docker. Hopefully, even if you are not (or are) a golang developer reading
this it's beautiful simplicity and widely applicable uses will aid you in your future development
if you are not leveraging the powers of it already.

Happy Hacking,  
Brian

Footnotes:
----------

<a name="pca">1</a>: Here principal components are meant to allude towards the idea of Principal
Component Analysis (PCA) they are not claimed to be the two _most important_ or _most
understandable_ but rather the _most descriptive_ ways of viewing some data we are observing by
maximizing the variance of an argument one can derive and build from them.
