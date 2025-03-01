# kickstart

See [https://nfra.infracamp.org](https://nfra.infracamp.org) for detailed documentation.

## TL;DR

On your local workstation, `kickstart.sh` will:
- Start up the container setting env `DEV_MODE=1` and giving you an **interactive shell** as user `user` inside the container.
- Mount the **project directory** to `/opt` inside the container so every user has the same absolute path.
- Expose **ports 80,4000,4100,4200** on localhost so you can access the service with any browser at `http://localhost`
  (configured by project in `.kickstartconfig` or global in `$HOME/.kickstartconfig`)
- **Detect operating system** and container service (OSx, Linux, WSL2)
- Set the **uid** of user `user` inside the container according to your actual uid so there will no permission problems
- Check for other running instances of the project (Choose between: Kill, Shell or Ignore)
- Securely mount your **ssh-key** into the container, so you can use git within your container
- Mount the **bash history** into the container
- Mount **cache directories** for **apt, npm, composer, pip** into the container
- Evaluate global `$HOME/.kickcstartonfig` file for additional mounts/ports/settings
- Securely provide developer's **secrets** from `$HOME/.kickstart/secrets/<project>/<secret_name>`to the container
- Set **environment variables** according to `.env`-file
- Detect and provide the **hosts's IP address** to the container (for running debuggers, etc) as env `DOCKER_HOST_IP`
- Start **additional services** from `.kick-stack.yml` in composer format
- Setup interactive shell (colors, screen-size, adjustments for osX, non-interactive shells)
- **Run commands** defined in `.kick.yml`-file in the project folder (if using kickstart-flavor-containers)
- Inform you about **updates** of `kickstart.sh` and provide auto-download updates by calling `./kickstart.sh --upgrade`
- Provide access to **skeleton projects** that can be defined in a central git repository

On testing stage `kickstart.sh` will:
- Execute the tests the same way they will be executed in CI/CD environment. So you can debug 
  on localhost instead of pushing over any over again.

On CI/CD pipeline `kickstart.sh` will:
- Ensure no ssh-keys or secrets are copied inside the container.
- **Auto-detect** `gitlab-ci`, `github-actions`, `jenkins` build environment and determine `TAG` and `BRANCH`
- Set permissions according to the build environment
- **Build the container** running `docker build` and tagging with the correct tags
- Logging into the **registry** accoring to the build environment
- **Pushing** to a registry defined inside the build environment

On Deploy-stage:
- Autodetect docker-swarm, kubernetes by environment inside build environment
- HTTP-PUSH to hooks urls

A bash script to start and manage your develompment containers.

## Quick Start

Change into your project directory.

Run the container defined in `.kick.yml`:
```
kickstart
```

Run a command defined in `.kick.yml`-`command:` section:
```
kickstart :[command]
```

List available skeletons:
```
kickstart skel list
```

Install skeleton:
```
kickstart skel install <name>
```

Upgrade to newest kickstart version:
```
kickstart upgrade
```

Run a ci-build (build and push using gitlab-ci-runner):
```
kickstart ci-build
```





## Documents index

- [InfraCamp Homepage](http://infracamp.org)
- [**Setting up your environment**](doc/setup/installing-ubuntu-debian-mac.md)
- [Bash Scripting 101](doc/bash_scripting101.md)
- [.kick.yml reference](doc/kick.yml.md)

## Project setup: Kickstart

**Copy'n'Paste installer script**: (execute as user in your project-directory)
```bash
sudo curl -o /usr/bin/kickstart "https://raw.githubusercontent.com/nfra-project/nfra-kickstart/master/dist/kickstart.sh" && sudo chmod +x /usr/bin/kickstart
```

The script will save [kickstart.sh](https://raw.githubusercontent.com/infracamp/kickstart/master/dist/kickstart.sh) to 
systems /usr/bin directory and set the executalbe bit.

**Run kickstart:**
```bash
kickstart
```

Kickstart will create an empty `.kick.yml` file in the current directory. You might want to edit
at least the `from:`-Line.


## .kick.yml - Kickstart configuration file. *([Full Docs](doc/kick.yml.md))*

```yaml
version: 1
from: "nfra/kickstart-flavor-bare:1.0"
ports: "80:80;4000:4000"
secrets: "some_secret keystore.yml"
stack:
    ..more options..
```

Run `kickstart` - the container should start.

To select a special flavor select

```yaml
version: 1
from: "nfra/kickstart-flavor-bare:1.0"
```



## Available Flavors

See [infracamp.org/container/](https://infracamp.org/container/) for
full list and links to their documentation.

## Writing config-files

`.kick.yml:`
```yaml
config_file:
  template: "config.php.dist"
  target: "config.php"
```

Will read `config.dist.php` file, which will be parsed copied into config.php.

`config.php.dist`
```php
<?php
define("CONF_MYSQL_HOST", "%CONF_MYSQL_HOST%");
define("VERSION_STRING", "%VERSION_STRING%");
```

The configuration will be loaded from environment variables.


## Development and Deploy Tool: `kick`

You can define commands and run it inside the container.

```
version: 1
from: "infracamp/kickstart-flavor-gaia"

command:
    build:
        - "echo 'Build called'"
    run:
        - "echo 'Run called'"
        
    do_something:
        - "echo 'doing something'"
```

- Will work from any directory
- All paths relative to .kickstart.yml
- Run commands: `kick do_something`


## Starting a stack of helper services

Kickstart will search for a file `.kick-stack.yml` in the project main
directory. If this file exists, it will be deployed as docker stack.

**Make sure, all services you want to access from within your container
are attached to the external network `project_name`**

Assume our project_name is `my_proj_1` and we want to provide a mysql service
```yaml
version: "3"
services:
  mysqld:
    image: mysql
    networks:
      - my_proj_1


networks:
  my_proj_1:
    external: true
```

The mysql service will be availabe as `my_proj_1_mysqld`.


## System-wide config file

Kickstart will read the user-config from:
```
~/.kickstartconfig
```

Available Options:

```
KICKSTART_DOCKER_RUN_OPTS=""        # Optional parameters passed to the docker run command
KICKSTART_PORTS="80:4200;25:25"     # Change the Port-Mappings
KICKSTART_WIN_PATH=                 # If running on windows - map bash 
```

## Secrets

Secrets can be added either via the command `kickstart secrets add <secretname>` or
via Environment variables (used for ci-builds). All variables names `KICKSECRET_name` will
be mounted to `/run/secrets/name`.


## Project-wide config file

```
./kickstartconfig
```


## Defaults

### Networking

By default, kickstart will configure debuggers to send data to `10.10.10.10`. So 
this ip should be added to your pc's networks.


### Add links to other containers

Start one or more containers. If you are not using kickstart, make sure
you specify a name with the parameter `--name`.

Create, if not already exisitng a project-wide `.kickstartconfig` file.
Add a Line:

```
KICKSTART_DOCKER_RUN_OPTS="--link otherContainerName"
```



## Building containers

You can build ready-to-deploy containers with kickstart. Just add a `Dockerfile`
to your Project-Folder

```dockerfile
FROM nfra/kickstart-flavor-php:8.0

ENV DEV_CONTAINER_NAME="some_name"
ENV HTTP_PORT=80

ADD / /opt
RUN ["bash", "-c",  "chown -R user /opt"]
RUN ["/kickstart/container/start.sh", "build"]

ENTRYPOINT ["/kickstart/container/start.sh", "standalone"]
```

Interval: A `kick interval` will be triggered every second (synchronous).

To save cpu-time you could add this to your `.kick.yml`
```yaml
command:
    interval:
      - "sleep 300"

```

### Building tagged containers with gitlab-ci

Add a tag to the master branch and add the gitlab-ci config

```yaml
latest:
  stage: build
  script:
    - ./kickstart.sh ci-build
  only:
    - master
    - /^v.*$/
```

This will create a `latest` tag on every push and a latest and `vx.x.x` tagged
docker image on tagged builds.


## Version files

Kickstart maintains a `VERSION` file in the projects root directory. To prevent
merge issues, the file is filled with fixed mock data during development and only
written if the parameter `--write-version` is present. (This is always the case
in ci-build mode).

The `VERSION` file is made for fast reading with static lines:

```
# This is an autogenerated Version file (from kickstart): 
4
Fri, 19 Feb 2021 12:33:13 +0100
Sun, 14 Feb 2021 13:08:41 +0100
315bc50
developer name
```

- Index0 (Line 1): Notice about the format
- Index1 (Line 2): The commit number (count of all commits)
- Index2 (Line 3): The Build date (current date)
- Index3 (Line 4): The Commit Date
- Index4 (Line 5): The Commit ID
- Index5 (Line 6): The Commit author's name


## Building own flavors

Feel free to build your own flavors.

Some rules:

- Each flavor should reside in an separate repository
- It must build the tags `latest` (stable release) and `testing` (current master branch build)
- It must provide tests
- And should provide easy to use documentation
- It should build using hub.docker.com public build service (free of charge!)

Flavor names derive from greek mystical names [click](https://de.wikipedia.org/wiki/Griechische_Mythologie)
