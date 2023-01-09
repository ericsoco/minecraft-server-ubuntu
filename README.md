# Minecraft Server on Orange Pi
Installing and managing a Minecraft server on Ubuntu


## Orange Pi: Hardware

I wanted to make a place for my kid to play Minecraft with his pals. I also wanted to try playing with a Raspberry Pi.

I quickly found that Raspberry Pis are near impossible to get a hold of without paying inflated prices, so I ended up with an Orange Pi 4B, a decently-priced Raspberry Pi clone. It'll run you around [$130 on Amazon](https://www.amazon.com/Orange-Pi-Rockchip-Supported-Computer/dp/B0B9BXJT8V/) with 4GB RAM and 16GB embedded flash storage, and a shell + power supply.

The [Orange Pi documentation](https://drive.google.com/drive/folders/1SCljqsaA6_KTdAxuqCBW38B6MWWKCRr-) is decent, and they have [OS images ready to download](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-pi-4-B.html). [This page](https://www.androidpimp.com/embedded/orange-pi-4-review/) helped me understand where the included heatsinks are supposed to be attached (spoiler: the CPU and the flash memory, not the RAM).


## Orange Pi: Ubuntu OS

My kid uses the "pocket" (mobile/tablet) version of Minecraft, [which is Bedrock, not Java](https://www.minecraft.net/en-us/article/java-or-bedrock-edition). So, I went with the Orange Pi Ubuntu image, as I found [this excellent tutorial](https://pimylifeup.com/ubuntu-minecraft-bedrock-server/) on installing Minecraft Bedrock on Ubuntu.

To get the OS onto the Orange Pi, you'll need an 8GB+ TF / MicroSD card. (They're interchangeable / basically the same thing.) Download the [Ubuntu Orange Pi 4B image](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-pi-4-B.html) on your PC (I'm using a Mac), unzip, and flash to the microSD. Vincent Stevenson has a [great video tutorial](https://www.youtube.com/watch?v=2oDdlho_WaQ) outlining the steps; [this slightly older Ubuntu/Orange Pi 3 tutorial](https://longervision.github.io/2019/06/05/SBCs/ARM/orange-pi-3-official/) has some great detail as well. I used [balenaEtcher](https://www.balena.io/etcher/) to flash the `.img` file in the downloaded zip file, then put the microSD into the Orange Pi.

I connected the Orange Pi to an open LAN port on my router with an ethernet cable, plugged in the power supply, and waited about 30 seconds. Then I opened the admin page for my router (http://192.168.42.1/admin for my [SmartRG SR515AC](https://www.lmi.net/wp-content/uploads/Gateway_User_Manual_v3_5.pdf), thanks [Sonic](https://www.sonic.com/)!) and found my Orange Pi's IP address in the Device Info > DHCP tab. Finally, I `ssh`d to the Orange Pi from my PC (on the same LAN of course), using the default `root`/`orangepi` password in the Orange Pi Ubuntu image: `ssh root@192.168.42.XX`. 

We're in!

```
  ___  ____  _   _  _   
 / _ \|  _ \(_) | || |  
| | | | |_) | | | || |_ 
| |_| |  __/| | |__   _|
 \___/|_|   |_|    |_|  
                        
Welcome to Orange Pi 3.0.6 Focal with Linux 5.10.43
```


## Orange Pi: Prep for server install
To wrap up the setup, we need to take a few more steps to prepare the Ubuntu install.

Update Ubuntu dependencies:
```
sudo apt update
```

Download needed utils and Java Runtime Environment (headless, since we don't need a GUI to run the server):
```
sudo apt install curl wget unzip grep screen -y
sudo apt-get install git openjdk-17-jre-headless
```

Create an admin user:
```
sudo usermod -a -G mcserver $USER
logout
```

Log into the new user and create a folder for the server install:
```
ssh mcserver@192.168.42.XX
mkdir minecraft-server
```


## Installing Minecraft server

When I found this [Bedrock / Ubuntu tutorial](https://pimylifeup.com/ubuntu-minecraft-bedrock-server/), I figured that would be the way to go. But [I quickly realized](https://www.reddit.com/r/admincraft/comments/tt7ig9/bedrock_server_for_arm/) that the Bedrock server doesn't have an version compiled to run on ARM processors, which is what we have on the Orange Pi. Uh-oh.

The TL;DR: Run a [PaperMC](https://papermc.io/) (Java) server with the [GeyserMC plugin](https://geysermc.org/) installed to enable Bedrock clients to connect, and with the [Floodgate plugin](https://github.com/GeyserMC/Floodgate/) to allow those without a paid Java account to connect. Don't bother with the vanilla Java server, nor with [Spigot](https://www.spigotmc.org/) -- [Paper improves on Spigot in a number of ways](https://madelinemiller.dev/blog/paper-vs-spigot/).

Install and run Paper: copy the URL of the latest build on the [Paper downloads page](https://papermc.io/downloads), download it to your Orange Pi, and run it once to get things set up:
```
# replace with the URL from the Paper downloads page
wget https://api.papermc.io/v2/projects/paper/versions/1.19.3/builds/372/downloads/paper-1.19.3-372.jar
java -Xms2G -Xmx2G -jar paper-1.19.3-372.jar --nogui
```

It will take a while to initialize the first time; once you get to the server prompt, type `stop` to stop the server. After running it once (maybe twice? can't remember), a `plugins/` folder will be created adjacent to the `.jar`. Download the Spigot builds of [Geyser](https://ci.opencollab.dev/job/GeyserMC/job/Geyser/job/master/) and [Floodgate](https://ci.opencollab.dev/job/GeyserMC/job/Floodgate/job/master/) into it:
```
cd plugins
# replace with the URL from the Geyser and Floodgate downloads pages
wget https://ci.opencollab.dev/job/GeyserMC/job/Geyser/job/master/lastSuccessfulBuild/artifact/bootstrap/spigot/build/libs/Geyser-Spigot.jar
wget https://ci.opencollab.dev/job/GeyserMC/job/Floodgate/job/master/lastSuccessfulBuild/artifact/spigot/build/libs/floodgate-spigot.jar
```

Run the server one more time to pick up and auto-configure the plugins:
```
cd ../
java -Xms2G -Xmx2G -jar paper-1.19.3-372.jar --nogui
```

Your Minecraft Java server should now be set up and ready to accept Bedrock clients.


## Making your server visible to the outside world
You can now connect with your Bedrock client over your LAN, via the Friends tab. It will appear as "Geyser / Another Geyser server". If that's good enough for you, congratulations! However, the whole point for me was so my kid's friends in far-off places can play Minecraft with him. So, we need set a static IP and port forward the required ports.



port forwarding (virtual server) - minecraft 25565 + geyser 19132
setting static IP (+ checking w/ `curl https://ipinfo.io/ip`)


## Tightening up
server.properties
start + stop scripts, w/ screen
systemd setup
server maintenance:
    ssh mcserver@192.168.42.42
    screen -r paper-server
testing server status w/ [yougetsignal](https://www.yougetsignal.com/tools/open-ports/)

## Links + tutorials
* [Orange Pi documentation](https://drive.google.com/drive/folders/1SCljqsaA6_KTdAxuqCBW38B6MWWKCRr-)
* [Orange Pi OS images](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-pi-4-B.html)
* [Orange Pi 4 review + details](https://www.androidpimp.com/embedded/orange-pi-4-review/)
* [Installing Ubuntu on Orange Pi 4](https://www.youtube.com/watch?v=2oDdlho_WaQ)
* [Installing Ubuntu on Orange Pi 3 tutorial](https://longervision.github.io/2019/06/05/SBCs/ARM/orange-pi-3-official/)
* [Minecraft: Bedrock vs Java](https://www.minecraft.net/en-us/article/java-or-bedrock-edition)
* [Minecraft Bedrock on Ubuntu tutorial](https://pimylifeup.com/ubuntu-minecraft-bedrock-server/)
* [SmartRG SR515AC manual](https://www.lmi.net/wp-content/uploads/Gateway_User_Manual_v3_5.pdf)
* [Paper vs Spigot](https://madelinemiller.dev/blog/paper-vs-spigot/)
* [Minecraft wiki: setting up a server](https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server)


## Resources + downloadables
* [balenaEtcher flash OS to MicroSD](https://www.balena.io/etcher/)
* [PaperMC](https://papermc.io/) Minecraft server
* [GeyserMC plugin](https://geysermc.org/)
* [Floodgate plugin](https://github.com/GeyserMC/Floodgate/)
* [Minecraft Server Tools](https://mctools.org/)
* [Server status checker](https://mcsrvstat.us/)
* [Open port verifier](https://www.yougetsignal.com/tools/open-ports/)


	
## WIP cruft	

looks like there's info on IP config here: https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server#Port_forwarding

to set this up (on Sonic SmartRG router; used this guide):

    go to router admin page (192.168.42.1/admin)
     Advanced Setup > NAT > Virtual Servers, Add, followed instructions here (set TCP/UDP to 25565 for external and internal)
    finally, checked static IP with

    curl https://ipinfo.io/ip

Eric Socolofsky <eric@transmote.com>
	
Wed, Jan 4, 9:18 PM (3 days ago)
	
to me, Gloriane
Start the Minecraft Bedrock Server at Boot on Ubuntu
Creating a startup + shutdown script using `screen` and a service to manage the server, and start automatically on boot

also:
Ubuntu startup script w/ extra server config features
Create Ubuntu startup script

next up: get client to connect. some things to work through:

    Tried to verify port is open (forwarded) on static IP via https://www.yougetsignal.com/tools/open-ports/, but XXX.XXX.XX.XX:25565 is showing as closed...
    Enable Bedrock client to connect via GeyserMC

Eric Socolofsky <eric@transmote.com>
	
Wed, Jan 4, 9:44 PM (3 days ago)
	
to me, Gloriane
looks like the gist for Geyser (Bedrock) is:

    Install Geyser as a plugin on the server -- acts as a proxy that translates between Bedrock client and Java server
    May also need to install Floodgate as a plugin to enable players without a Java account (only a Bedrock account) to auth+connect
    Bedrock port is different than Java, will have to port forward 19132 UDP
    Looks like all of this requires a new Spigot (CraftBukkit) server jar, as described here (note there are instructions here for creating a screen-based server management script also)

open question: do i need to do something w/ firewalls to get the port forwarding to work?
Eric Socolofsky <eric@transmote.com>
	
Wed, Jan 4, 9:49 PM (3 days ago)
	
to me, Gloriane
side note, i kept seeing references to jre-headless which is probably more performant than a java install w/ a GUI (that I won't use), so i switched:

sudo apt autoremove git openjdk-17-jre
sudo apt-get install git openjdk-17-jre-headless
Eric Socolofsky <eric@transmote.com>
	
Thu, Jan 5, 9:52 PM (2 days ago)
	
to me, Gloriane
I've now tried three different server installs:

    vanilla Java (from Mojang/MS)
    Spigot
    Paper

Read that Paper is more modern + preferred to Spigot.

Paper install instructions
Adding plugins (note: /plugins folder doesn't exist until the server is run...twice?)

re port forwarding, looks like i need to configure ubuntu to use a static IP
https://ubuntu.com/server/docs/network-configuration (find "Static IP")

consider using Aikar's flags to optimize performance

just noticed that Geyser has a bunch of its own configs in plugins/Geyser-Spigot/config.yml, will want to update some e.g. passthrough-motd
per Geyser/Floodgate instructions, i set `auth-type: floodgate` in geyser's config.yml
more details on geyser config.yml here

...I think we're now all set up on the server side?
just need to figure out why the port forwarding isn't working, then start trying to connect w/ bedrock client (ipad).
Eric Socolofsky <eric@transmote.com>
	
Fri, Jan 6, 8:53 PM (23 hours ago)
	
to me, Gloriane
specified static IP for orangepi on router:
Advanced Setup > LAN > Static IP Lease List > Add entry
orangepi MAC address: XX:XX:XX:XX:XX:XX
static IP I chose: 192.168.42.42

this ^^ did it!
also reconfigured port forwarding ("Virtual Server") to have a single service with both Minecraft main and Geyser ports configured within it. Don't think that makes a difference? But documenting JIC.
Screenshot 2023-01-06 at 8.51.06 PM.png
Note also that yougetsignal shows port 25565 (minecraft) as open, but 19132 (geyser) as closed. However, I can add it at the bottom of Servers tab in minecraft (client, on ipad) just fine.
Eric Socolofsky <eric@transmote.com>
	
Fri, Jan 6, 9:50 PM (23 hours ago)
	
to me, Gloriane
last bits:

    set up script to run on boot, using screen
    look into server script i noted above on minecraft wiki
    document all this shit (post on GH?)

Eric Socolofsky <eric@transmote.com>
	
7:32 PM (1 hour ago)
	
to me, Gloriane
server tools like MOTD and allowlist creator
server status (online or not)

created a start-server.sh and stop-server.sh
these manage server thru a linux screen named `paper-server`
so to manager the server / get to its console, attach to the screen: `screen -r paper-server`

created minecraft-server.service, borrowing bits from:
https://blog.infoitech.co.uk/linux-systemd-service-unit-minecraft/
https://pimylifeup.com/ubuntu-minecraft-bedrock-server/#start-the-minecraft-bedrock-server-at-boot-on-ubuntu
https://minecraft.fandom.com/wiki/Tutorials/Ubuntu_startup_script

tested a hard reboot and the server is up

reminder, to get to the server console:

    ssh mcserver@192.168.42.42
    screen -r paper-server

	
