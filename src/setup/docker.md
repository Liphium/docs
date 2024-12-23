# Docker

So you want to make your own town, eh? Well **if you've never touched a server you probably shouldn't attempt this** as servers (especially with Linux) are not a newbie-friendly environment. But with that out of the way or if you're seeking a challenge, let's get a fresh new town onto your server!

Also before we get started, you can always message me on Discord if you have any kind of problem with the installation or have some kind of other question. You can get to the Liphium Discord server through the invite in the navigation bar or menu at the top of the page.

One more thing I want to add is that the software we're going to be installing is called station. It's the backend I created that acts as the base of your town. Just so you aren't confused when you read it throughout this guide.

## Requirements

Please make sure you have everything you need before trying to install station because otherwise it can be quite a bit of pain to get everything set up.

- A domain that you will never change or sell (because otherwise the entire platform falls apart as the client relies on this)
- A mail server for Liphium to send you emails with SMTP credentials (it is required or registration will fail when people try to make an account).
- A PostgreSQL server that you can connect to.

## Tools we will use

Alright, one more thing before we get started on the installation. I just want you to know what we are going to use so everything is clear. I'll have instructions on how to install Nginx and Certbot later. Although I just link you to their websites, respectively.

- **Nginx**: Nginx is a reverse proxy for web requests that we'll use to expose your Liphium town to the outside world. We use it here because we're gonna create multiple domains pointing to one server and that wouldn't be possible without Nginx. We also need it to install SSL certificates for your domain so you can use the app properly.
- **PostgreSQL**: The database we're going to use for Liphium. All of your data is stored in here.
- **Certbot**: Well, this is gonna be the way we make your Liphium town secure by installing SSL certificates for your domain. It's completely free and really easy to use. I even donated 10€ because it's just awesome. Getting certificates was really difficult in the past, and [you can donate, too](https://supporters.eff.org/donate/support-work-on-certbot).
- **Docker**: Docker is going to be used to install the actual Liphium server (called station) onto your server. We're going to install PostgreSQL using Docker as well.

With all of that explained, let's finally get to installing.

## Step 1: Getting your PostgreSQL server ready

Connect to your PostgreSQL server and let's create a few databases. We're going to need two databases: One for all of the accounts and one for all of the chat messages. You can create those two by typing `CREATE DATABASE chat; CREATE DATABASE main;`. Different names can also be used, just be sure to use the correct ones later.

## Step 2: Creating the actual Liphium town

**1.** Alright, now it's time for you to venture out by yourself for the first time. And that's to [installing Docker](https://docs.docker.com/engine/install/). They have amazing documentation over there and you should hopefully be able to get it running your machine. Complete it until the end where they tell you to run the hello-world image and when you've done all of that, you can come back here.

**2.** If you haven't done so already, it's a good time to make a folder for all of the files for your Liphium town now (like /home/liphium or something). But now, let's pull the official Liphium image from Docker Hub: `docker pull liphium/chat:latest`.

**3.** Before we start getting into the configuration file, it's a good time to actually get your domain records set up. They will sometimes take a long time to be applied and if we're lucky they will already be ready when we get to actually connecting to the town.

| Type | Name   | Content      | Description                                                  |
| ---- | ------ | ------------ | ------------------------------------------------------------ |
| A    | main   | YOUR_IP_HERE | The domain of the main instance (will be entered in Liphium) |
| A    | chat   | YOUR_IP_HERE | The domain of your chat server                               |
| A    | spaces | YOUR_IP_HERE | The domain of your Spaces server                             |

**4.** Now let's create the configuration/environment file for your Liphium town. To do this, please follow our [config setup guide](./config-setup.md#creating-a-new-configuration) and come back here once you finished creating the config.

**5.** If you chose local storage for your files, please create a folder called `files` in the current directory. All of your files will go into there.

**6.** Now run the docker container by using the command below. Make sure to replace `config.env` with the name of your environment/configuration file (or leave it alone if you called yours `config.env`).

```sh
docker run --env-file config.env -p 127.0.0.1:4002:4002 -p 127.0.0.1:4001:4001 -p 127.0.0.1:4000:4000 -v ./files:/home liphium/chat
```

**7.** Alright, after running the container once, it should print out that you should set `TC_PUBLIC_KEY` and `TC_PRIVATE_KEY`. Take the thing the container printed out and just add it to the bottom of `config.env`. Make sure to set both of those environment/configuration variables from the same run of Liphium. Otherwise they are not going to work together. Also make sure you remove the all the spaces and the double quotes. It should look something like this: `TC_PUBLIC_KEY=value`. `value` should not be surrounded by ", otherwise starting your town will not work. Do the same for `TC_PRIVATE_KEY` as well.

**8.** Now, for the last time, run the container again. It's going to ask you to set the `SYSTEM_UUID` environment variable to the thing it prints out. Also give it the same treatment as above and make it look like this: `SYSTEM_UUID=value`. `value` should again not be surrounded by ", otherwise station will not work.

**9.** Your setup of the main Liphium town is now complete. You should now be able to just run the thing and it should already be connecting to the database. The only thing that shouldn't work is grabbing the server public key. But we're going to fix that now.

## Step 3: Exposing your town to the internet with Nginx

**1.** First, you of course need to install Nginx. If you are on Ubuntu or Debian, you can just run the command below. If not, you might have to look up a tutorial on how to do it or just use the package your package manager provides (if it exists).

```sh
apt install nginx
```

**2.** After installing Nginx, we're going to need to set up a few sites. There should already be a directory called `/etc/nginx/sites-available`, but if it is not there, just create it. Enter this directory.

**3.** Now you're going to need a few configurations again. Create the files in the `/etc/nginx/sites-available` folder as specified below:

File name: `liphium-main`

```sh
server {
  server_name main.example.com;

  location /v1/account/files/upload {
    client_max_body_size 10m; # Change this based on your preferences (10m = 10 megabytes)
    proxy_pass http://localhost:4000;
  }

  location / {
     proxy_pass http://localhost:4000;
  }
}
```

File name: `liphium-chat`

```sh
server {
    server_name chat.example.com;

    # Live share subscribe
    location /liveshare/subscribe {
        proxy_http_version 1.1;

        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;
        proxy_buffering off;
        proxy_cache off;

        proxy_set_header Connection "keep-alive";
        chunked_transfer_encoding off;

        proxy_pass http://localhost:4001/liveshare/subscribe;
    }

    # WebSocket endpoint
    location /gateway {
        proxy_http_version 1.1;

        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://localhost:4001/gateway;
    }

    # Every other endpoint
    location / {
        proxy_pass http://localhost:4001;
    }
}
```

File name: `liphium-spaces`

```sh
server {
    server_name spaces.example.com;

    # The WebSocket gateway
    location /gateway {
        proxy_http_version 1.1;

        proxy_connect_timeout 7d;
        proxy_send_timeout 7d;
        proxy_read_timeout 7d;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_pass http://localhost:4002/gateway;
    }

    location / {
        proxy_pass http://localhost:4002;
    }
}
```

**4.** Change the domains in the configurations (behind `server_name`) you just downloaded from me to reflect your domain setup (for example: main.example.com -> main.liphium.com). Make sure to not forget the semicolon at the end!

**5.** Let's make Nginx actually use those configurations. To do that just run the commands below to create links to the files in the `/etc/nginx/sites-enabled` folder.

```sh
ln -s /etc/nginx/sites-available/liphium-main /etc/nginx/sites-enabled/liphium-main
ln -s /etc/nginx/sites-available/liphium-chat /etc/nginx/sites-enabled/liphium-chat
ln -s /etc/nginx/sites-available/liphium-spaces /etc/nginx/sites-enabled/liphium-spaces
```

**6.** Run `nginx -t` to validate that the configurations work properly. If they don't, do what the command tells you to fix the configurations up bit by bit.

**7.** You can now restart Nginx by typing `systemctl restart nginx`. You should now have the configurations loaded.

**8.** Try to go to one of your domains, like `http://main.example.com` (replaced with your domain) and see if there is a nginx error there. If there is, everything worked. If there isn't, everything could still be working fine and your domains could just not be updated yet. As I specified above, that can take up to 48 hours.

## Step 4: Adding SSL certificates to your town using Certbot

**1.** For Certbot to work, there are actually two things you need to install. You can follow [this guide](https://certbot.eff.org/instructions?ws=nginx&os=snap) from the official website. **Please follow their guide until step 5**.

**2.** To secure your town, we first need to make sure your domains are already updated and already redirect to the correct server. To verify if they are already set up correctly, go to `main.example.com` (replaced with your domain) and check if there is some sort of error from nginx. If it's not there yet, you'll have to wait a little bit before getting your certificates. This is because your domain isn't pointing to the server yet. This can take up to 48 hours to happen in some cases. So if this happened to you just be patient for now, I know this sucks, but it can happen.

**3.** When you did see the error from nginx, you can now just run `certbot --nginx` to apply the certificates. The CLI will ask your for a few things. For example, they'll ask you to enter your email address. Please do, so you are always notified if something is wrong with your certificates. When they finally ask which domains you want to secure, just leave the field blank to select all of them. Certbot will then do its magic and you are officially done with the setup.

## Step 5: Getting into your town

You can now use any of the official client apps to connect to your Liphium town by using the main domain (the one you replaced `main.example.com` with until now). **Make sure to not enter https:// or http:// in front of the domain**.

You will then have to create an account using a email address, a password and a username. After entering your email, you will be asked for an invite. This is a token required for anyone wanting to create an account in your town. Each invite can only be used once and the value you set `SYSTEM_UUID` to in your configuration file is an invite station creates automatically. Use that one to create your account, you will automatically get admin permissions as well as long as you are the first account created.

If you want to take a look at the settings of your town, you can find all of that under the "Your town" category when you go to the settings in the Liphium app. If you want to invite other people to your town, you can go to Settings -> Invites -> Generate invite to create invites for other people to be able to create accounts in your town. I hope you have a lot of fun with Liphium, and wish you a nice rest of your day!
