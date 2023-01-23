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

Minecraft servers expect requests from a Minecraft client on [port 25565, via TCP/UDP](https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server). Geyser proxy servers communicate over [port 19132, via UDP](https://wiki.geysermc.org/geyser/setup/). Therefore, we need to make those ports available from our server via a technique called [_port forwarding_](https://wiki.geysermc.org/geyser/setup/). The approach varies widely across different router makes and firmwares; you can find [info for your router on portforward.com](https://portforward.com/router.htm).

First, set up a static IP for your Orange Pi on your router. For my Sonic SmartRG router:
* Go to the router admin page (192.168.42.1/admin)
* Navigate to Advanced Setup > LAN and, under "Static IP Lease List", click Add entries
* Switch over to your Orange Pi console and get its MAC address via [`ip link`](https://itsfoss.com/find-mac-address-linux/)
* Choose a static IP host ID (the fourth number in the address, after the network ID configured by your router / LAN). E.g. `123` is the host ID in `192.168.42.123`.

Now that your router assigns the same, static IP to your Orange Pi every time it connects to your network, you can find the external (or "public") IP for your Orange Pi. This is the IP that Minecraft clients will use to connect to your server. Switch back over to your Orange Pi console and type `curl https://ipinfo.io/ip`. Copy this down -- this is your external IP.

Then, forward the necessary ports. For my Sonic SmartRG router, I used [this guide on portforward.com](https://portforward.com/smartrg/sr516ac/). In short:
* Go to the router admin page (192.168.42.1/admin)
* Navigate to Advanced Setup > NAT > Virtual Servers and click Add
* Since my Orange Pi is hardwired to my router, I selected `ipoe_eth4/eth4.1` for the "Use interface" dropdown
* Create one entry for Minecraft server requests: 25565 for external and internal port start and end, and TCP/UDP for protocol
* Create one entry for Geyser server requests: 19132 for external and internal port start and end, and UDP for protocol

Now you can check that your server is now accessible from the outside world. Start your server from the Orange Pi console as described in the above section, and wait for it to complete startup. You can then use a tool like [yougetsignal's port checker](https://www.yougetsignal.com/tools/open-ports/) for this. Note that, for a correct setup, yougetsignal shows port 25565 (minecraft) as open, but 19132 (geyser) as closed. 

Finally, set up your server on a Minecraft Bedrock client (like the [iOS version](https://apps.apple.com/us/app/minecraft/id479516143)). Our favorite setup tutorial [outlines the steps here](https://pimylifeup.com/ubuntu-minecraft-bedrock-server/#connecting-to-the-server), but the TL;DR:
* Log into your Microsoft/Minecraft account
* Open the Servers tab from the startup screen
* Scroll to the bottom and tap Add Server
* Give the server a name (that will display only on this client), type in the external IP we found above, and specify port 25565
* Tap the server you just configured to proceed to play!



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
* [Minecraft wiki: Port forwarding](https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server#Port_forwarding)


## Resources + downloadables
* [balenaEtcher flash OS to MicroSD](https://www.balena.io/etcher/)
* [PaperMC](https://papermc.io/) Minecraft server
* [GeyserMC plugin](https://geysermc.org/)
* [Floodgate plugin](https://github.com/GeyserMC/Floodgate/)
* [Minecraft Server Tools](https://mctools.org/)
* [Server status checker](https://mcsrvstat.us/)
* [Open port verifier](https://www.yougetsignal.com/tools/open-ports/)


	
## WIP cruft	

	
to me, Gloriane
Start the Minecraft Bedrock Server at Boot on Ubuntu
Creating a startup + shutdown script using `screen` and a service to manage the server, and start automatically on boot

also:
Ubuntu startup script w/ extra server config features
Create Ubuntu startup script


consider using Aikar's flags to optimize performance

just noticed that Geyser has a bunch of its own configs in plugins/Geyser-Spigot/config.yml, will want to update some e.g. passthrough-motd
per Geyser/Floodgate instructions, i set `auth-type: floodgate` in geyser's config.yml
more details on geyser config.yml here

	
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

	
