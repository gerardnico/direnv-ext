# Direnv ext - Add vault and multi-env support to direnv


## About

Do you want to:
* add secrets from [Hashicorp vault](https://www.vaultproject.io/)
* use multiple environments files (`.env`,`.env.docker`)
for your project ?

The `direnv extensions script` is here to help.

## Example

With this `.env` file in your project root:
* Add one or more `.env` file in your project root
```ini
POSTGRES_USER=postgres
POSTGRES_PASSWORD=vault:secret/postgres/password
```
You would get 
```bash
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=welcome # value from vault
export DIRENV_EXPORTED_ENVS=POSTGRES_USER,POSTGRES_PASSWORD
```

`DIRENV_EXPORTED_ENVS` may be used to recreate a list of env. 
For instance, [for docker in the form `-e KEY=VALUE`](#how-to-use-direnv_exported_envs-to-list-and-pass-envs-as-argument-at-the-command-line)


## How it works

You create an `.envrc` file at the root of your project
with the following command
```bash
eval "$(direnv-ext-print)"
```

## Vault Value Syntax

The vault syntax value is `vault:$ENGINE/path/to/secret/fieldName`

You would have installed vault locally and initialized it with
```bash
vault login
```


## How to use DIRENV_EXPORTED_ENVS to list and pass envs as argument at the command line

This example shows you how to pass environment variable to docker in the form `-e KEY=VALUE`
```bash
declare -a DOCKER_ENV_ARGS=()
IFS=',' read -ra DIRENV_EXPORTED_KEYS_AS_ARRAY <<< "$DIRENV_EXPORTED_KEYS"
for ENV in "${DIRENV_EXPORTED_KEYS_AS_ARRAY[@]}"; do
    # Filter out all docker variable
    if [[ $ENV == DOCKER* ]]; then
        continue
    fi
    if [[ -v $ENV ]]; then
        DOCKER_ENV_ARGS+=("-e")
        DOCKER_ENV_ARGS+=("$ENV=${!ENV}")
    fi
done

# then add them to docker
docker run \
  "${DOCKER_ENV_ARGS[@]}" 
```

## Install

### Homebrew

* Install `direnv-ext` with [homebrew](https://brew.sh/)
```bash
brew install --HEAD gerardnico/tap/direnvext
```

### Configuration
You may want to add a higher timeout in your `direnv.toml` to avoid the timeout warning.
```toml
[global]
# to avoid: direnv is taking a while to execute
# a call to the external http vault may take a long time
warn_timeout = "3s"
```
