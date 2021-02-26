# Reference `.kick.yml`


## The Reference yaml

```yaml
version: 1
from: "infracamp/kickstart-falvor-gaia"

# Write environment into config files
config_file:
    template: config.php.dist
    target: config.php

# Install packages via apt-get
packages: [package1, package2]

# Set Environment Variables
env:
  - SOME_ENV=Some value 
  - PATH="/some/path:$PATH"
  - "SOME_ENV=${SOME_ENV:-defaultValue}"      # Define default environment

# Execute commands
command:
    command_name1:
      - "script to exec (as user)"
        
    command_shell: |
      if [ -f /some/file ]; then
         echo "file found"
      fi;
    build:
        - "sudo cp /opt/xyz /etc/opt/xyz"
        - "echo 'This is executed on build time'"
        
    run:
        - "echo 'This is executed on run time'"
        
    interval:
        - "echo 'This will be executed in the standalone server every 60 seconds'"    
    
```


## Discussion

### The `from:` section

It specifies, which container kickstart.sh will download and execute.
You can specify any container of official [dockerhub](http://hub.docker.com)
repositories.

For best performance you'll specify a `infracamp/kickstart-flavor-*`
container. See [infracamp.org/containers](https://infracamp.org/containers/)

### Writing config-files `config_file`

The container will call the action `kick write_config_file` on startup.
If a section `config_file` is defined, it will replace `%ENV_NAME%` by
the value of the environment variable.

Best practice is, to put the `target`-file (here: `config.php`) to
`.gitignore`. 

Examples:

```php
# config.php.dist:
<?php

define("CONF_MYSQL_HOST", "%CONF_MYSQL_HOST%");
```

### Install packages via [apt-get](https://wiki.ubuntuusers.de/apt/apt-get/)

during the start of the container, all packages specified here are searched for and, if available, also installed.

##### Example:
```yaml
packages: [php8.0-curl, php8.0-http, php8.0-raphf, php8.0-xml]
```

the example shows all packages that needs to be installed for [phpunit](https://phpunit.de/) as an example.

### Defining commands: The `commands:`-section

All commands can be run ***inside*** the container by running

```
kick [command-name]
```

Or from ***outside*** the container by running 

```
./kickstart.sh [command-name]
```

#### Predefined commands

##### `build`-command

This command is run before anything is configured inside the container.
(No enviromnent, etc). If you want to install additional software,
this is the right place to do it

##### `run`-command

Is executed right before the container services (e.g. apache) starts.
This is the right time to look at the environment and configure the
container.

#### `interval`-command

When running in standalone mode, this command will be executed all **60 seconds**

In development mode, you can run it yourself by running `kick interval`.
(In development mode you don't want any background-tasks to run automaticly.

***There is no lock mechanism: You have to take care of the processes your
own***

 
 




