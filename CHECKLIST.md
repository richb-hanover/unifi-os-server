# CHECKLIST

To get started with **Unifi OS Server**,
you must modify the _docker-compose.yaml_
file for your local environment.
Customize these variables in the file:

- `UOS_SYSTEM_IP` must be set to the address
  of the computer where you are running Unifi OS Server
- All the `volumes` must point to a good place on your
  local hard drive.
  Since no other processes are running on my host,
  I replace `/path/to/unifi-os-server` in the original file
  with `${HOME}/uos-data` for each of the lines.
- You may need to adjust the `cgroup` settings if you are
  not running on standard Linux. 
  Claude and ChatGPT are your friends.
- Check the **Convenience Techniques** below for long-term
  maintenance.
  
## Testing and Running

To test and run the server, use these commands:

```
# stop any previous Unifi Network or Unifi OS Server

cd directory-containing-docker-compose.yaml-file
docker compose up -d        # to start the server

... or...

docker compose down         # to stop the server

# browse to `your-ip-address:11433` to see Unifi OS Server.
```


## Updating to a new Unifi OS Server Version

Stop the container, then start it again,
using the procedure above.
When re-starting the container, Docker checks for `latest`
and reloads the image if necessary.

# Convenience techniques

The steps above work just fine, however,
they can lead to hassles staying sync'd with the main repo
as it evolves.
The tips below allow your installation to follow the
main repo over time.

## Using git

Although the _docker-compose.yaml_ file contains everything
required to run Unifi OS Server,
you won't automatically get updates
(however infrequent).

To get an entire copy of the Unifi OS Server repo on your
local machine, use the `git` program.
(The repo is fairly small.)
Use these commands to create the **main directory** named _unifi-os-server_ inside the _good-directory-to-hold-source-files_:

```
cd good-directory-to-hold-source-files
git clone https://github.com/lemker/unifi-os-server.git
```

To update your local copy use `git pull` like this:

```
cd the-main-directory
git pull
```

## Using an Override File

A downside of directly modifying the _docker-compose.yaml_
file is that a subsequent `git pull` overwrites it.

To address this, Docker supports an "override" facility
that merges the contents of a file named
_docker-compose.override.yaml_ with the main _yaml_ file.
This allows you to place installation-specific variables
in that separate file.

To do this, create a new _docker-compose.override.yaml_ file,
save it in the **main directory** and paste in these lines:

```
# docker-compose.override.yaml

# This file contains values that override the defaults from
# the main repo's docker-compose.yaml file.
# See the CHECKLIST.md file for details
---
services:
  unifi-os-server:
    environment:
      - UOS_SYSTEM_IP=192.168.1.1 # USE YOUR SYSTEM'S IP ADDRESS
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw   # This is already correct
      # I save these files in the top-level "uos-data" directory
      # You could choose another location
      - ${HOME}/uos-data/persistent:/persistent
      - ${HOME}/uos-data/var-log:/var/log
      - ${HOME}/uos-data/data:/data
      - ${HOME}/uos-data/srv:/srv
      - ${HOME}/uos-data/var-lib-unifi:/var/lib/unifi
      - ${HOME}/uos-data/var-lib-mongodb:/var/lib/mongodb
      - ${HOME}/uos-data/etc-rabbitmq-ssl:/etc/rabbitmq/ssl```
```

**BACK UP THIS FILE!** It is **not** saved anywhere else.
It is mentioned in the _.gitignore_ file so that it
will not be overwritten by a subsequent `git pull`.

## Testing and Running the Override File

Use the same procedure as above. `docker compose up -d` to start.  `docker compose down` to stop.
