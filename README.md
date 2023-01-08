# Minecraft Server on Orange Pi
Installing and managing a Minecraft server on Ubuntu

## Orange Pi setup

I wanted to make a place for my kid to play Minecraft with his pals. I also wanted to try playing with a Raspberry Pi.

I quickly found that Raspberry Pis are near impossible to get a hold of without paying inflated prices, so I ended up with an Orange Pi 4B, a decently-priced Raspberry Pi clone. It'll run you around [$130 on Amazon](https://www.amazon.com/Orange-Pi-Rockchip-Supported-Computer/dp/B0B9BXJT8V/) with 4GB RAM and 16GB embedded flash storage, and a shell + power supply.

The [Orange Pi documentation](https://drive.google.com/drive/folders/1SCljqsaA6_KTdAxuqCBW38B6MWWKCRr-) is decent, and they have [OS images ready to download](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-pi-4-B.html). [This page](https://www.androidpimp.com/embedded/orange-pi-4-review/) helped me understand where the included heatsinks are supposed to be attached (spoiler: the CPU and the flash memory, not the RAM).

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

## Installing Minecraft server
set up user (`mcsesrver`)
java headless
bedrock --> java + geyser + floodgate
spigot --> paper
port forwarding (virtual server) - minecraft 25565 + geyser 19132
setting static IP (+ checking w/ `curl https://ipinfo.io/ip`)

## Tightening up
start + stop scripts, w/ screen
systemd setup
server maintenance:
    ssh mcserver@192.168.42.42
    screen -r paper-server

## Resources + Links



	
## WIP cruft	
to me, Gloriane
video instructions for flashing ubuntu image to TF card + connecting via ssh on local network (get IP from router page once connected via CAT cable to router)
default login: root / orangepi
then `apt-get update` to update installed software packages

official Orange Pi 4B Ubuntu image
 2 Attachments  •  Scanned by Gmail
Preview YouTube video Orange Pi Zero Official Ubuntu Image Install and Setup Guide
Eric Socolofsky <eric@transmote.com>
	
Tue, Jan 3, 5:22 PM (4 days ago)
	
to me, Gloriane
other misc info/links

    Orange Pi 4 docs
    Orange Pi 4 Review w/ setup instructions + photos
    Official Ubuntu Orange Pi 4B image
    Installing Ubuntu on Orange Pi 3
    Orange Pi 4B user manual

Also, to get IP when hardwired (CAT cable) to sonic router, login to 192.168.42.1/admin (user: admin / pass: key from back of router), then navigate to Device Info > DHCP and look for IP listed for hostname 'orangepi4'. Currently it's 192.168.42.35, so I login via ssh root@192.168.42.35 w/ password: orangepi (root/orangepi are default for Orange Pi Ubuntu image)

next steps:
* possible (necessary?) to set static IP?
* how to use `screen` to keep connection from laptop open?
* ensure everything is updated to latest version on ubuntu install
* install bedrock server
* document everything
 2 Attachments  •  Scanned by Gmail
Eric Socolofsky <eric@transmote.com>
	
Tue, Jan 3, 7:45 PM (4 days ago)
	
to me, Gloriane
support & download page for Orange Pi 4B (incl OS images)

looks like i can't install a Bedrock server directly on an ARM processor (e.g. Orange Pi, Raspberry Pi, etc).
This reddit thread suggests installing a Java server and adding GeyserMC to enable Bedrock clients to connect to a Java server. (It also suggests installing Floodgate; its docs suggest that Bedrock connections only work for users with a Java edition account, and Floodgate removes this requirement.)

this post offers a walkthrough to install Java server on ARM, though it uses CentOS...will try on my Ubuntu install. It's just Java, right?
ok, ubuntu is a bit different. here's a post on installing java on ubuntu, but the TL;DR:

sudo apt install default-jdk

(or maybe default-jre suffices? but i installed JDK.)

ok, so ubuntu default is Java 11. Minecraft Server distro was compiled w/ Java 17, so instead (following this site) i'm trying

apt install openjdk-17-jre

oh and look, here's info on server requirements and on setting up a server, straight from the source.

next TODOs:

    rename user from `bedrock` to `mcserver`, since it's not a bedrock server after all >.<
    continue w/ configuration, like from this bedrock tutorial (note we're not installing bedrock anymore!!) and details from minecraft wiki

Eric Socolofsky <eric@transmote.com>
	
Tue, Jan 3, 7:46 PM (4 days ago)
	
to me, Gloriane
also -- do i need to configure open ports / static IP so others can connect? (see firewall ports section on this tutorial)
Eric Socolofsky <eric@transmote.com>
	
Tue, Jan 3, 7:49 PM (4 days ago)
	
to me, Gloriane
looks like there's info on IP config here: https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server#Port_forwarding
Eric Socolofsky <eric@transmote.com>
	
Tue, Jan 3, 8:42 PM (4 days ago)
	
to me, Gloriane
ok -- static IP is 135.180.65.19

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

    Tried to verify port is open (forwarded) on static IP via https://www.yougetsignal.com/tools/open-ports/, but 135.180.65.19:25565 is showing as closed...
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
orangepi MAC address: 6e:7c:9a:76:0f:89
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

	
