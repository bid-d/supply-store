# Table of contents
- [Table of contents](#table-of-contents)
  - [Usage](#usage)
  - [Prerequisites](#prerequisites)
  - [Version Tested](#version-tested)
  - [Setup](#setup)
  - [Debug](#debug)
  - [Custom CLI Commands](#custom-cli-commands)
## Usage

This configuration is used for setting up a full magento 2 local development.  

## Prerequisites
-  Operating systems (Linux x86-64): such as RedHat Enterprise Linux (RHEL), CentOS, Ubuntu, Debian, and similar
-  Memory requirement: At least 2GB Ram
-  Hard Drive: SSD is preferred over HDD
-  [Docker installation (For Ubuntu)](https://docs.docker.com/engine/install/ubuntu/)
-  [Post-Installation for Docker (Avoid running Docker as Root)](https://docs.docker.com/engine/install/linux-postinstall/)
-  [Docker Compose installation](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04)
-  Composer pre-installed is not required
-  Turn off any services running on port 80, 3306, and 9200 (or 9300)

## Version Tested
- PHP: 7.4
- Magento: 2.4.2

## Setup
- In your project directory, run following commands to install:

```bash
# Create an empty directory
mkdir project-name && cd project-name

# Download the Docker Compose template:
curl -s https://raw.githubusercontent.com/bid-d/docker-magento/master/lib/template | bash

# Replace with existing source code of your existing Magento instance:
git clone --branch develop git@bitbucket.org:convert/supply-store.git src

# Create a DNS host entry for the site:
# You can modify `local.marketplace2.jajuma.de` to any domain you like:

echo "127.0.0.1 ::1 local.convert" | sudo tee -a /etc/hosts

# Start some containers, copy files to them and then restart the containers:
docker-compose -f docker-compose.yml up -d

# Run composer install
# In order to run this command, we need to put the auth.json into ~/.composer folder
# In the installation process, it will ask for hidden token, please be aware
bin/composer install

# Fix Owner
bin/fixowns

# Fix Permission
bin/fixperms

# Add env.php and config.php files to project
# You need to make sure that DB credentials between env/db.env and src/app/etc/env.php files match each other
mv env.example.php src/app/etc/env.php

# Download db to current directory and import it:
bin/mysql < supply-store.sql

# Import app-specific environment settings:
bin/magento app:config:import

# Set base URLs to local environment URL (default is local.marketplace2.jajuma.de)
bin/magento config:set web/secure/base_url https://local.convert/

bin/magento config:set web/unsecure/base_url https://local.convert/

bin/magento config:set web/secure/base_link_url https://local.convert/

# Setup elasticsearch
bin/magento config:set catalog/search/engine 'elasticsearch7'

bin/magento config:set catalog/search/elasticsearch7_server_hostname 'elasticsearch'

bin/magento config:set catalog/search/elasticsearch7_server_port '9200'
	
bin/restart

# Your site is ready at https://local.convert/
```

## Debug
This setup already included Xdebug installation.
In order to use Xdebug, you need to install Xdebug extension through VSCode Extensions then enable it by running: `bin/xdebug enable`

## Custom CLI Commands

- `bin/bash`: Drop into the bash prompt of your Docker container. The `phpfpm` container should be mainly used to access the filesystem within Docker.
- `bin/composer`: Run the composer binary. Ex. `bin/composer install`
- `bin/fixowns`: This will fix filesystem ownerships within the container.
- `bin/fixperms`: This will fix filesystem permissions within the container.
- `bin/grunt`: Run the grunt binary. Ex. `bin/grunt exec`
- `bin/magento`: Run the Magento CLI. Ex: `bin/magento cache:flush`
- `bin/mysql`: Run the MySQL CLI with database config from `env/db.env`. Ex. `bin/mysql -e "EXPLAIN core_config_data"` or`bin/mysql < backups/magento.sql`
- `bin/npm`: Run the npm binary. Ex. `bin/npm install`
- `bin/remove`: Remove all containers.
- `bin/removeall`: Remove all containers, networks, volumes, and images.
- `bin/removevolumes`: Remove all volumes.
- `bin/restart`: Stop and then start all containers.
- `bin/start`: Start all containers, good practice to use this instead of `docker-compose up -d`, as it may contain additional helpers.
- `bin/status`: Check the container status.
- `bin/stop`: Stop all containers.
- `bin/xdebug`: Disable or enable Xdebug. Accepts params `disable` (default) or `enable`. Ex. `bin/xdebug enable`