---
title: "Laravel Sail and M1 Mac"
date: 2021-05-23T14:32:07-04:00
hero: /images/heros/hero-space.png
images: "images/"
menu:
  sidebar:
    name: "Laravel Sail and M1 Mac"
    identifier: scheduling
    weight: -270
tags: [laravel, note]
---

Some notes to self will add more after time if needed.

## Mac M1

Had a MySQL error:

```
ERROR: no matching manifest for linux/arm64/v8 in the manifest list entries
```

All I had to do was update `docker-composer.yml`:

```
File: docker-compose.yml
24:     mysql:
25:         image: 'mysql:8.0'
26:         ports:
27:             - '${FORWARD_DB_PORT:-3306}:3306'
```

To:

```
  mysql:
    image: mariadb:10.5.8
    ports:
      - "${FORWARD_DB_PORT:-3306}:3306"
```

Then I disabled `meilisearch` else everytime I ran `sail anything` it would stop the docker container.

```
File: docker-compose.yml
50:   #   meilisearch:
51:   #     image: "getmeili/meilisearch:latest"
52:   #     ports:
53:   #       - "${FORWARD_MEILISEARCH_PORT:-7700}:7700"
54:   #     volumes:
55:   #       - "sailmeilisearch:/data.ms"
56:   #     networks:
57:   #       - sail
```

I will have to come back to this one to fix it some info is [here](https://github.com/meilisearch/MeiliSearch/issues/1195)

## Links

- [Xdebug](https://medium.com/geekculture/debug-your-laravel-sail-applications-with-xdebug-160ad70fcd41)
- Some overall m1 issues [here](https://github.com/laravel/sail/issues/104)
