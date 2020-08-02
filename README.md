<img align="right" src="https://raw.githubusercontent.com/henkall/docker-cod4/master/cod4.ico">
[hub]: https://hub.docker.com/r/henkallsn/fivem_esx_bundle
[git]: https://github.com/Andruida/fivem

# [henkallsn/fivem_esx_bundle][hub]

This docker image allows you to run a server for FiveM, a modded GTA multiplayer program.
This image includes [txAdmin](https://github.com/tabarra/txAdmin), an in-browser server management software.
Upon first run, the configuration is generated in the host mount for the `/config` directory, and for the `/txData` directory (that contains the txAdmin configuration).

This bundle is made to run with a Mariadb server.

## Licence Key

A freely obtained licence key is required to use this server, which should be declared as `$LICENCE_KEY`. A tutorial on how to obtain a licence key can be found [here](https://forum.fivem.net/t/explained-how-to-make-add-a-server-key/56120)

## Usage

Use the docker-compose script provided:

```sh
---	  
version: '2'
services:
  fivem:
    image: henkallsn/fivem_esx_bundle:latest
    container_name: fivem
    restart: on-failure
    stdin_open: true
    tty: true
    volumes:
      # Remember to change.
      - "/path/to/resources/folder:/config"
      # Remember to change.
      - "/path/to/txAdmin/config:/txData"
      # DO NOT CHANGE THIS (sql file dir)
      - sqlfile:/sqlfile
    ports:
      - "30120:30120"
      - "30120:30120/udp"
      - "40120:40120"
    environment:
      SERVER_PROFILE: "default"
      LICENCE_KEY: "license-key-here"
      FIVEM_PORT: "30120"
      WEB_PORT: "40120"
      HOST_UID: 1000
      HOST_GID: 100
      FIVEM_HOSTNAME: "FiveMESX-Server"
      STEAM_WEBAPIKEY: "steam-api-key"
    depends_on:
      - mariadb
  mariadb:
    image: mariadb:10.4
    restart: on-failure
    volumes:
      # Remember to change.
      - /path/to/mysql:/var/lib/mysql
      # DO NOT CHANGE THIS (sql file dir)
      - sqlfile:/docker-entrypoint-initdb.d/:ro
    environment:
      MYSQL_ROOT_PASSWORD: password
  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    restart: on-failure
    ports:
      # Port to phpmyadmin webgui
      - 8100:80
    environment:
      - PMA_HOST=mariadb
    depends_on:
      - mariadb
```

When the container is running you can access txAdmin on the specified port and login with the username `administrator` and the password `adminadmin`. After login, immediately change login the password.

_It is important that you use `interactive` and `pseudo-tty` options otherwise the container will crash on startup_
See [issue #3](https://github.com/spritsail/fivem/issues/3)

### Environment Varibles

- `LICENSE_KEY` - This is a required variable for the licence key needed to start the server.
- `RCON_PASSWORD` - A password to use for the RCON functionality of the fxserver. If not specified, a random 16 character password is assigned. This is only used upon creation of the default configs
- `WEB_PORT` - Sets up txAdmin to run on this port. If not specified, will use `40120` by default.
- `FIVEM_PORT` - If the server.cfg file needs to be generated, the server will use this port. If not specified, will use `30120` by default.
- `HOST_GID` - The files that are generated by the container will be written with this group ID. You must use numeric IDs. If not specified, will use `0` (root).
- `HOST_UID` - The files that are generated by the container will be written with this user ID. You must use numeric IDs. If not specified, will use `0` (root).
- `SERVER_PROFILE` - profile name used by txAdmin. If not specified, will use `dev_server`.

## Credits

 - This image is based on the [spritsail/fivem](https://hub.docker.com/r/spritsail/fivem) image. Thanks to **Spritsail** !
 - Thanks to **tabarra** as the creator and maintainer of the [txAdmin](https://github.com/tabarra/txAdmin) repository!
 - Thanks to [Andruida][git] that I forked this code from
