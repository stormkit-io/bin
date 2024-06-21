# Stormkit Self-Hosted Edition

This repository contains the binaries to run Stormkit on Self-Hosted environments. It's strongly advised to our Docker Images instead of the binaries as the required dependencies are managed for users automatically.

## Getting Started

Check our [docker-compose.yaml](./docker-compose.yaml) file. It's already configured to work on a single machine and contains all microservices needed for Stormkit to function as expected.

Once you have configured the containers, you will need to set the following required environment variables:

```bash
# This tells under which domain to host Stormkit.
# In this example:
#
# stormkit.example.org => Stormkit Frontend
# api.example.org      => Stormkit API
# *.example.org        => Deployment Previews
STORMKIT_DOMAIN=example.org

# The application secret used internally to encrypt/decrypt variables.
# This must be a 32 characters long alphanumeric variable.
STORMKIT_APP_SECRET=xzSX0KEntt9EO8pQM0msmVm3WkQktbJu

# The username used to login to the Admin Dashboard. This is where you will
# deploy the Stormkit Frontend. After the initial deployment, this can be removed.
STORMKIT_ADMIN_USERNAME=root

# The password used to login to the Admin Dashboard. This is where you will
# deploy the Stormkit Frontend. After the initial deployment, this can be removed.
STORMKIT_ADMIN_PASSWORD=123456

# The host name of the database. For docker-compose, this is the name of the service.
POSTGRES_HOST=db

# The port to connect to the database.
POSTGRES_PORT=5432

# The name of the database.
POSTGRES_DB=stormkit_db

# The postgres user to connect to the database.
POSTGRES_USER=stormkit_admin

# The password of POSTGRES_USER.
POSTGRES_PASSWORD=123456

# The address for the redis instance. This is a passwordless service so make sure to host
# your Redis instance in a private cloud. For docker compose, this is the name of the service.
# The port number has to specified as well.
REDIS_ADDR=redis:6379
```

PS: Make sure to generate your own values.

## Authentication

Once deployed your Stormkit Frontend you may notice that the UI provides no option to authenticate.
This is because you will need to setup your own authentication method.

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
This will prevent users outside your organization to sign up to your Self-Hosted Stormkit instance.

Now click on the `General` tab and:

1. Create a client Secret
1. Create a private key (at the end of the page)

Once you have your GitHub app created, add these additional environment variables:

```bash
# This is your GitHub App ID. This can be found under the General tab of your GitHub App page.
GITHUB_APP_ID=927000

# The account owner of the GitHub App.
# This can be found next to the `Owned by:` in the General tab of your GitHub App page.
GITHUB_ACCOUNT=stormkit-io

# The GitHub App client ID. This can be found under the General tab of your GitHub App page.
GITHUB_CLIENT_ID='Iv1.<random_token'

# This is the Client secret that we generated previously after creating the GitHub App.
GITHUB_SECRET='your_secret'

# The Base64 encoded private key generated previously after creating the GitHub App.
# You can use an online service such as https://www.base64encode.org/ to encode your secret.
# Make sure to enclose your variable with quotes.
GITHUB_PRIV_KEY='WW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5IC0gWW91ciBiYXNlIDY0IGVuY29kZWQgZ2l0aHViIHByaXZhdGUga2V5'
```

Finally, restart the services so that environment variables are picked.
