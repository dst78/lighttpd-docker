# lighttpd Docker image

Lighttpd on Alpine Linux, ready for SSL and virtual hosts.

Find the build scripts and example files on [github](https://github.com/dst78/lighttpd-docker/), and the docker image on [docker hub](https://hub.docker.com/repository/docker/anoikisnomads/lighttpd).

This version is based on Alpine Linux 3.18 and lighttpd 1.4.71

## Build

To build this yourself:

* fork my repo https://github.com/dst78/lighttpd-docker.git
* create a local clone of your repository
* within your local clone, build your docker image:
  * with docker: `docker build .`
  * OR with docker compose: `docker compose build`

## Usage

Before starting anything do have a look at `data/config/lighttpd.conf` and adapt the configuration to your needs. The `lighttpd.conf` file itself only contains the fundamentals. 

Any configuration of a lighttpd module or virtualhost config is assumed to be in individual files within `data/config/`: 

* add module configs as `data/config/mod_*.conf`
* add virtual host configs as `data/config/vhost_*.conf`

In a module config you will need to register the module itself before writing the actual configuration:

```
server.modules += (
  "mod_access",
)
```

### a word about HTTP and HTTPS

The image comes with mod_openssl and mod_rewrite enabled. Place certificates for the main host and all virtual hosts in `data/certs/`.

By default lighttpd will serve HTTP on port 80 and HTTPS on port 443.

If you want to automatically redirect HTTP traffic to HTTPS, have a look at `data/config/mod_rewrite.conf`.

### docker

The image is available on DockerHub, GitHub Packages and Quay.io. Pick the registry you are comfortable with.

* `docker pull anoikisnomads/lighttpd:latest`
* `docker pull quay.io/anoikisnomads/lighttpd:latest`
* `docker pull ghcr.io/dst78/lighttpd:latest`

### docker compose

A working docker-compose.yml file is provided as inpiration. Tweak to your needs.

The image is available on DockerHub, GitHub Packages and Quay.io. See the example docker-compose below and pick the one you're comfortable with.

### serving a single site

```
version: "3.0"
services:
  lighttpd:
    image: anoikisnomads/lighttpd:latest
#    image: quay.io/anoikisnomads/lighttpd:latest
#    image: ghcr.io/dst78/lighttpd:latest
    container_name: lighttpd
    hostname: lighttpd
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./data/config:/etc/lighttpd # lighttpd configs, including vhosts and mods
      - ./data/certs:/var/www/certs # place SSL certificates here
      - ./data/htdocs:/var/www/htdocs # htdocs for the default host
    restart: unless-stopped
    tty: true
```

### serving multiple sites through virtual hosts

```
version: "3.0"
services:
  lighttpd:
    image: anoikisnomads/lighttpd:latest
#    image: quay.io/anoikisnomads/lighttpd:latest
#    image: ghcr.io/dst78/lighttpd:latest
    container_name: lighttpd
    hostname: lighttpd
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./data/config:/etc/lighttpd # lighttpd configs, including vhosts and mods
      - ./data/certs:/var/www/certs # place SSL certificates here
      - ./data/htdocs:/var/www/htdocs # htdocs for the default host
      - ./data/vhosts/vhost1:/var/www/vhost1 # example folder for virtual host
      - ./data/vhosts/vhost2:/var/www/vhost2 # example folder for virtual host
    restart: unless-stopped
    tty: true
```

### user- and group id

By default the lighttpd user runs as user-id 100 and group-id 101. This means that if you set the server to write files on the mounted docker volumes, these files would be owned by whatever these IDs map on your host machine.

You can change the IDs of the user by supplying environment variables:

* PUID to set the user id 
* PGID to set the group id 

```
version: "3.0"
services:
  lighttpd:
    image: anoikisnomads/lighttpd:latest
#    image: quay.io/anoikisnomads/lighttpd:latest
#    image: ghcr.io/dst78/lighttpd:latest
    container_name: lighttpd
    hostname: lighttpd
    ports:
      - 80:80
      - 443:443
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./data/config:/etc/lighttpd
      - ./data/certs:/var/www/certs
      - ./data/htdocs:/var/www/htdocs
    restart: unless-stopped
    tty: true
```


## About

Originally written by [Sébastien Pujadas](http://pujadas.net), released under the [MIT license](http://opensource.org/licenses/MIT).

Forked and adapted by Dominique Stender, 2023.

