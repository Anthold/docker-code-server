[Code-server](https://coder.com) is VS Code running on a remote server, accessible through the browser.
- Code on your Chromebook, tablet, and laptop with a consistent dev environment.
- If you have a Windows or Mac workstation, more easily develop for Linux.
- Take advantage of large cloud servers to speed up tests, compilations, downloads, and more.
- Preserve battery life when you're on the go.
- All intensive computation runs on your server.
- You're no longer running excess instances of Chrome.

## Supported Architectures

We utilise the docker manifest for multi-platform awareness. More information is available from docker [here](https://github.com/docker/distribution/blob/master/docs/spec/manifest-v2-2.md#manifest-list) 

Access the webui at `http://<your-ip>:8443`.
For github integration, drop your ssh key in to `/config/.ssh`.
Then open a terminal from the top menu and set your github username and email via the following commands

```bash
git config --global user.name "username"
git config --global user.email "email address"
```

### Hashed code-server password

How to create the [hashed password](https://github.com/cdr/code-server/blob/master/docs/FAQ.md#can-i-store-my-password-hashed).

## Usage

Here are some example snippets to help you get started creating a container.

### docker-compose (recommended, [click here for more info](https://docs.linuxserver.io/general/docker-compose))

```yaml
---
version: "2.1"
services:
  code-server:
    build:
      context: ./
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - PASSWORD=password #optional
      - HASHED_PASSWORD= #optional
      - SUDO_PASSWORD=password #optional
      - SUDO_PASSWORD_HASH= #optional
      - PROXY_DOMAIN=code-server.my.domain #optional
      - DEFAULT_WORKSPACE=/config/workspace #optional
    volumes:
      - /path/to/appdata/config:/config
    ports:
      - 8443:8443
    restart: unless-stopped
```

### docker cli ([click here for more info](https://docs.docker.com/engine/reference/commandline/cli/))

```bash
docker run -d \
  --name=code-server \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e PASSWORD=password `#optional` \
  -e HASHED_PASSWORD= `#optional` \
  -e SUDO_PASSWORD=password `#optional` \
  -e SUDO_PASSWORD_HASH= `#optional` \
  -e PROXY_DOMAIN=code-server.my.domain `#optional` \
  -e DEFAULT_WORKSPACE=/config/workspace `#optional` \
  -p 8443:8443 \
  -v /path/to/appdata/config:/config \
  --restart unless-stopped \
  .
```

## Parameters

Container images are configured using parameters passed at runtime (such as those above). These parameters are separated by a colon and indicate `<external>:<internal>` respectively. For example, `-p 8080:80` would expose port `80` from inside the container to be accessible from the host's IP on port `8080` outside the container.

| Parameter | Function |
| :----: | --- |
| `-p 8443` | web gui |
| `-e PUID=1000` | for UserID - see below for explanation |
| `-e PGID=1000` | for GroupID - see below for explanation |
| `-e TZ=Europe/London` | Specify a timezone to use EG Europe/London |
| `-e PASSWORD=password` | Optional web gui password, if `PASSWORD` or `HASHED_PASSWORD` is not provided, there will be no auth. |
| `-e HASHED_PASSWORD=` | Optional web gui password, overrides `PASSWORD`, instructions on how to create it is below. |
| `-e SUDO_PASSWORD=password` | If this optional variable is set, user will have sudo access in the code-server terminal with the specified password. |
| `-e SUDO_PASSWORD_HASH=` | Optionally set sudo password via hash (takes priority over `SUDO_PASSWORD` var). Format is `$type$salt$hashed`. |
| `-e PROXY_DOMAIN=code-server.my.domain` | If this optional variable is set, this domain will be proxied for subdomain proxying. See [Documentation](https://github.com/cdr/code-server/blob/master/docs/FAQ.md#sub-domains) |
| `-e DEFAULT_WORKSPACE=/config/workspace` | If this optional variable is set, code-server will open this directory by default |
| `-v /config` | Contains all relevant configuration files. |

## Environment variables from files (Docker secrets)

You can set any environment variable from a file by using a special prepend `FILE__`.

As an example:

```bash
-e FILE__PASSWORD=/run/secrets/mysecretpassword
```

Will set the environment variable `PASSWORD` based on the contents of the `/run/secrets/mysecretpassword` file.

## Umask for running applications

For all of our images we provide the ability to override the default umask settings for services started within the containers using the optional `-e UMASK=022` setting.
Keep in mind umask is not chmod it subtracts from permissions based on it's value it does not add. Please read up [here](https://en.wikipedia.org/wiki/Umask) before asking for support.

## User / Group Identifiers

When using volumes (`-v` flags) permissions issues can arise between the host OS and the container, we avoid this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id user` as below:

```bash
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

## Docker Mods

[![Docker Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=code-server&query=%24.mods%5B%27code-server%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=code-server "view available mods for this container.") [![Docker Universal Mods](https://img.shields.io/badge/dynamic/yaml?color=94398d&labelColor=555555&logoColor=ffffff&style=for-the-badge&label=universal&query=%24.mods%5B%27universal%27%5D.mod_count&url=https%3A%2F%2Fraw.githubusercontent.com%2Flinuxserver%2Fdocker-mods%2Fmaster%2Fmod-list.yml)](https://mods.linuxserver.io/?mod=universal "view available universal mods.")

We publish various [Docker Mods](https://github.com/linuxserver/docker-mods) to enable additional functionality within the containers. The list of Mods available for this image (if any) as well as universal mods that can be applied to any one of our images can be accessed via the dynamic badges above.

## Support Info

* Shell access whilst the container is running: `docker exec -it code-server /bin/bash`
* To monitor the logs of the container in realtime: `docker logs -f code-server`
* container version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' code-server`
* image version number
  * `docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/code-server:latest`

## Updating Info

Most of our images are static, versioned, and require an image update and container recreation to update the app inside. With some exceptions (ie. nextcloud, plex), we do not recommend or support updating apps inside the container. Please consult the [Application Setup](#application-setup) section above to see if it is recommended for the image.

Below are the instructions for updating containers:

### Via Docker Compose

* Update all images: `docker-compose pull`
  * or update a single image: `docker-compose pull code-server`
* Let compose update all containers as necessary: `docker-compose up -d`
  * or update a single container: `docker-compose up -d code-server`
* You can also remove the old dangling images: `docker image prune`

### Via Docker Run

* Update the image: `docker pull lscr.io/linuxserver/code-server:latest`
* Stop the running container: `docker stop code-server`
* Delete the container: `docker rm code-server`
* Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your `/config` folder and settings will be preserved)
* You can also remove the old dangling images: `docker image prune`

### Via Watchtower auto-updater (only use if you don't remember the original parameters)

* Pull the latest image at its tag and replace it with the same env variables in one run:

  ```bash
  docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower \
  --run-once code-server
  ```

* You can also remove the old dangling images: `docker image prune`

**Note:** We do not endorse the use of Watchtower as a solution to automated updates of existing Docker containers. In fact we generally discourage automated updates. However, this is a useful tool for one-time manual updates of containers where you have forgotten the original parameters. In the long term, we highly recommend using [Docker Compose](https://docs.linuxserver.io/general/docker-compose).

### Image Update Notifications - Diun (Docker Image Update Notifier)

* We recommend [Diun](https://crazymax.dev/diun/) for update notifications. Other tools that automatically update containers unattended are not recommended or supported.