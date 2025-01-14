<div align="center">
  <p>Based on Mark Shust's Docker Configuration for Magento</p>
  <img src="https://img.shields.io/badge/magento-2.X-brightgreen.svg?logo=magento&longCache=true" alt="Supported Magento Versions" />
  <a href="https://hub.docker.com/r/markoshust/magento-php/" target="_blank"><img src="https://img.shields.io/docker/pulls/markoshust/magento-php.svg?label=php%20docker%20pulls" alt="Docker Hub Pulls - PHP" /></a>
  <a href="https://hub.docker.com/r/markoshust/magento-nginx/" target="_blank"><img src="https://img.shields.io/docker/pulls/markoshust/magento-nginx.svg?label=nginx%20docker%20pulls" alt="Docker Hub Pulls - Nginx" /></a>
  <a href="https://github.com/markshust/docker-magento/graphs/commit-activity" target="_blank"><img src="https://img.shields.io/badge/maintained%3F-yes-brightgreen.svg" alt="Maintained - Yes" /></a>
  <img src="https://img.shields.io/badge/apple%20silicon%20support-yes-brightgreen" alt="Apple Silicon Support" />
  <a href="https://opensource.org/licenses/MIT" target="_blank"><img src="https://img.shields.io/badge/license-MIT-blue.svg" /></a>
</div>

## Table of contents

