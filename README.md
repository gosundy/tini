<!--

#####################################
# THIS FILE IS AUTOGENERATED!       #
# Edit ./tpl/README.md.in instead   #
#####################################

-->


Tini - A tiny but valid `init` for containers
=============================================

[![Build Status](https://travis-ci.org/krallin/tini.svg?branch=master)](https://travis-ci.org/krallin/tini)

Tini is the simplest `init` you could think of.

All Tini does is spawn a single child (Tini is meant to be run in a container),
and wait for it to exit all the while reaping zombies and performing
signal forwarding.


Using Tini
----------

*NOTE: There are [pre-built Docker images available for Tini][10]. If
you're currently using an Ubuntu or CentOS image as your base, you can use
one of those as a drop-in replacement.*

*NOTE: There are Tini packages for Alpine Linux and NixOS. See below for
installation instructions.*

Add Tini to your container, and make it executable. Then, just invoke Tini
and pass your program and its arguments as arguments to Tini.

In Docker, you will want to use an entrypoint so you don't have to remember
to manually invoke Tini:

    # Add Tini
    ENV TINI_VERSION v0.8.4
    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
    RUN chmod +x /tini
    ENTRYPOINT ["/tini", "--"]

    # Run your program under Tini
    CMD ["/your/program", "-and", "-its", "arguments"]
    # or docker run your-image /your/program ...

Note that you *can* skip the `--` under certain conditions, but you might
as well always include it to be safe. If you see an error message that
looks like `tini: invalid option -- 'c'`, then you *need* to add the `--`.

Arguments for Tini itself should be passed like `-v` in the following example:
`/tini -v -- /your/program`.

*NOTE: The binary linked above is a 64-bit dynamically-linked binary.*


### Signed binaries ###

The `tini` and `tini-static` binaries are signed using the key `0527A9B7`.

You can verify their signatures using `gpg` (which you may install using
your package manager):

    ENV TINI_VERSION v0.8.4
    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini.asc /tini.asc
    RUN gpg --keyserver ha.pool.sks-keyservers.net --recv-keys 0527A9B7 \
     && gpg --verify /tini.asc


### Alpine Linux Package ###

On Alpine Linux, you can use the following command to install Tini (currently
available as a `testing` package):

    apk add --update --repository http://dl-1.alpinelinux.org/alpine/edge/testing/ tini


### NixOS ###

Using Nix, you can use the following command to install Tini:

    nix-env --install tini


Options
-------

### Verbosity ###

The `-v` argument can be used for extra verbose output (you can pass it up to
3 times, e.g. `-vvv`).


### Subreaping ###

By default, Tini needs to run as PID 1 so that it can reap zombies (by
running as PID 1, zombies get re-parented to Tini).

If for some reason, you cannot run Tini as PID 1, you should register Tini as
a process subreaper instead (only in Linux >= 3.4), by either:

  + Passing the `-s` argument to Tini (`tini -s -- ...`)
  + Setting the environment variable `TINI_SUBREAPER`
    (e.g. `export TINI_SUBREAPER=`).

This will ensure that zombies get re-parented to Tini despite Tini not running
as PID 1.

*NOTE: Tini will issue a warning if it detects that it isn't running as PID 1
and isn't registered as a subreaper. If you don't see a warning, you're fine.*


### Process group killing ###

By default, Tini only kills its immediate child process.  This can be
inconvenient if sending a signal to that process does have the desired
effect.  For example, if you do

    docker run krallin/ubuntu-tini sh -c 'sleep 10'

and ctrl-C it, nothing happens: SIGINT is sent to the 'sh' process,
but that shell won't react to it while it is waiting for the 'sleep'
to finish.

With the `-g` option, Tini kills the child process group , so that
every process in the group gets the signal. This corresponds more
closely to what happens when you do ctrl-C etc. in a terminal: The
signal is sent to the foreground process group.


More
----

### Existing Entrypoint ###

Tini can also be used with an existing entrypoint in your container!

Assuming your entrypoint was `/docker-entrypoint.sh`, then you would use:

    ENTRYPOINT ["/tini", "--", "/docker-entrypoint.sh"]


### Statically-Linked Version ###

Tini has very few dependencies (it only depends on libc), but if your
container fails to start, you might want to consider using the statically-built
version instead:

    ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static /tini


### Size Considerations ###

Tini is a very small file (in the 10KB range), so it doesn't add much weight
to your container.

The statically-linked version is bigger, but still < 1M.


Building Tini
-------------

If you'd rather not download the binary, you can build Tini by running
`cmake . && make`.

Before building, you probably also want to run:

    export CFLAGS="-DPR_SET_CHILD_SUBREAPER=36 -DPR_GET_CHILD_SUBREAPER=37"

This ensure that even if you're building on a system that has old Linux Kernel
headers (< 3.4), Tini will be built with child subreaper support. This is
usually what you want if you're going to use Tini with Docker (if your host
Kernel supports Docker, it should also support child subreapers).


Understanding Tini
------------------

After spawning your process, Tini will wait for signals and forward those
to the child process, and periodically reap zombie processes that may be
created within your container.

When the "first" child process exits (`/your/program` in the examples above),
Tini exits as well, with the exit code of the child process (so you can
check your container's exit code to know whether the child exited
successfully).


Debugging
---------

If something isn't working just like you expect, consider increasing the
verbosity level (up to 3):

    tini -v    -- bash -c 'exit 1'
    tini -vv   -- true
    tini -vvv  -- pwd


Authors
=======

Maintainer:

  + [Thomas Orozco][20]

Contributors:

  + [Tianon Gravi][21]
  + [David Wragg][22]

Special thanks to:

  + [Danilo Bürger][23] for packaging Tini for Alpine
  + [Asko Soukka][24] for packaging Tini for Nix


  [10]: https://github.com/krallin/tini-images
  [20]: https://github.com/krallin/
  [21]: https://github.com/tianon
  [22]: https://github.com/dpw
  [23]: https://github.com/danilobuerger
  [24]: https://github.com/datakurre
