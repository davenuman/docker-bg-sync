# bg-sync

This container continuously syncs files between two directories. This is useful
for avoiding the filesystem slowness on Docker for Mac, for instance. It's also
generally useful for any other time where you have a slow filesystem as a source
of files that need to be read inside of a container.

## Usage

This is a basic `docker-compose.yml` that demonstrates usage of this container.

```

version: '2'

services:
  web:
    image: php:7.0-apache
    volumes:
      - /var/www/myapp

  bg-sync:
    image: cweagans/bg-sync
    volumes:
      - .:/source
    volumes_from:
      - web
    environment:
      - SYNC_DESTINATION=/var/www/myapp
      - SYNC_WAIT=30
      - SYNC_VERBOSE=1
    privileged: true
```

## Environment variables

This container uses values from a handful of environment variables. These are
documented below.

  * **`SYNC_SOURCE`** (default: `/source`): The path inside the container which
    will be used as the source of the file sync. Most of the time, you probably
    shouldn't change the value of this variable. Instead, just bind-mount your
    files into the container at `/source` and call it a day.
  * **`SYNC_DESTINATION`** (default: `/destination`): When files are changed in
    `SYNC_SOURCE`, they will be copied over to the equivalent paths in `SYNC_DESTINATION`.
    If you are using bg-sync to avoid filesystem slowness, you should set this
    path to whatever path the volume is at in your application container. In the
    example above, for instance, this would be `/var/www/myapp`.
  * **`SYNC_VERBOSE`** (default: "0"): Set this variable to "1" to get more log
    output from Unison.
  * **`SYNC_WAIT`** (default: "30"): Wait time in seconds between unison repeats.
  * **`SYNC_EXTRA_UNISON_PROFILE_OPTS`** (default: ''): The value of this variable
    will be appended to the end of the Unison profile that's automatically generated
    when this container is started. Ensure that the syntax is valid. If you have
    more than one option that you want to add, simply make this a multiline string.
    **IMPORTANT**: The *ability* to add extra lines to your Unison profile is
    supported by the bg-sync project. The *results* of what might happen because
    of this configuration is *not*. Use this option at your own risk.

## Why not use *x*?

I've probably tried it and didn't like it for one reason or another. If you think
that's not the case, open an issue. I'd love to be told about another project that
makes it possible to deprecate this one.

Specific reasons:

* **[Docker Sync](https://github.com/eugenmayer/docker-sync)**: Although the choice
  of sync backends is cool, I think it's better to offer just one and make sure
  that it works as close to perfect as is possible. The big deal breaker with
  Docker Sync, though, is that it requires installing extra stuff on the host.
  The entire point of containerization is that you shouldn't have to worry about
  that kind of stuff.
* **[Docker Magic Sync](https://github.com/mickaelperrin/docker-magic-sync)**:
  I like the idea - using Docker Gen to generate a set of Unison profiles + a
  supervisord config is pretty clever. However, I rarely need to sync more than
  one pair of directories, and when I do, each set of directories usually needs
  different configurations. With bg-sync, you can spin up multiple instances of
  the container, each with different Unison configurations if you want to.


## Credits

* Mickael Perrin - [docker-magic-sync](https://github.com/mickaelperrin/docker-magic-sync)
* Eugen Mayer - [docker-sync](https://github.com/eugenmayer/docker-sync)

