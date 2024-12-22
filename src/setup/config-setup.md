# Configuration

This page will teach you all about the configuration file/environment variables required for your Liphium town. If you haven't followed a guide for installation, please find one that suites your needs right in the menu/sidebar.

If you came here from an installation tutorial, you can skip straight to the [creating a new configuration](#creating-a-new-configuration) part.

## An overview of the configuration

First, here is a basic overview with explanations of every configuration value.

```sh
# Domain config
#
# The base paths of all your servers and their servers. Liphium uses this
# to tell the client the domain of the server.
BASE_PATH=main.liphium.com
BASE_PORT=4000
CHAT_NODE=chat.liphium.com
CHAT_NODE_PORT=4001
SPACE_NODE=spaces.liphium.com
SPACE_NODE_PORT=4002

# App config
#
# APP_NAME is used in emails (you can change it) to the people in your town.
# The other values should just be kept the default (if you're not a dev).
# LISTEN is what address station will listen on.
# CLI enables a command line utility for developers (off by default).
# PROTOCOL is how the backend will contact the chat server and space station.
APP_NAME=Liphium
LISTEN=0.0.0.0
CLI=false
PROTOCOL=https://

# Test mode config
#
# TESTING is if station should be ran in testing mode or not.
# We recommend leaving TESTING on to see logs more clearly and be able to report
# errors to us more easily.
# TESTING_AMOUNT generates testing tokens. 0 is the best value to keep it at.
# If it isn't there, station will crash on startup.
TESTING=true
TESTING_AMOUNT=0

# Database for the main backend server
#
# The database credentials the main backend will use for accounts, files and more.
DB_USER=postgres
DB_PASSWORD=pw
DB_DATABASE=main
DB_HOST=address
DB_PORT=5432

# Database for the chat server
#
# The database credentials the chat server will use for conversations, status
# and more.
CN_DB_USER=postgres
CN_DB_PASSWORD=pw
CN_DB_DATABASE=chat
CN_DB_HOST=address
CN_DB_PORT=5432

# The secret for all JWT tokens used by Liphium, mostly connection tokens.
JWT_SECRET=generate_some_random_value

# File storage configuration
#
# How and where your files are stored.
# This should not be modified unless you know what you are doing. Read:
# https://docs.liphium.com/setup/config-setup.html#file-storage-configuration
FILE_REPO_TYPE=local
FILE_REPO=/home

# SMTP credentials
#
# The SMTP credentials used to connect directly to your mail server.
SMTP_SERVER=mail.example.com
SMTP_PORT=0000
SMTP_IDENTITY=liphium
SMTP_FROM=no-reply@example.com
SMTP_USER=user
SMTP_PW=pw

# A public and private key for reverse proxy protection.
#
# I commented it out here because I wanna prevent weird errors because of station
# generating this automatically and then the user having to paste it into this file.
#
# TC_PUBLIC_KEY=key_here
# TC_PRIVATE_KEY=key_here

# The UUID for the system (will also be printed by station)
# SYSTEM_UUID=uuid_here
```

### File storage configuration

First of all, **do not change your file storage configuration if there are any files uploaded to your instance**. There are ways to change it, but I currently don't want to write documentation about it. If you run into this case, please create an issue [in this repository](https://github.com/Liphium/docs) and I will provide instructions on how to migrate. The process is too complicated to explain here.

If you don't want to change it, but just learn how to configure it, here is an overview:

{{#tabs }}
{{#tab name="Local storage" }}

Configuring local file storage is quite easy. Set the repository type to `local` and just point it to a directory. In this case we use `/home` but you can change it to anything. Relative paths (like `../storage`) are not tested and we have no intention of ever fixing them in case they don't work.

```sh
FILE_REPO_TYPE=local
FILE_REPO=/home
```

{{#endtab }}
{{#tab name="S3-compatible storage" }}

For any S3-compatible storage, you can configure like specified below. Please don't worry about the `r2` type. This was implemented when I was using Cloudflare R2 (their S3 compatible storage), but has since then been validated with more S3 providers and should work with any S3-compatible API, in case you experience any issues, please reach out to us.

`FILE_REPO` and `FILE_REPO_BUCKET` will be concatenate to form a URL that station will reach out to.

Read and write permissions to the the specified bucket are enough. Station will never create buckets or need other ones. Please manage your permissions well.

```sh
FILE_REPO_TYPE=r2
FILE_REPO_KEY_ID=s3_key_id
FILE_REPO_KEY=s3_secret_key
FILE_REPO=s3_bucket_url
FILE_REPO_BUCKET=s3_bucket_name
```

{{#endtab }}
{{#endtabs }}

## Creating a new configuration

This guide is specifically for creating an environment/configuration file for your town. If you aren't coming from one of the official tutorials or some installation guide, please follow that first, until you are sent back to this page.

**1.** You already created subdomains on your domain for each of the servers that Liphium exposes in step 3. Change the following values of the configuration file above to point to your domain (replace example.com with your domain):

- BASE_PATH=main.example.com
- CHAT_NODE=chat.example.com
- SPACE_NODE=spaces.example.com

**Please do not add https:// or http:// to your domain**. This is a common issue that breaks functionality of Liphium.

**2.** You created a main and chat PostgreSQL database before in this guide. Please also fill the data needed for the connection into the following configuration fields:

- DB_USER=your_username
- DB_PASSWORD=your_password
- DB_DATABASE=main
- DB_HOST=postgres_address (to reach the local system use `172.17.0.1` instead of `localhost`, [here's why](https://forums.docker.com/t/how-to-reach-localhost-on-host-from-docker-container/113321))
- DB_PORT=5432 (default port of postgres)

Now repeat the same for the database configuration of the chat server:

- CN_DB_USER=your_username
- CN_DB_PASSWORD=your_password
- CN_DB_DATABASE=chat
- CN_DB_HOST=postgres_address (to reach the local system use `172.17.0.1` instead of `localhost`, [here's why](https://forums.docker.com/t/how-to-reach-localhost-on-host-from-docker-container/113321))
- CN_DB_PORT=5432 (default port of postgres)

**3.** Please generate a random string on some website or in your favorite password manager to paste into the `JWT_SECRET` configuration value. Please be sure to make it **extra long** (like **80-100 characters** should be enough) as this is a really important thing that you don't want others to guess. You can also not change this very easily in the future. So make sure you use something random and very long.

**4.** Now change the following configuration values for your mail server that Station will use:

- SMTP_SERVER=mail.example.com (the domain of your mail server, replace `example.com` with your domain)
- SMTP_PORT=your_smtp_port
- SMTP_IDENTITY=your_smtp_identity
- SMTP_FROM=no-reply@example.com (the email you want Liphium to use)
- SMTP_USER=your_smtp_username
- SMTP_PW=your_smtp_password

**5.** Now you need to decide which kind of storage you want to use for your files. If you just want a folder, go to the local storage tab below. If you want to use Cloudflare R2, AWS S3 or other file storage providers, click the tab for that.

{{#tabs }}
{{#tab name="Local storage" }}

With the start command all of our official guides use, station just saves all of the files into the home directory by default, so we'll use that as the example here.

Configuring local file storage is quite easy. Set the `FILE_REPO_TYPE` to `local` and just point `FILE_REPO` to a directory. You can do it just like below.

```sh
FILE_REPO_TYPE=local
FILE_REPO=/home
```

{{#endtab }}
{{#tab name="S3-compatible storage" }}

For any S3-compatible storage, you can configure like specified below. Please don't worry about the `r2` type. This was implemented when I was using Cloudflare R2 (their S3 compatible storage), but has since then been validated with more S3 providers and should work with any S3-compatible API, in case you experience any issues, please reach out to us.

`FILE_REPO` and `FILE_REPO_BUCKET` will be concatenate to form a URL that station will reach out to.

Read and write permissions to the the specified bucket are enough. Station will never create buckets or need other ones. Please manage your permissions well.

```sh
FILE_REPO_TYPE=r2
FILE_REPO_KEY_ID=s3_key_id
FILE_REPO_KEY=s3_secret_key
FILE_REPO=s3_bucket_url
FILE_REPO_BUCKET=s3_bucket_name
```

{{#endtab }}
{{#endtabs }}

You've now successfully created a configuration for your town. More values will need to be added, but each tutorial will cover that in the next few steps.
