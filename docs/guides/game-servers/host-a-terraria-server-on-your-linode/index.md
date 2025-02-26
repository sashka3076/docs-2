---
slug: host-a-terraria-server-on-your-linode
author:
  name: Linode Community
  email: docs@linode.com
description: 'In this guide, you will learn how to install and configure Terraria, a two-dimensional sandbox game similar to Minecraft, on a Linode.'
og_description: 'In this guide, you will learn how to install and configure Terraria, a two-dimensional sandbox game similar to Minecraft, on a Linode.'
keywords: ["terraria", "steam", "minecraft", "gaming"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
published: 2015-12-21
modified: 2021-06-29
modified_by:
  name: Linode
title: 'How to Setup a Terraria Linux Server'
contributor:
  name: Tyler Langlois
  link: https://github.com/tylerjl
external_resources:
 - '[Terraria Wiki](http://terraria.gamepedia.com/Terraria_Wiki)'
 - '[Terraria Wiki: Server](http://terraria.gamepedia.com/Server)'
 - '[Terraria Wiki: Setting up a Terraria Server](http://terraria.gamepedia.com/Guide:Setting_up_a_Terraria_server)'
aliases: ['/game-servers/host-a-terraria-server-on-your-linode/','/applications/game-servers/host-a-terraria-server-on-your-linode/']
dedicated_cpu_link: true
---

![Hosta a Terraria Server on Your Linode](terraria-server.png "Hosta a Terraria Server on Your Linode")

[Terraria](https://terraria.org/) is a two-dimensional sandbox game, similar to [Minecraft](https://minecraft.net/), which allows players to explore, build, and battle in an open world.

## Does Terraria support Linux?

In 2015, the Terraria developers announced [support for Linux](https://terraria.org/#/news/terraria-1-3-0-8-now-for-mac-linux-too-), which means that players can host their own standalone Terraria servers.

This guide outlines the steps required to run a Terraria server for yourself and others to play on. These steps are compatible with any Linux distribution that uses [systemd](https://www.freedesktop.org/wiki/Software/systemd/). This includes recent versions of CentOS, Debian and Ubuntu, Arch Linux and Fedora.

Due to Terraria's system requirements, a Linode with at least two CPU cores and adequate RAM is required. For this reason, we recommend using our [4GB plan or higher](https://www.linode.com/pricing) when following this guide. If your Linode does not meet Terraria's minimum requirements, the process will crash intermittently.

## Before You Begin

1.  If you have not already done so, create a Linode account and Compute Instance. See our [Getting Started with Linode](/docs/guides/getting-started/) and [Creating a Compute Instance](/docs/guides/creating-a-compute-instance/) guides.

1.  Follow our [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/) guide to update your system. You may also wish to set the timezone, configure your hostname, create a limited user account, and harden SSH access.

## Configure a Firewall for Terraria

{{< note >}}
Terraria only uses IPv4 and does not use IPv6.
{{< /note >}}

### Firewalld

Firewalld is the default iptables controller in CentOS 7+ and Fedora. See our [guide on using firewalld](/docs/guides/introduction-to-firewalld-on-centos/) for more information.

1.  Enable and start firewalld:

        sudo systemctl enable firewalld && sudo systemctl start firewalld

    You should be using the public zone by default. Verify with:

        sudo firewall-cmd --get-active-zones

2.  Create a firewalld service file for Terraria:

    {{< file "/etc/firewalld/services/terraria.xml" aconf >}}
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Terraria</short>
  <description>Open TCP port 7777 for incoming Terraria client connections.</description>
  <port protocol="tcp" port="7777"/>
</service>

{{< /file >}}


3.  Enable the firewalld service, reload firewalld and verify that the Terraria service is being used:

        sudo firewall-cmd --zone=public --permanent --add-service=terraria
        sudo firewall-cmd --reload
        sudo firewall-cmd --zone=public --permanent --list-services

    The output of the last command should be similar to:

        dhcpv6-client ssh terraria

### UFW

[UFW (Uncomplicated Firewall)](/docs/security/firewalls/configure-firewall-with-ufw/) is an iptables controller packaged with Ubuntu, but it's not installed in Debian by default.

1.  If needed, install UFW:

        sudo apt install ufw

2.  Add SSH and a rule for Terraria. It's important you add rules before enabling UFW. If you don't, you'll terminate your SSH session and will need to access your Linode using [Lish](/docs/guides/using-the-lish-console/):

        sudo ufw allow ssh
        sudo ufw allow 7777/tcp

3.  After your rules are added, enable UFW. Next, remove the Terraria rule for IPv6 since it's not needed:

        sudo ufw enable
        sudo ufw delete 4

    {{< note >}}
The second command in this step, `sudo ufw delete 4` references the fourth rule in your UFW ruleset. If you need to configure additional rules for different services, adjust this as necessary. You can see your UFW ruleset with `sudo ufw status` to make sure you're removing the correct rule.
{{< /note >}}

### iptables

To manually configure iptables without using a controller, see our [iptables guide](/docs/guides/control-network-traffic-with-iptables/) for a general ruleset.

1.  You'll also want to add the rule below for Terraria:

        sudo iptables -A INPUT -p tcp --dport 7777 -j ACCEPT

2.  Verify with:

        sudo iptables -vL


## Install and Configure Terraria on Linux

1.  Download the Terraria tarball. You'll need to check [Terraria's website](https://terraria.fandom.com/wiki/Server#Downloads) for the current release version. Right-click and copy the link to use with `curl` or `wget`. We'll use 1.4.2.3 as an example in this guide:

        sudo curl -O https://terraria.org/api/download/pc-dedicated-server/terraria-server-1423.zip

    {{< note >}}
Before you install Terraria, be sure the version you download is the same as the clients that will be connecting to it.
{{< /note >}}

2. You will need the `unzip` utility to decompress the .zip file. Install it using your distribution's package manager:

    **Debian/Ubuntu:**

        sudo apt install unzip

    **CentOS:**

        sudo yum install unzip

3.  Extract the archive:

        sudo unzip terraria-server-*

4. The Terraria Server file will contain an executable that must have execute permissions set to run the server. Enter the following command to do this:

        sudo chmod +x ~/1423/Linux/TerrariaServer.bin.x86_64

1.  Terraria can be set up with a server configuration file that you can edit with options such as automatic world creation, server passwords, difficulty, [and other options](http://terraria.gamepedia.com/Server#serverconfig). You can create a basic server configuration file now to configure your server before it launches.

The options below will automatically create and serve the world `MyWorld` when the game server starts up. Note that you should change `MyWorld` to a world name of your choice.

{{< file "/home/example_user/1423/Linux/serverconfig.txt" ini >}}
world=/srv/terraria/Worlds/MyWorld.wld
autocreate=1
worldname=MyWorld
worldpath=/srv/terraria/Worlds
{{< /file >}}


## Managing the Terraria Service

### Screen

Terraria runs an interactive console as part of its server process. While useful, accessing this console can be challenging when operating game servers under service managers. The problem can be solved by running Terraria in a [screen session](https://www.gnu.org/software/screen/) that will enable you to send arbitrary commands to the listening admin console within Screen.

Install Screen with the system's package manager:

**CentOS:**

    sudo yum install screen

**Debian/Ubuntu:**

    sudo apt install screen

### systemd

It's useful to have an automated way to start, stop, and bring up Terraria on boot. This is important if the system restarts unexpectedly.

Create the following file to define the `terraria` systemd service, replacing `example_user` with your limited username:

{{< file "/etc/systemd/system/terraria.service" ini >}}
[Unit]
Description=server daemon for terraria

[Service]
Type=forking
User=root
KillMode=none
ExecStart=/usr/bin/screen -dmS terraria /bin/bash -c "/home/example_user/1423/Linux/TerrariaServer.bin.x86_64 -config /home/example_user/1423/Linux/serverconfig.txt"
ExecStop=/usr/local/bin/terrariad exit

[Install]
WantedBy=multi-user.target

{{< /file >}}


*   **ExecStart** instructs systemd to spawn a screen session containing the 64-bit `TerrariaServer` binary, which starts the daemon. `KillMode=none` is used to ensure that systemd does not prematurely kill the server before it has had a chance to save and shut down gracefully.

*   **ExecStop** calls a script to send the `exit` command to Terraria, which tell the server to ensure that the world is saved before shutting down. In the next section, we'll create a script which will send the necessary commands to the running Terraria server.

{{< caution >}}
This script is intended to save your world in the event that you reboot the operating system within the Linode. It is **not** intended to save your progress if you reboot your Linode from the Linode Manager. If you must reboot your Linode, first stop the Terraria service using `sudo systemctl stop terraria`. This will save your world, and then you can reboot from the Linode Manager.
{{< /caution >}}

### Create a Script for Basic Terraria Administration

The Terraria administration script needs two primary functions:

*   Attaching to the running screen session, which offers a helpful administration console.
*   The ability to broadcast input into the screen session so the script can be run to save the world, exit the server, etc.

1.  Create a `terrariad` file, enter the following script, then save and close:

    {{< file "/usr/local/bin/terrariad" >}}
#!/usr/bin/env bash

send="`printf \"$*\r\"`"
attach="screen -r terraria"
inject="screen -S terraria -X stuff $send"

if [ "$1" = "attach" ] ; then cmd="$attach" ; else cmd="$inject" ; fi

if [ "`stat -c '%u' /var/run/screen/S-terraria/`" = "$UID" ]
then
    $cmd
else
    su - root -c "$cmd"
fi
{{< /file >}}


2.  Verify that you can execute the script:

        sudo chmod +x /usr/local/bin/terrariad

This script permits you to both:

*  Attach to the console for direct administration, and
*  Send the console commands like `save` or `exit` while it's running without needing to attach at all (useful when services like systemd need to send server commands).

{{< note >}}
Throughout the rest of this guide, you may encounter "command not found" errors when running the `terrariad` command. This may result from the directory `/usr/local/bin/` not being found in the `$PATH` when running sudo commands, which can occur with some Linux distributions. You can work around this problem by calling the script with the full path. For example, instead of running `sudo terrariad attach`, use `sudo /usr/local/bin/terrariad attach`.
{{< /note >}}

## Running Terraria Linux Server

### Start and Enable the Terraria Server

Now that the game server is installed, the scripts are written, and the service is ready, the server can be started with a single command:

    sudo systemctl start terraria

The first time you run the server, it will generate the world defined earlier. This will take a while, so give it time before trying to connect. To watch the world generation progress, use:

    sudo terrariad attach

In addition to starting and stopping the `terraria` service, systemd can also use the service file created earlier to automatically start Terraria on boot.

To enable the service at startup:

    sudo systemctl enable terraria

If the operating system is restarted for any reason, Terraria will launch itself on reboot.

### Server Status

To check if the server is running, use the command:

    sudo systemctl status terraria

The output should be similar to:

{{< output >}}
● terraria.service
   Loaded: loaded (/etc/systemd/system/terraria.service; disabled)
   Active: active (running) since Tue 2017-03-07 17:37:03 UTC; 7s ago
  Process: 31143 ExecStart=/usr/bin/screen -dmS terraria /bin/bash -c /opt/terraria/TerrariaServer.bin.x86_64 -config /opt/terraria/serverconfig.txt (code=exited, status=0/SUCCESS)
 Main PID: 31144 (screen)
   CGroup: /system.slice/terraria.service
           ├─31144 /usr/bin/SCREEN -dmS terraria /bin/bash -c /opt/terraria/TerrariaServer.bin.x86_64 -config /opt/terraria/serverconfig.txt
           └─31145 /opt/terraria/TerrariaServer.bin.x86_64 -config /opt/terraria/serverconfig.txt
{{< /output >}}

### Stop the Server

If you ever need to shut down Terraria, use the following command to save the world and shut down the game server:

    sudo systemctl stop terraria

### Attach to the Console

In the course of running your server, you may need to attach to the console to do things like kick players or change the message of the day (MOTD). To enter the Terraria server console with the `terrariad` script use:

    sudo terrariad attach

Type `help` to get a list of commands. Once you're done, use the keyboard shortcut **CTRL+A** then **D** to detach from the screen session and leave it running in the background. More keyboard shortcuts for Screen can be found in the [Screen default key bindings documentation](http://www.gnu.org/software/screen/manual/html_node/Default-Key-Bindings.html#Default-Key-Bindings).

## How do I host a Terraria Server for free?

To host a Terraria server for free, choose a free plan first and then choose the country where you are planning to host this Terraria server.

## How do I find my server port for Terraria?

Default server port for Terraria is 7777. But if you have port forwarding enabled, your server port is your IP address + Colin + the port number.
