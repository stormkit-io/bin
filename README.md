# Stormkit Self-Hosted Edition

This repository contains the binaries to run Stormkit on Self-Hosted environments. It's strongly advised to our Docker Images instead of the binaries as the required dependencies are managed for users automatically.

## Getting Started

Check our [docker-compose.yaml](./docker-compose.yaml) file. It's already configured to work on a single machine and contains all microservices needed for Stormkit to function as expected.

## Authentication

You will need to setup your own authentication method.

### GitHub

If you are using GitHub, create a GitHub app:

1. Visit https://github.com/settings/apps (or your organization's app page)
1. Click on New GitHub App
1. Fill in `GitHub App name` and `Homepage URL` as you wish
1. Callback URL: `https://api.example.org/auth/github/callback`
1. Setup URL: `https://api.example.org/auth/github/installation`
1. Check `Redirect on update`
1. Webhook Active: `True`
1. Webhook URL: `https://api.example.org/app/webhooks/github/deploy`
1. Enable SSL verification

PS: replace example.org with your `STORMKIT_DOMAIN`

Next, make sure to grant the following permissions:

| Name                         | Value          |
| ---------------------------- | -------------- |
| `Repository:Administration`  | Read and Write |
| `Repository:Checks`          | Read and Write |
| `Repository:Commit statuses` | Read and Write |
| `Repository:Contents`        | Read-only      |
| `Repository:Pull requests`   | Read and Write |
| `Repository:Webhooks`        | Read-only      |
| `Account:Email addresses`    | Read-only      |

Next, subscribe to the following events:

- Pull request
- Push

Finally, choose `Only on this account` for the question `Where can this GitHub App be installed?`.
This will prevent users outside your organization to create apps on your Stormkit instance.

Now click on the `General` tab and:

1. Create a client secret
1. Create a private key (scroll down to the bottom of the page)

Once you have your GitHub app created, add these additional environment variables:

```bash
# This is your GitHub App ID. This can be found under the General tab of your GitHub App page.
GITHUB_APP_ID=927000

# This is the slug of your GitHub App name. It can be found in the URL:
# https://github.com/settings/apps/stormkit-io
GITHUB_APP_NAME=stormkit-io

# The GitHub App client ID. This can be found under the General tab of your GitHub App page.
GITHUB_CLIENT_ID='Iv1.random_token'

# This is the Client secret that we generated previously after creating the GitHub App.
GITHUB_SECRET='your_secret'

# The Base64 encoded private key generated previously after creating the GitHub App.
# You can use `openssl` to encode your string:
#
# cat my-key.pem | openssl base64 -A
#
# Make sure to enclose your variable with quotes.
GITHUB_PRIV_KEY='WW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5'
```

## Configuring your Domain

Go to your DNS provider and create following A record:

```
*.example.org => IP Address of your Instance
```

Stormkit uses two reserverd subdomains:

`api` and `stormkit`. The rest will be used for deployment previews.

## Automatic SSL

Stormkit handles the SSL certificates for you automatically.

## Starting your stack

Execute the following command to start your swarm stack:

```bash
# If you're using Docker Swarm:
# Initialize stack (only needed for master node)
docker swarm init
docker stack deploy -c docker-compose.yaml stormkit

# If you're using Docker Compose:
docker compose up -d
```

## Rolling updates

You can update your service using the following commands:

```bash
docker compose pull

# If you are using Docker Swarm:
docker stack deploy -c docker-compose.yaml stormkit

# If you are using Docker Compose:
docker compose down "<service_name>" && docker compose up -d "<service_name>"
```

## Updating environment variables

If you need to update environment variables simply run the following command:

```bash
# If you are using Docker Swarm:
docker stack deploy -c docker-compose.yaml stormkit

# If you are using Docker Compose:
docker compose down "<service_name>" && docker compose up -d "<service_name>"
```

Docker Swarm will update only affected containers.

## Useful commands

| Command                                               | Description                                               |
| ----------------------------------------------------- | --------------------------------------------------------- |
| `docker stack ps --no-trunc stormkit`                 | List available services with their status (detailed view) |
| `docker stack services stormkit`                      | List available services                                   |
| `docker stack rm stormkit`                            | Remove the whole stack                                    |
| `docker stack deploy -c docker-compose.yaml stormkit` | Deploy stack or latest changes                            |

## Troubleshooting

- I'm seeing the `no matching manifest for linux/arm64/v8 in the manifest list entries` error

  Try adding `platform: linux/amd64` in your docker-compose.yaml file under each service.

  ```yaml
  // example
  workerserver:
      image: ghcr.io/stormkit-io/workerserver:latest
      platform: linux/amd64
  ```

- `docker stack ps stormkit` displays `no suitable node (unsupported platform on 1 node)` error

  This is due to `docker swarm` not supporing the `platform` property. If you're running on your local environment,
  try running `docker compose up` instead.