- [Usage](#usage)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Updates](#updates)
- [Custom CLI Commands](#custom-cli-commands)
- [Misc Info](#misc-info)
- [Credits](#credits)
- [License](#license)

## Usage

This configuration is intended to be used as a Docker-based development environment for Magento 2 to work with versions under 2.4.6 that still use elasticsearch, if you're working with 2.4.6+ I suggest to use the official [docker-magento](https://github.com/markshust/docker-magento).

For a better compability, specially for use with WSL, I suggest the use of Ubuntu 18.04 LTS, for later versions you can face a couple of problems using original Mark Shust's for existing projects for Magento < 2.4.6


Folders:

- `images`: Docker images for nginx and php
- `compose`: sample setups with Docker Compose

## Prerequisites

This setup assumes you are running Docker on a computer with at least 6GB of RAM allocated to Docker, a dual-core, and an SSD hard drive. [Download & Install Docker Desktop](https://www.docker.com/products/docker-desktop).

This configuration has been tested on Linux. Windows is supported through the use of Docker on WSL2.

## Setup

#### Existing Projects

```bash
# Take a backup of your existing database
# For Adobe Cloud use Magento Cloud CLI to get your DB:
magento-cloud db:dump

# Create your project directory then go into it:
mkdir -p ~/Sites/magento
cd $_

# Download the Docker Compose template:
curl -s https://raw.githubusercontent.com/phillipesouza/docker-magento-for-work/master/lib/template | bash

# Pull your project source code of your Magento instance:
git clone git@github.com:myrepo.git src

# Start some containers, copy files to them and then restart the containers:
bin/start --no-dev

#Create auth.json at src with your active Adobe Cloud Credentials: 
[Get you Adobe Commerce credentials here](https://experienceleague.adobe.com/docs/commerce-operations/installation-guide/prerequisites/authentication-keys.html?lang=en).
## if you can't find your cloud credentials there, ask your manager to give you access

bin/copytocontainer --all ## Initial copy will take a few minutes...
# This will copy all content from src to containers

# If your vendor directory was empty, populate it with:
bin/composer install

# Import your project's database:
bin/mysql < ../existing/magento.sql

# If needed date database connection details to use the above Docker MySQL credentials:
# Also note: creds for the MySQL server are defined at startup from env/db.env
# vi src/app/etc/env.php

# Import app-specific environment settings:
bin/magento app:config:import

# Create a DNS host entry and setup Magento base url
bin/setup-domain yoursite.test
## For those using WSL add this to your hosts file too: C:\Windows\System32\drivers\etc
### it should look like this:           127.0.0.1       magento.test

bin/restart --no-dev

open https://magento.test
```
## Custom CLI Commands

- `bin/analyse`: Run `phpstan analyse` within the container to statically analyse code, passing in directory to analyse. Ex. `bin/analyse app/code`
- `bin/bash`: Drop into the bash prompt of your Docker container. The `phpfpm` container should be mainly used to access the filesystem within Docker.
- `bin/cache-clean`: Access the [cache-clean](https://github.com/mage2tv/magento-cache-clean) CLI. Note the watcher is automatically started at startup in `bin/start`. Ex. `bin/cache-clean config full_page`
- `bin/cli`: Run any CLI command without going into the bash prompt. Ex. `bin/cli ls`
- `bin/clinotty`: Run any CLI command with no TTY. Ex. `bin/clinotty chmod u+x bin/magento`
- `bin/cliq`: The same as `bin/cli`, but pipes all output to `/dev/null`. Useful for a quiet CLI, or implementing long-running processes.
- `bin/composer`: Run the composer binary. Ex. `bin/composer install`
- `bin/copyfromcontainer`: Copy folders or files from container to host. Ex. `bin/copyfromcontainer vendor`
- `bin/copytocontainer`: Copy folders or files from host to container. Ex. `bin/copytocontainer --all`
- `bin/cron`: Start or stop the cron service. Ex. `bin/cron start`
- `bin/dev-urn-catalog-generate`: Generate URN's for PhpStorm and remap paths to local host. Restart PhpStorm after running this command.
- `bin/devconsole`: Alias for `bin/n98-magerun2 dev:console`
- `bin/docker-compose`: Support V1 (`docker-compose`) and V2 (`docker compose`) docker compose command, and use custom configuration files, such as `compose.yml` and `compose.dev.yml`
- `bin/download`: Download specific Magento version from Composer to the container, with optional arguments of the version (2.4.6 [default]) and type ("community" [default], "enterprise", or "mageos"). Ex. `bin/download 2.4.6 enterprise`
- `bin/debug-cli`: Enable Xdebug for bin/magento, with an optional argument of the IDE key. Defaults to PHPSTORM Ex. `bin/debug-cli enable PHPSTORM`
- `bin/fixowns`: This will fix filesystem ownerships within the container.
- `bin/fixperms`: This will fix filesystem permissions within the container.
- `bin/grunt`: Run the grunt binary. Ex. `bin/grunt exec`
- `bin/install-php-extensions`: Install PHP extension in the container. Ex. `bin/install-php-extensions sourceguardian`
- `bin/magento`: Run the Magento CLI. Ex: `bin/magento cache:flush`
- `bin/mftf`: Run the Magento MFTF. Ex: `bin/mftf build:project`
- `bin/mysql`: Run the MySQL CLI with database config from `env/db.env`. Ex. `bin/mysql -e "EXPLAIN core_config_data"` or`bin/mysql < magento.sql`
- `bin/mysqldump`: Backup the Magento database. Ex. `bin/mysqldump > magento.sql`
- `bin/n98-magerun2`: Access the [n98-magerun2](https://github.com/netz98/n98-magerun2) CLI. Ex: `bin/n98-magerun2 dev:console`
- `bin/node`: Run the node binary. Ex. `bin/node --version`
- `bin/npm`: Run the npm binary. Ex. `bin/npm install`
- `bin/phpcbf`: Auto-fix PHP_CodeSniffer errors with Magento2 options. Ex. `bin/phpcbf <path-to-extension>`
- `bin/phpcs`: Run PHP_CodeSniffer with Magento2 options. Ex. `bin/phpcs <path-to-extension>`
- `bin/phpcs-json-report`: Run PHP_CodeSniffer with Magento2 options and save to `report.json` file. Ex. `bin/phpcs-json-report <path-to-extension>`
- `bin/pwa-studio`: (BETA) Start the PWA Studio server. Note that Chrome will throw SSL cert errors and not allow you to view the site, but Firefox will.
- `bin/redis`: Run a command from the redis container. Ex. `bin/redis redis-cli monitor`
- `bin/remove`: Remove all containers.
- `bin/removeall`: Remove all containers, networks, volumes, and images, calling `bin/stopall` before doing so.
- `bin/removevolumes`: Remove all volumes.
- `bin/restart`: Stop and then start all containers.
- `bin/root`: Run any CLI command as root without going into the bash prompt. Ex `bin/root apt-get install nano`
- `bin/rootnotty`: Run any CLI command as root with no TTY. Ex `bin/rootnotty chown -R app:app /var/www/html`
- `bin/setup`: Run the Magento setup process to install Magento from the source code, with optional domain name. Defaults to `magento.test`. Ex. `bin/setup magento.test`
- `bin/setup-composer-auth`: Setup authentication credentials for Composer.
- `bin/setup-domain`: Setup Magento domain name. Ex: `bin/setup-domain magento.test`
- `bin/setup-grunt`: Install and configure Grunt JavaScript task runner to compile .less files
- `bin/setup-pwa-studio`: (BETA) Install PWA Studio (requires NodeJS and Yarn to be installed on the host machine). Pass in your base site domain, otherwise the default `master-7rqtwti-mfwmkrjfqvbjk.us-4.magentosite.cloud` will be used. Ex: `bin/setup-pwa-studio magento.test`
- `bin/setup-ssl`: Generate an SSL certificate for one or more domains. Ex. `bin/setup-ssl magento.test foo.test`
- `bin/setup-ssl-ca`: Generate a certificate authority and copy it to the host.
- `bin/start`: Start all containers, good practice to use this instead of `docker-compose up -d`, as it may contain additional helpers.
- `bin/status`: Check the container status.
- `bin/stop`: Stop all project containers.
- `bin/stopall`: Stop all docker running containers
- `bin/update`: Update your project to the most recent version of `docker-magento`.
- `bin/xdebug`: Disable or enable Xdebug. Accepts params `disable` (default) or `enable`. Ex. `bin/xdebug enable`

## Misc Info

### Install fails because project directory is not empty

The most common issue with a failed docker-magento install is getting this error:

```
Project directory "/var/www/html/." is not empty error
```

This message occurs when _something_ fails to execute correctly during an install, and a subsequent install is re-attempted. Unfortunately, when attempting a second (or third) install, it's possible the `src` directory is no longer empty. This prevents Composer from creating the new project because it needs to create new projects within an empty directory.

The workaround to this is that once you have fixed the issue that was initially preventing your install from completing, you will need to completely remove the assets from the previously attempted install before attempting a subsequent install.

You can do this by running:

```
bin/removeall
cd ..
rm -rf yourproject
```

Then, create your new project directory again so you can attempt the install process again. The `bin/removeall` command removes all previous Docker containers & volumes related to the specific project directory you are within. You can then attempt the install process again.

### Caching

For an improved developer experience, caches are automatically refreshed when related files are updated, courtesy of [cache-clean](https://github.com/mage2tv/magento-cache-clean). This means you can keep all of the standard Magento caches enabled, and this script will only clear the specific caches needed, and only when necessary.

To disable this functionality, uncomment the last line in the `bin/start` file to disable the watcher.

### Database

The hostname of each service is the name of the service within the `compose.yaml` file. So for example, MySQL's hostname is `db` (not `localhost`) when accessing it from within a Docker container. Elasticsearch's hostname is `elasticsearch`.

To connect to the MySQL CLI tool of the Docker instance, run:

```
bin/mysql
```

You can use the `bin/mysql` script to import a database, for example a file stored in your local host directory at `magento.sql`:

```
bin/mysql < magento.sql
```

You also can use `bin/mysqldump` to export the database. The file will appear in your local host directory at `magento.sql`:

```
bin/mysqldump > magento.sql
```

> Getting an "Access denied, you need (at least one of) the SUPER privilege(s) for this operation." message when running one of the above lines? Try running it as root with:
> ```
> bin/clinotty mysql -hdb -uroot -pmagento magento < src/backup.sql
> ```
> You can also remove the DEFINER lines from the MySQL backup file with:
> ```
> sed 's/\sDEFINER=`[^`]*`@`[^`]*`//g' -i src/backup.sql
> ```

### Composer Authentication

First setup Magento Marketplace authentication (details in the [DevDocs](http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html)).

Copy `src/auth.json.sample` to `src/auth.json`. Then, update the username and password values with your Magento public and private keys, respectively. Finally, copy the file to the container by running `bin/copytocontainer auth.json`.

### Email / Mailcatcher

View emails sent locally through Mailcatcher by visiting [http://{yourdomain}:1080](http://{yourdomain}:1080). During development, it's easiest to test emails using a third-party module such as [Mageplaza's SMTP module](https://github.com/mageplaza/magento-2-smtp). In order to use mailcatcher, set the mailserver host to `mailcatcher` and set port to `1025`. Note that this port is different from the mailcatcher interface to read the emails.

### Redis

Redis is now the default cache and session storage engine, and is automatically configured & enabled when running `bin/setup` on new installs.

Use the following lines to enable Redis on existing installs:

**Enable for Cache:**

`bin/magento setup:config:set --cache-backend=redis --cache-backend-redis-server=redis --cache-backend-redis-db=0`

**Enable for Full Page Cache:**

`bin/magento setup:config:set --page-cache=redis --page-cache-redis-server=redis --page-cache-redis-db=1`

**Enable for Session:**

`bin/magento setup:config:set --session-save=redis --session-save-redis-host=redis --session-save-redis-log-level=4 --session-save-redis-db=2`

You may also monitor Redis by running: `bin/redis redis-cli monitor`

For more information about Redis usage with Magento, <a href="https://devdocs.magento.com/guides/v2.4/config-guide/redis/redis-session.html" target="_blank">see the DevDocs</a>.

### PhpMyAdmin

PhpMyAdmin is built into the `compose.dev.yaml` file. Simply open `http://localhost:8080` in a web browser.

### Xdebug & VS Code

Install and enable the PHP Debug extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug).

Otherwise, this project now automatically sets up Xdebug support with VS Code. If you wish to set this up manually, please see the [`.vscode/launch.json`](https://github.com/markshust/docker-magento/blame/master/compose/.vscode/launch.json) file.

### Xdebug & VS Code in a WSL2 environment

Install and enable the PHP Debug extension from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug).

Otherwise, this project now automatically sets up Xdebug support with VS Code. If you wish to set this up manually, please see the [`.vscode/launch.json`](https://github.com/markshust/docker-magento/blame/master/compose/.vscode/launch.json) file.

1. In VS Code, make sure that it's running in a WSL window, rather than in the default window.
2. Install the [`PHP Debug`](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug) extension on VS Code.
3. Create a new configuration file inside the project. Go to the `Run and Debug` section in VS Code, then click on `create a launch.json file`.
4. Attention to the following configs inside the file:
    * The port must be the same as the port on the xdebug.ini file.
    ```bash
      bin/cli cat /usr/local/etc/php/php.ini
    ```
    ```bash
      memory_limit = 4G
      max_execution_time = 1800
      zlib.output_compression = On
      cgi.fix_pathinfo = 0
      date.timezone = UTC

      xdebug.mode = debug
      xdebug.client_host = host.docker.internal
      xdebug.idekey = PHPSTORM
      xdebug.client_port=9003
      #You can uncomment the following line to force the debug with each request
      #xdebug.start_with_request=yes

      upload_max_filesize = 100M
      post_max_size = 100M
      max_input_vars = 10000
    ```
    * The pathMappings should have the same folder path as the project inside the Docker container.
    ```json
      {
          "version": "0.2.0",
          "configurations": [
              {
                  "name": "Listen for XDebug",
                  "type": "php",
                  "request": "launch",
                  "port": 9003,
                  "pathMappings": {
                      "/var/www/html": "${workspaceFolder}"
                  },
                  "hostname": "localhost"
              }
          ]
      }
    ```
5. Run the following command in the Windows Powershell. It allows WSL through the firewall, otherwise breakpoints might not be hitten.
    ```powershell
    New-NetFirewallRule -DisplayName "WSL" -Direction Inbound  -InterfaceAlias "vEthernet (WSL)"  -Action Allow
    ```

### Xdebug & PhpStorm

1.  First, install the [Chrome Xdebug helper](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc). After installed, right click on the Chrome icon for it and go to Options. Under IDE Key, select PhpStorm from the list to set the IDE Key to "PHPSTORM", then click Save.

2.  Next, enable Xdebug debugging in the PHP container by running: `bin/xdebug enable`.

3.  Then, open `PhpStorm > Preferences > PHP` and configure:

    * `CLI Interpreter`
        * Create a new interpreter from the `From Docker, Vagrant, VM...` list.
        * Select the Docker Compose option.
        * For Server, select `Docker`. If you don't have Docker set up as a server, create one and name it `Docker`.
        * For Configuration files, add both the `compose.yaml` and `compose.dev.yaml` files from your project directory.
        * For Service, select `phpfpm`, then click OK.
        * Name this CLI Interpreter `phpfpm`, then click OK again.

    * `Path mappings`
        * There is no need to define a path mapping in this area.

4. Open `PhpStorm > Preferences > PHP > Debug` and ensure Debug Port is set to `9000,9003`.

5. Open `PhpStorm > Preferences > PHP > Servers` and create a new server:

    * For the Name, set this to the value of your domain name (ex. `magento.test`).
    * For the Host, set this to the value of your domain name (ex. `magento.test`).
    * Keep port set to `80`.
    * Check the "Use path mappings" box and map `src` to the absolute path of `/var/www/html`.

6. Go to `Run > Edit Configurations` and create a new `PHP Remote Debug` configuration.

    * Set the Name to the name of your domain (ex. `magento.test`).
    * Check the `Filter debug connection by IDE key` checkbox, select the Server you just setup.
    * For IDE key, enter `PHPSTORM`. This value should match the IDE Key value set by the Chrome Xdebug Helper.
    * Click OK to finish setting up the remote debugger in PHPStorm.

7. Open up `pub/index.php` and set a breakpoint near the end of the file.

    * Start the debugger with `Run > Debug 'magento.test'`, then open up a web browser.
    * Ensure the Chrome Xdebug helper is enabled by clicking on it and selecting Debug. The icon should turn bright green.
    * Navigate to your Magento store URL, and Xdebug should now trigger the debugger within PhpStorm at the toggled breakpoint.

### SSH

Since version `40.0.0`, this project supports connecting to Docker with SSH/SFTP. This means that if you solely use either PhpStorm or VSCode, you no longer need to selectively mount host volumes in order to gain bi-directional sync capabilities from host to container. This will enable full speed in the native filesystem, as all files will be stored directly in the `appdata` container volume, rather than being synced from the host. This is especially useful if you'd like to sync larger directories such as `generated`, `pub` & `vendor`.

Copy `compose.dev-ssh.yaml` to `compose.dev.yaml` before installing Magento to take advantage of this setup. Then, create an SFTP connection at  Preferences -> Build, Execution, Deployment -> Deployment. Connect to `localhost` and use `app` for the username & password. You can set additional options for working with Magento in PhpStorm at Preferences -> Build, Execution, Deployment -> Deployment -> Options.

Note that you must use your IDE's SSH/SFTP functionality, otherwise changes will not be synced. To re-sync your host environment at any time, run:

```
bin/copyfromcontainer --all
```
### Grunt + LiveReload for Frontend Development

#### Create a new theme and make it active

Create your new theme at `app/design/frontend/VendorName/theme-name`, with the related `composer.json`, `registration.php` and `theme.xml` files.

Make your new theme active at Admin > Content > Design > Configuration. Click the Edit button next to Global Scope, and set the Applied Theme to your new theme name, and click Save Configuration.

#### Load the LiveReload client file

To create a connection to LiveReload, you'll need to insert the LiveReload script into your theme. You can do this by creating a file in your theme at `Magento_Theme/layout/default_head_blocks.xml` with the contents:

```xml
<?xml version="1.0"?>
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <head>
        <script defer="true" src="/livereload.js?port=443" src_type="url"/>
    </head>
</page>
```

The "?port=443" parameter is important, otherwise the `livereload.js` script won't work.

While we're at it, let's also create an initial LESS file so we have something to test. Create a new file in your theme at `web/css/source/_extend.less` with the contents:

```css
body {
    background: white;
}
```

You'll need to clear the Magento cache to enable your module, and make sure this layout XML update is properly loaded.

Your new theme should now be active at `https://yourdomain.test`. Since this is a new theme, it should appear the same as the parent theme defined in your theme.xml file, which is usually Blank.

#### Set up Grunt

Run `bin/setup-grunt`. This will set up the Grunt configuration files for your new theme. It's important to run this step after setting up your new theme, not before.

#### Start the Grunt watcher

Grunt can watch for filesystem changes by running `bin/grunt watch`. You can optionally pass in the `--verbose` or `-v` flag to toggle verbose mode on. This will let you know what's going on under the hood, so you can be sure it is compiling & watching the correct files, and updating them as changes are made.

#### LiveReload Browser extension

Running the `grunt watch` process also spawns the LiveReload server. Your browser needs to connect to this server, and this is done by installing the [LiveReload browser extension](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei?hl=en).

In your browser, be sure to also open the Google Chrome Dev Tools, go to the Network tab, and click "Disable cache". This will ensure the browser does not long-cache static file assets, such as JavaScript & CSS files, which is important during development.

Ensure the LiveReload browser icon has been toggled on, and refresh the page. We can confirm the LiveReload script is loaded by going to the Network tab and ensuring the `livereload.js` file is loaded, and that it also spawns off a new websocket request to `/livereload`.

#### Test LiveReload

Since this is all set, let's update the CSS file to a different background color:

```css
body {
    background: blue;
}
```

Upon saving this file, we will see the Grunt watcher detect the changes, and your browser should automatically load the new style without you needing to refresh the page, and without a full browser refresh.

## Credits

<a href="https://markshust.com" target="_blank">Mark Shust</a>

## License

[MIT](https://opensource.org/licenses/MIT)
