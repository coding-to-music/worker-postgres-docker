# worker-postgres-docker
# Cloudflare Workers + PostgreSQL
# Query Postgres from Workers using a database connector


# 🚀 Javascript full-stack 🚀

https://github.com/coding-to-music/worker-postgres-docker

https://worker-postgres-docker.vercel.app

https://worker-postgres-docker.vercel.app/api/auth

By Cloudflare Documentation

https://github.com/cloudflare/worker-template-postgres/

https://developers.cloudflare.com/workers/tutorials/query-postgres-from-workers-using-database-connectors/



## Environment Values

```java
```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/worker-postgres-docker.git
git push -u origin main
```

This repo contains example code and a PostgreSQL driver that can be used in any Workers project. If
you are interested in using the driver _outside_ of this template, copy the `driver/postgres` module
into your project's `node_modules` or directly alongside your source.

## Usage

Before you start, please refer to the **[official tutorial](https://developers.cloudflare.com/workers/tutorials/query-postgres-from-workers-using-database-connectors)**.

```typescript
const client = new Client({
    user: '<DATABASE_USER>',
    database: '<DATABASE_NAME>',
    // hostname is the full URL to your pre-created Cloudflare Tunnel, see documentation here:
    // https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/create-tunnel
    hostname: env.TUNNEL_HOST || 'https://dev.example.com',
    password: env.DATABASE_PASSWORD, // use a secret to store passwords
    port: '<DATABASE_PORT>',
})

await client.connect()
```

**Please Note:**
- you must use this config object format vs. a database connection string
- the `hostname` property must be the URL to your Cloudflare Tunnel, _NOT_ your database host
    - your Tunnel will be configured to connect to your database host

## Running the Postgres Demo

`postgres/docker-compose.yml`

This docker-compose composition will get you up and running with a local instance of `postgresql`, 
`pgbouncer` in front to provide connection pooling, and a copy of `cloudflared` to enable your 
applications to securely connect, through a encrypted tunnel, without opening any ports up locally.

### Usage

> from within `scripts/postgres`, run: 

1. **Create credentials file (first time only)**
```java
docker run -v ~/.cloudflared:/etc/cloudflared cloudflare/cloudflared:2021.10.5 login
```

2. **Start a local dev stack (cloudflared/pgbouncer/postgres)**
```java
TUNNEL_HOSTNAME=dev.example.com docker-compose up
```

---
updated: 2021-11-15
difficulty: Beginner
content_type: 📝 Tutorial
pcx-content-type: tutorial
title: Query Postgres from Workers using a database connector
---

# Query Postgres from Workers using a database connector

<TutorialsBeforeYouStart />

## Overview

In this tutorial, you will learn how to retrieve data in your Cloudflare Workers applications from a PostgreSQL database using [Postgres database connector](https://github.com/cloudflare/worker-template-postgres).

{{<Aside type="note">}}

If you are using a MySQL database, refer to the [MySQL database connector](https://github.com/cloudflare/worker-template-mysql) template.

{{</Aside>}}

For a quick start, you will use Docker to run a local instance of Postgres and PgBouncer, and to securely expose the stack to the Internet using Cloudflare Tunnel.

## Basic project scaffolding

To get started:

1.  Run the following `git` command to clone a basic [Postgres database connector](https://github.com/cloudflare/worker-template-postgres) project.
2.  After running the `git clone` command, `cd` into the new project.

```java
git clone https://github.com/cloudflare/worker-template-postgres/
cd worker-template-postgres
```

## Install Cloudflared and download the pem certificate

```java
cd to home directory

cd

wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb 

cloudflared tunnel login

It has an error writing to my home directory

so create the target directory

sudo touch .cloudflared


sudo chmod 777 .cloudflared/

cloudflared tunnel login
```

## Cloudflare Tunnel authentication

To create and manage secure Cloudflare Tunnels, you first need to authenticate `cloudflared` CLI.
Skip this step if you already have authenticated `cloudflared` locally.

```java
docker run -v ~/.cloudflared:/etc/cloudflared cloudflare/cloudflared:2021.11.0 login
```

Output

```java
Please open the following URL and log in with your Cloudflare account:

