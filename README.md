# Docker Hook

> Automatic Docker Deployment via [Webhooks](https://docs.docker.com/docker-hub/repos/#webhooks)

`docker-hook` listens to incoming HTTP requests and triggers your specified command.

## Setup

### 1. Prepare Your Server

#### Download `docker-hook`

No worries - it just downloads a Python script. There won't be anything installed or written elsewhere.

    curl https://raw.githubusercontent.com/yasaricli/docker-hook/master/docker-hook > docker-hook
    chmod +x docker-hook

#### Start `docker-hook`

```sh
$ docker-hook -t <auth-token> -c <command>
```

##### Auth-Token

Please choose a secure `auth-token` string or generate one with `$ uuidgen`. Keep it safe or otherwise other people might be able to trigger the specified command. You can pass the `auth-token` as a command line argument, or set it as an environment variable.

##### Command

The `command` can be any bash command of your choice. See the following [example](#example). This command will be triggered each time someone makes a HTTP request.

### 2. Configuration On Docker Hub

Add a webhook like on the following image. `example.com` can be the domain of your server or its ip address. `docker-hook` listens to port `8555`. Please replace `my-super-safe-token` with your `auth-token`.

![](http://i.imgur.com/B6QyfmC.png)

## Example

This example will stop the current running `yourname/app` container, pull the newest version and start a new container.

```sh
$ docker-hook -t my-super-safe-token -c sh ./deploy.sh
```

If you want to store the authentication token as an environment variable, then run
```sh
# Make sure there is no space before or after the `=` sign
$ export DOCKER_AUTH_TOKEN=<my-super-safe-token>
$ docker-hook -c sh ./deploy.sh
```
#### `deploy.sh`

```sh
#! /bin/bash

docker ps | grep $REPOSITORY_NAME | awk '{print $1}' | xargs docker stop
docker pull $REPOSITORY_NAME
docker run -d $REPOSITORY_NAME
```

You can now test it by pushing something to `yourname/app` or by running the following command where `yourdomain.com` is either a domain pointing to your server or just its ip address.

```sh
$ curl -X POST yourdomain.com:8555/my-super-safe-token
```

## How it works

`docker-hook` is written in plain Python and does have **no further dependencies**. It uses `BaseHTTPRequestHandler` to listen for incoming HTTP requests from Docker Hub and then executes the provided [command](#command) if the [authentification](#auth-token) was successful.

## Caveat

This tool was built as a proof-of-concept and might not be completly secure. If you have any improvement suggestions please open up an issue.

## Options

Env variables to pass into [deploysh](#deploy.sh)

```
REPOSITORY_STATUS
REPOSITORY_REPO_NAME
REPOSITORY_DESCRIPTION
REPOSITORY_NAMESPACE
REPOSITORY_IS_TRUSTED
REPOSITORY_FULL_DESCRIPTION
REPOSITORY_COMMENT_COUNT
REPOSITORY_STAR_COUNT
REPOSITORY_REPO_URL
REPOSITORY_OWNER
REPOSITORY_DATE_CREATED
REPOSITORY_IS_OFFICIAL
REPOSITORY_DOCKERFILE
REPOSITORY_IS_PRIVATE
REPOSITORY_NAME
```

## License

[MIT License](http://opensource.org/licenses/MIT)
