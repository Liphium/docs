# Docker

So you want to make your own town, eh? Well **if you've never touched a server you probably shouldn't attempt this** as servers (especially with Linux) are not a newbie-friendly environment. But with that out of the way or if you're seeking a challenge, let's get a fresh new town onto your server!

Also before we get started, you can always message me on Discord if you have any kind of problem with the installation or have some kind of other question. You can get to the Liphium Discord server through the invite in the navigation bar or menu at the top of the page.

### Requirements

Please make sure you have everything you need installed before installing Liphium because otherwise it can be quite a bit of pain to get everything set up.

- A domain that you will [never change or sell](docs/general/faq/#why-do-i-need-a-domain-for-my-town) (read the linked FAQ question for more information).
- A server running Linux.
- The latest version of Docker. You can learn how install it [here](https://docs.docker.com/engine/install/).
- A mail server for Liphium to send you emails with SMTP credentials (it is required or registration will fail when people try to make an account).
- A PostgreSQL server that you can connect to.
- Decent skills with Docker (there might be issues).

### Tools we will use

Alright, one more thing before we get started on the installation. I just want you to know what we are going to use so everything is clear. I'll have instructions on how to install Nginx and Certbot later. Although I just link you to their websites, respectively.

- **Nginx**: Nginx is a reverse proxy for web requests that we'll use to expose your Liphium town to the outside world. We use it here because we're gonna create multiple domains pointing to the your server and it wouldn't be possible without nginx. We also need it to install SSL certificates for your domain so you can use the app properly.
- **PostgreSQL**: The database we're going to use for Liphium. All of your data is stored in here.
- **Certbot**: Well, this is gonna be the way we make your Liphium town secure by installing SSL certificates for your domain. It's completely free and really easy to use. I even donated 10€ because it's just awesome. [You can too](https://supporters.eff.org/donate/support-work-on-certbot).
- **Docker**: Docker is going to be used to install the actual Liphium server (called station) onto your server. We're going to install PostgreSQL using Docker as well.

With all of that explained, let's finally get to installing.

### Step 1: Getting your PostgreSQL server ready

Connect to your PostgreSQL server and let's create a few databases. We're going to need two databases: One for all of the accounts and one for all of the chat messages. You can create those two by typing `CREATE DATABASE chat; CREATE DATABASE main;`. Different names can also be used, just be sure to use the correct ones later.

### Step 2: Creating the actual Liphium town

**1.** Alright, now it's time for you to venture out by yourself for the first time. And that's to [installing Docker](https://docs.docker.com/engine/install/). They have amazing documentation over there and you should hopefully be able to get it running your machine. Complete it until the end where they tell you to run the hello-world image and when you've done all of that, you can come back here.

**2.** If you haven't done so already, it's a good time to make a folder for all of the files for your Liphium town now (like /home/liphium or something). But now, let's pull the official Liphium image from Docker Hub: `docker pull liphium/chat:latest`.

**3.** Before we start getting into the configuration file, it's a good time to actually get your domain records set up. They will sometimes take a long time to be applied and if we're lucky they will already be ready when we get to actually connecting to the town.

| Type | Name   | Content      | Description                                                  |
| ---- | ------ | ------------ | ------------------------------------------------------------ |
| A    | main   | YOUR_IP_HERE | The domain of the main instance (will be entered in Liphium) |
| A    | chat   | YOUR_IP_HERE | The domain of your chat server                               |
| A    | spaces | YOUR_IP_HERE | The domain of your Spaces server                             |

# Separate config setup for future guides

**4.** Now let's create an environment file for your Liphium town. Copy the file below and call it `config.env`.

```sh
# BACKEND CONFIGURATION

# Domain config
BASE_PATH=main.liphium.com
BASE_PORT=4000
CHAT_NODE=chat.liphium.com
CHAT_NODE_PORT=4001
SPACE_NODE=spaces.liphium.com
SPACE_NODE_PORT=4002

# App config
APP_NAME=Liphium
TESTING=true
LISTEN=0.0.0.0
TESTING_AMOUNT=2
PROTOCOL=https://
CLI=false

# Database for the main server
DB_USER=postgres
DB_PASSWORD=pw
DB_DATABASE=main
DB_HOST=address
DB_PORT=5432

# Database for the chat server
CN_DB_USER=postgres
CN_DB_PASSWORD=pw
CN_DB_DATABASE=chat
CN_DB_HOST=address
CN_DB_PORT=5432

# JWT
JWT_SECRET=generate_some_random_value

# File storage folder
FILE_REPO_TYPE=local
FILE_REPO=/home

# SMTP (for emails)
SMTP_SERVER=mail.example.com
SMTP_PORT=0000
SMTP_IDENTITY=liphium
SMTP_FROM=no-reply@example.com
SMTP_USER=user
SMTP_PW=pw
```

**5.** You already created subdomains on your domain for each of the servers that Liphium exposes in step 3. Change the following values of the configuration file above to point to your domain (replace example.com with your domain):

- BASE_PATH=main.example.com
- CHAT_NODE=chat.example.com
- SPACE_NODE=spaces.example.com

**Please do not add https:// or http:// to your domain**. This is a common issue that breaks functionality of Liphium.

**6.** You created a main and chat PostgreSQL database before in this guide. Please also fill the data needed for the connection into the following configuration fields:

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

**7.** Please generate a random string on some website or in your favorite password manager to paste into the `JWT_SECRET` configuration value. Please be sure to make it **extra long** (like **80-100 characters** should be enough) as this is a really important thing that you don't want others to guess. You can also not change this very easily in the future. So make sure you use something random and very long.

**8.** Now change the following configuration values for your mail server that Station will use:

- SMTP_SERVER=mail.example.com (the domain of your mail server, replace `example.com` with your domain)
- SMTP_PORT=your_smtp_port
- SMTP_IDENTITY=your_smtp_identity
- SMTP_FROM=no-reply@example.com (the email you want Liphium to use)
- SMTP_USER=your_smtp_username
- SMTP_PW=your_smtp_password

# Continue working here (on S3)

**9.** Now you need to decide which kind of storage you want to use for your files. If you just want a folder, go to the local storage tab below. If you want to use Cloudflare R2, AWS S3 or other file storage providers, click the tab for that.

{{#tabs }}
{{#tab name="Local storage" }}
Create a `files` folder in your current directory. This is where all the files uploaded to Liphium will be stored. Everything should work if you follow the guide now. Just as a note, you can optionally take a look into the `FILE_REPO` configuration value. That's where the files will be put inside of your container (/home is the default).
{{#endtab }}
{{#tab name="S3-compatible storage" }}
still needs work
{{#endtab }}
{{#endtabs }}

**10.** Now run the docker container by using the command below. Make sure to replace `config.env` with the name of your environment file.

```sh
docker run --env-file config.env -p 127.0.0.1:4002:4002 -p 127.0.0.1:4001:4001 -p 127.0.0.1:4000:4000 -v ./files:/home liphium/chat
```

**11.** Alright, after running the container once, it should print out that you should set `TC_PUBLIC_KEY` and `TC_PRIVATE_KEY`. Take the thing the container printed out and just add it to the bottom of `config.env`. Make sure to set both of those environment variables from the same run of Liphium. Otherwise they are not going to work together. Also make sure you remove the all the spaces and the double quotes. It should look something like this: TC_PUBLIC_KEY=value. Do the same for `TC_PRIVATE_KEY` as well.

**12.** Now, for the last time, run the container again. It's going to ask you to set the `SYSTEM_UUID` environment variable to the thing it prints out. Also give it the same treatment as above and make it look like this: SYSTEM_UUID=value.

**13.** Your setup of the main Liphium town is now complete. You should now be able to just run the thing and it should already be connecting to the database. The only thing that shouldn't work is grabbing the server public key. But we're going to fix that now.

### Step 3: Exposing your town to the internet with Nginx

**1.** First, you of course need to install Nginx. If you are on Ubuntu or Debian, you can just run the command below. If not, you might have to look up a tutorial on how to do it or just use the package your package manager provides (if it exists).

```sh
apt install nginx
```

**2.** After installing Nginx, we're going to need to set up a few sites. There should already be a directory called `/etc/nginx/sites-available`, but if it is not there, just create it. Enter this directory.

**3.** Now you're going to need two configs again. As usual, just download them from [here](https://gist.github.com/Unbreathable/0469cfd271b84340429c140dde830642). This time it's three files though. Make sure you grab all of them and put them into the `/etc/nginx/sites-available` folder.

**4.** Change the domains in the configurations (behind `server_name`) you just downloaded from me to reflect your domain setup (for example: main.liphium.com -> main.example.com). Make sure to not forget the semicolon at the end!

**5.** Let's make Nginx actually use those configurations. To do that just run the commands below to create links to the files in the `/etc/nginx/sites-enabled` folder.

```sh
ln -s /etc/nginx/sites-available/liphium-main /etc/nginx/sites-enabled/liphium-main
ln -s /etc/nginx/sites-available/liphium-chat /etc/nginx/sites-enabled/liphium-chat
ln -s /etc/nginx/sites-available/liphium-spaces /etc/nginx/sites-enabled/liphium-spaces
```

**6.** Run `nginx -t` to validate that the configurations work properly. If they don't, do what the command tells you to fix the configurations up bit by bit.

**7.** You can now restart Nginx by typing `systemctl restart nginx`. You should now have the configurations loaded.

### Step 4: Adding SSL certificates to your town using Certbot

**1.** For Certbot to work, there are actually two things you need to install. You can follow [this guide](https://certbot.eff.org/instructions?ws=nginx&os=snap) from the official website. **Please follow their guide until step 5**.

**2.** To secure your town, we first need to make sure your domains are already updated and already redirect to the correct server. To verify if they are already set up correctly, go to `main.yourdomain.com` (or whatever you used) and check if there is some sort of error from nginx. If it's not there yet, you'll have to wait a little bit before getting your certificates. This is because your domain isn't pointing to the server yet. This can take up to 48 hours to happen in some cases. So if this happened to you just be patient for now, I know this sucks, but it can happen.

**3.** When you did see the error from nginx, you can now just run `certbot --nginx` to apply the certificates. The CLI will ask your for a few things. For example, they'll ask you to enter your email address. Please do, so you are always notified if something is wrong with your certificates. When they finally ask which domains you want to secure, just leave the field blank to select all of them. Certbot will then do its magic and you are officially done with the setup.

### Step 5: Getting into your town

When you now connect to your Liphium town for the first time with any client app, you can enter the `SYSTEM_UUID` environment variable as the invite. This is an invite created to make sure an account can be created.

If you just created the first account on your Liphium town, it will automatically have admin permissions. You can now go to Settings -> Invites -> Generate invite to invite other people to your town.

### We're finally done

I know it took a long time to get Liphium installed. Thanks for following this guide until now and I hope you didn't run into too many issues. I'm tired now and I'm gonna go watch a few episodes of Shikanoko now. This thing took like 3-4 hours to write and I hope it helped.