https://dash.cloudflare.com/argotunnel?callback=https%3A%2F%2Flogin.cloudflareaccess.org etc

Leave cloudflared running to download the cert automatically.
2022-05-24T06:51:55Z INF Waiting for login...
2022-05-24T06:52:48Z INF Waiting for login...
You have successfully logged in.
If you wish to copy your credentials to a server, they have been saved to:
~/.cloudflared/cert.pem
```

Verify

```java
ls ~/.cloudflared/
```

```java
cert.pem
```

Running this command will:

- Prompt you to select your Cloudflare account and hostname.
- Download credentials and allow `cloudflared` to create Tunnels and DNS records.

## actually create and name the tunnel

```java
cloudflared tunnel create my-tunnel
```

```java
2022-05-24T09:03:40Z ERR Configuration file /home/tmc/.cloudflared/config.yml was empty
Tunnel credentials written to /home/tmc/.cloudflared/b98f6dff-6605-43c4-b83a-2315e4.json. cloudflared chose this file based on where your origin certificate was found. Keep this file secret. To revoke these credentials, delete the tunnel.

Created tunnel my-tunnel with id b98f6dff-6605-43c4-b83a-2315e4
```

## Confirm that the tunnel has been successfully created by running:

https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide/#set-up-a-tunnel-remotely-dashboard-setup

```java
cloudflared tunnel list
```

Output

```java
2022-05-24T09:08:37Z ERR Configuration file /home/tmc/.cloudflared/config.yml was empty
You can obtain more detailed information for each tunnel with `cloudflared tunnel info <name/uuid>`
ID                                   NAME                 CREATED              CONNECTIONS  
c51576d0-b652-4499-9ab5-f65fe7c355db all-knowledge-tunnel 2022-05-24T07:21:23Z 2xEWR, 2xORD 
b98f6dff-6605-43c4-b83a-2315e409920c my-tunnel            2022-05-24T09:03:41Z              
```

## ​​4. Create a configuration file

Create a configuration file in your .cloudflared directory using any text editor. This file will configure the tunnel to route traffic from a given origin to the hostname of your choice.

Add the following fields to the file:

If you are connecting an application

```java
url: http://localhost:8000
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
```

If you are connecting a network

```java
tunnel: <Tunnel-UUID>
credentials-file: /root/.cloudflared/<Tunnel-UUID>.json
warp-routing:
  enabled: true
```

Confirm that the configuration file has been successfully created by running:

```java
cat ~/.cloudflared/config.yml
```

## Start and prepare Postgres database

### Start the Postgres server

{{<Aside type="warning" header="Warning">}}

Cloudflare Tunnel will be accessible from the Internet once you run the following `docker compose` command. Cloudflare recommends that you secure your `TUNNEL_HOSTNAME` behind [Cloudflare Access](/cloudflare-one/applications/configure-apps/self-hosted-apps) before you continue.

{{</Aside>}}

You can find a prepared `docker-compose` file that does not require any changes in `scripts/postgres` with the following services:

1.  **postgres**
2.  **pgbouncer** - Placed in front of Postgres to provide connection pooling.
3.  **cloudflared** - Allows your applications to connect securely, through a encrypted tunnel, without opening any local ports.

Run the following commands to start all services. Replace `postgres-tunnel.example.com` with a hostname on your Cloudflare zone to route traffic through this tunnel.

```java
cd scripts/postgres
export TUNNEL_HOSTNAME=postgres-tunnel.example.com

# in my case, use my url
export TUNNEL_HOSTNAME=all-knowledge.info
docker-compose up

