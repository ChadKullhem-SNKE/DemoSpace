# Xplore Spine Admin Portal

This is the web based admin portal for the Xplore Spine app. It provides access to an instructor portal via Brainlab ID authentication and serves as the login portal for Xplore Spine users via an in game browser.

## Development

To run this app locally you just need to run `npm install` and then `npm start` in the `server` directory.

Before the first time you run it you must create a `.env` as described below.

### Required Node Version

From what I can tell node `v20.11.1` is REQUIRED for the version of better-sqlite3 we're using to work. On WSL I tried running on `v22` and `v24` and it failed to start with an error about missing native bindings.

If you're using NVM (you really should be) you can install the correct version as follows

```bash
# if you don't have this version yet
nvm install v20.11.1

# to select it as the version of node to use
nvm use v20.11.1
```

### Secrets

To start this application you must have the secrets for the Brainlab ID authentication system (saml). These can be retrieved from the 1Password vault `Xplore Spine Brainlab ID Certs`

Download 

- `saml-brainlab.key` - put in (gitignored) `certs` directory in root of repository
- `saml-brainlab.crt` - put in (gitignored) `certs` directory in root of repository
- `xplore-spine-env-config-dev` - rename to `.env` and put in `server` directory

There are two paths (key/crt) in the .env file which should point to the path of the certs directory where you just placed the key/crt file. These files will be loaded when the server starts.

## Configuration

The following values must be defined in production (add them to ~/.profile)

Note some characters have special meaning in a bash script (e.g. `$`). So, if any values contain special characters they should be wrapped in single quotes (e.g. `export SOMETHING='my$special$value'`)

```bash
# port the web server should listen on
export PORT=443
# hostname of the service (used for SAML identity)
export HOST_NAME=
# SAML X.509 public key of identity provider
export SAML_IDP_PUBLIC_KEY=
# SAML identity provider entity id (url)
export SAML_IDP_ENTRY=
# SAML identity provider single logout service url
export SAML_IDP_SLS=
# SAML account management url (user is sent here if they click the "account" nav item)
export SAML_ACCOUNT_URL=
# Absolute path to the SAML certificate (public key) for this service
export SAML_SP_CERT_PATH=
# Absolute path to the SAML private key for this service
export SAML_SP_KEY_PATH=
# Absolute path where the sqlite databases should be stored (directory with write access)
export DB_PATH=
# Absolute path to the TLS private key to use to establish https
export TLS_KEY_PATH=
# Absolute path to the TLS certificate (public key) to use to establish https
export TLS_CERT_PATH=
# A unique string to use for securing session data
export SESSION_SECRET=
# Login user for gmail SMTP server used to send emails
export EMAIL_USER=
# Login password for gmail SMTP server used to send emails
export EMAIL_PASSWORD=
# Url of the brainlab id api
export BRAINLAB_API_URL=
# Secret api key for the brainlab id api
export BRAINLAB_API_KEY=
# Username / password used in basic auth for the brainlab id api. format: username:password
export BRAINLAB_API_AUTH=
# Url for the licensing API (Brainlab Quotebase Service bus)
export LICENSE_API_URL=
# Secrate api key for licensing API (Brainlab Quotebase Service bus)
export LICENSE_API_KEY=
# Secret api key for the Level Ex Analytics Data Warehouse
DATA_WAREHOUSE_KEY=
```

For development this app looks for a `.env` file in the root of the `server` directory. The allowed config values are the same as above and just need to be included in the format KEY=value (no quotes, no spaces between the equal and key/value). The only values required for development are `NODE_ENV` (must be dev) and `DB_PATH`

Minimal example:

```bash
NODE_ENV=dev
DB_PATH=/valid/path/to/store/sqlite/database
```

These values will be loaded automatically using the new env file support in node v20+ (as opposed to a 3rd party library). See the scripts in the package.json file that include the `--env-file=` for an example. To make this work we have to use the full relative path to the nest binary.

Note, setting `NODE_ENV=dev` will bypass saml auth and use mock users to make it easier to work on pages locally. Anything, that depends on real SAML auth has to be done on an actual server with a https enabled hostname.

## Deploying

This app is deployed via Github Actions using release branches. Any update to the `release/dev` branch will be automatically deployed to the dev environment. To deploy to the production environment merge changes into the `releases/prod` branch to trigger a deployment. As of this writing the deploy takes about 1 minute to complete and results in less than 10 seconds of downtime.

```bash
# Starting from main branch

# ensure you have all the latest changes from main
git pull

# switch to the release branch (releases/dev or releases/prod)
git checkout releases/dev

# merge changes from main to the release branch
get merge main

# push the changes to the release branch (this will trigger a deployment)
git push
```

Watch the deploy status here: https://github.com/LevelEx/spinex-admin-portal/actions

### Environments

| Environment | Branch        | url                                 |
| ----------- | ------------- | ----------------------------------- |
| Development | releases/dev  | https://spinex-dev.apps.lvlex.link/ |
| Production  | releases/prod | https://spinex.apps.lvlex.link/     |

### Manual deploy

You can also manually deploy by running `deploy.sh` in the root of the project and providing the target host as an argument.

```bash
./deploy.sh spinex.v2.lvlex.link
```

Ensure you have the ssh key in your ssh agent before running the deploy script. You can add it with `ssh-add /path/to/the/key/privkey.pem`.

## Logs

This app runs using the [systemd](https://systemd.io/) the built in system/service manager shipped with ubuntu. This will automatically restart the app if it crashes. Additionally, it handles all of our logs.

You can use the following commands to see logs

```bash
# show logs of standard out
cat /var/log/spinex/out.log

# show logs of standard error
cat /var/log/spinex/err.log

# show end of logs and follow for output
tail -f /var/log/spinex/out.log

```

## Initial deploy / server setup

This app is currently being deployed to simple EC2 instances. Dev is running in the V2 AWS Account and prod is running in the brainlab AWS account. This app could also easily be containerized in the future if that is desired. You'd just have to pass in the required environment variables and expose port 443.

For details about manual set up of the EC2 instances see [notes/server-setup.md](notes/server-setup.md).
