# Docker

So you want to make your own town, eh? Well **if you've never touched a server you probably shouldn't attempt this** as servers (especially with Linux) are not a newbie-friendly environment. But with that out of the way or if you're seeking a challenge, let's get a fresh new town onto your server!

Also before we get started, you can always message me on Discord if you have any kind of problem with the installation or have some kind of other question. You can get to the Liphium Discord server through the invite in the navigation bar or menu at the top of the page.

## Requirements

Please make sure you have everything you need installed before installing Liphium because otherwise it can be quite a bit of pain to get everything set up.

- A domain that you will [never change or sell](docs/general/faq/#why-do-i-need-a-domain-for-my-town) (read the linked FAQ question for more information).
- A server running Linux.
- The latest version of Docker. You can learn how install it [here](https://docs.docker.com/engine/install/).
- A mail server for Liphium to send you emails with SMTP credentials (it is required or registration will fail when people try to make an account).
- A PostgreSQL server that you can connect to.
- Decent skills with Docker (there might be issues).

## Tools we will use

Alright, one more thing before we get started on the installation. I just want you to know what we are going to use so everything is clear. I'll have instructions on how to install Nginx and Certbot later. Although I just link you to their websites, respectively.

- **Nginx**: Nginx is a reverse proxy for web requests that we'll use to expose your Liphium town to the outside world. We use it here because we're gonna create multiple domains pointing to the your server and it wouldn't be possible without nginx. We also need it to install SSL certificates for your domain so you can use the app properly.
- **PostgreSQL**: The database we're going to use for Liphium. All of your data is stored in here.
- **Certbot**: Well, this is gonna be the way we make your Liphium town secure by installing SSL certificates for your domain. It's completely free and really easy to use. I even donated 10€ because it's just awesome. [You can too](https://supporters.eff.org/donate/support-work-on-certbot).
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

# TODO: Finish tutorial here

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

## Step 4: Adding SSL certificates to your town using Certbot

**1.** For Certbot to work, there are actually two things you need to install. You can follow [this guide](https://certbot.eff.org/instructions?ws=nginx&os=snap) from the official website. **Please follow their guide until step 5**.

**2.** To secure your town, we first need to make sure your domains are already updated and already redirect to the correct server. To verify if they are already set up correctly, go to `main.yourdomain.com` (or whatever you used) and check if there is some sort of error from nginx. If it's not there yet, you'll have to wait a little bit before getting your certificates. This is because your domain isn't pointing to the server yet. This can take up to 48 hours to happen in some cases. So if this happened to you just be patient for now, I know this sucks, but it can happen.

**3.** When you did see the error from nginx, you can now just run `certbot --nginx` to apply the certificates. The CLI will ask your for a few things. For example, they'll ask you to enter your email address. Please do, so you are always notified if something is wrong with your certificates. When they finally ask which domains you want to secure, just leave the field blank to select all of them. Certbot will then do its magic and you are officially done with the setup.

## Step 5: Getting into your town

When you now connect to your Liphium town for the first time with any client app, you can enter the `SYSTEM_UUID` environment variable as the invite. This is an invite created to make sure an account can be created.

If you just created the first account on your Liphium town, it will automatically have admin permissions. You can now go to Settings -> Invites -> Generate invite to invite other people to your town.

## We're finally done

I know it took a long time to get Liphium installed. Thanks for following this guide until now and I hope you didn't run into too many issues. I'm tired now and I'm gonna go watch a few episodes of Shikanoko now. This thing took like 3-4 hours to write and I hope it helped.