# Alternative: Run `docker compose up -D` to start docker-compose detached
```

`docker-compose` will spin up and configure all the services for you, including the creation of the Tunnel's DNS record.
The DNS record will point to the Cloudflare Tunnel, which keeps a secure connection between a local instance of `cloudflared` and the Cloudflare network.

### Import example dataset

Once Postgres is up and running, seed the database with a schema and a dataset. For this tutorial, you will use the Pagila schema and dataset. Use `docker exec` to execute a command inside the running Postgres container and import [Pagila](https://github.com/devrimgunduz/pagila) schema and dataset.

```java
curl https://raw.githubusercontent.com/devrimgunduz/pagila/master/pagila-schema.sql | docker exec -i postgres_postgresql_1 psql -U postgres -d postgres
curl https://raw.githubusercontent.com/devrimgunduz/pagila/master/pagila-data.sql | docker exec -i postgres_postgresql_1 psql -U postgres -d postgres
```

The above commands will download the SQL schema and dataset files from Pagila's GitHub repository and execute them in your local Postgres database instance.

## Edit Worker and query Pagila dataset

### Database connection settings

In `src/index.ts`, replace `https://dev.example.com` with your Cloudflare Tunnel hostname, ensuring that it is prefixed with the `https://` protocol:

```js
---
filename: src/index.ts
highlight: [4]
---
const client = new Client({
  user: 'postgres',
  database: 'postgres',
  hostname: 'https://REPLACE_WITH_TUNNEL_HOSTNAME',
  password: '',
  port: 5432,
});
```

At this point, you can deploy your Worker and make a request to it to verify that your database connection is working.

### Query Pagila dataset

The template script includes a simple query to select a number (`SELECT 42;`) that is executed in the database. Edit the script to query the imported Pagila dataset if the `pagila-table` query parameter is present.

```js
// Query the database.

// Parse the URL, and get the 'pagila-table' query parameter (which may not exist)
const url = new URL(request.url);
const pagilaTable = url.searchParams.get('pagila-table');

let result;
// if pagilaTable is defined, run a query on the Pagila dataset
if (
  [
    'actor',
    'address',
    'category',
    'city',
    'country',
    'customer',
    'film',
    'film_actor',
    'film_category',
    'inventory',
    'language',
    'payment',
    'payment_p2020_01',
    'payment_p2020_02',
    'payment_p2020_03',
    'payment_p2020_04',
    'payment_p2020_05',
    'payment_p2020_06',
    'rental',
    'staff',
    'store',
  ].includes(pagilaTable)
) {
  result = await client.queryObject(`SELECT * FROM ${pagilaTable};`);
} else {
  const param = 42;
  result = await client.queryObject(`SELECT ${param} as answer;`);
}

// Return result from database.
return new Response(JSON.stringify(result));
```

## Worker deployment

In `wrangler.toml`, enter your Cloudflare account ID in the line containing `account_id`:

{{<Aside type="note">}}

[Refer to Get started](/workers/get-started/guide#7-configure-your-project-for-deployment) if you need help finding your Cloudflare account ID.

{{</Aside>}}

```toml
---
filename: wrangler.toml
highlight: [3]
---
name = "worker-postgres-template"
type = "javascript"
account_id = ""
```

Publish your function:

```java
wrangler publish
✨  Built successfully, built project size is 10 KiB.
✨  Successfully published your script to
 https://workers-postgres-template.example.workers.dev
```

### Set secrets

Create and save [a Client ID and a Client Secret](/cloudflare-one/identity/service-auth/service-tokens) to Worker secrets in case your Tunnel is protected by Cloudflare Access.

```java
wrangler secret put CF_CLIENT_ID
wrangler secret put CF_CLIENT_SECRET
```

### Test the Worker

Request some of the Pagila tables by adding the `?pagila-table` query parameter with a table name to the URL of the Worker.

```java
curl https://example.workers.dev/?pagila-table=actor
curl https://example.workers.dev/?pagila-table=address
curl https://example.workers.dev/?pagila-table=country
curl https://example.workers.dev/?pagila-table=language
```

## Cleanup

Run the following command to stop and remove the Docker containers and networks:

```java
docker compose down

# Stop and remove containers, networks
```

## Related resources

If you found this tutorial useful, continue building with other Cloudflare Workers tutorials below.

<!-- - [Authorize users with Auth0](/workers/tutorials/authorize-users-with-auth0/) -->
- [Build a Slackbot](/workers/tutorials/build-a-slackbot/)
- [GitHub SMS notifications using Twilio](/workers/tutorials/github-sms-notifications-using-twilio/)