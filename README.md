btsyncctl
===========

[BitTorrent Sync](http://www.bittorrent.com/sync) is a powerful cross-platform file sharing application. Think of it as a decentralized version of Dropbox with no charges (it's free), no limits, and no middle-man.

btsyncctl is a simple bash script meant to automate the process of starting, stopping, and checking the status of the BitTorrent Sync application (`btsync`) running on a Linux desktop or server. `btsync` runs as a non-privileged user and any user with adequate sudo privileges can use `btsyncctl` to pass it basic controls like `start`, `stop`, and `status`.

###Installation Instructions for btsync and btsyncctl


#####Add btsyncctl to your system

    git clone https://github.com/devhen/btsyncctl.git
    sudo cp btsyncctl/btsyncctl /usr/bin/
    sudo chmod +x /usr/bin/btsyncctl


#####Create the user that btsync will run as

On RHEL / CentOS / Fedora:

    sudo adduser btsync

On Debian / Ubuntu:

    sudo adduser btsync --disabled-password


#####Download btsync

Get btsync for your system architecture (i386 or x64) from here: <http://www.bittorrent.com/sync/download>

Put the btsync binary somewhere it can be executed by your btsync user, such as `~/bin`:

    sudo su -l btsync
    mkdir ~/bin
    cp /path/to/btsync ~/bin/


#####Create a btsync config file

    sudo btsyncctl --dump-sample-config > /home/btsync/btsync.conf
    sudo chown btsync:btsync /home/btsync/btsync.conf
Customize your config file as you see fit. Most settings can be left on their defaults but I recommend setting `device_name` (to your hostname, for example), and setting `force_https` to `true`.

#####Start btsync

    sudo btsyncctl start


#####Check btsync status

    sudo btsyncctl status

This shows whether btsync is running and if so, the version number, architecture (32-bit or 64-bit), listening ports (if you have `netstat`installed), time since it was started, and how much memory its using.


#####Secure the WebUI port


The default WebUI port is 8888. If you are running btsync on your local computer you should be able to access the WebUI right away by going to:

    https://localhost:8888

If you are running btsync on a remote server you'll probably want to open the WebUI port for your IP address only.

On RHEL 7 / CentOS 7 / Fedora with firewalld:

    sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="YOUR.IP.ADDR.HERE" port port="8888" protocol="tcp" accept' --permanent
    sudo firewall-cmd --reload

On Debian / Ubuntu with ufw:

    sudo ufw allow from YOUR.IP.ADDR.HERE to any port 8888 proto tcp

You should now be able to access the WebUI by going to:

    https://yourdomain.com:8888

The first time you access the WebUI it will prompt you to set the WebUI username and password.


#####Open the listening port


Without your listening port open to the public, the btsync network may have to provide a "relay server" to peers that are unable to connect to your btsync server. Using relay servers can slow down transfer speeds so for best performance you should open btsync's listening port by setting `listening_port` in btsync.conf to an available port and then opening that port in your firewall.

On RHEL 7 / CentOS 7 / Fedora with firewalld:

    sudo firewall-cmd --add-port=12345/tcp --permanent
    sudo firewall-cmd --add-port=12345/udp --permanent
    sudo firewall-cmd --reload

On Debian / Ubuntu with ufw:

    sudo ufw allow 12345

If you are behind a router you should enable UPnP in btsync.conf by setting `use_upnp` to `true`. If UPnP is not enabled on your router you will need to manually forward the listening port to the IP of the machine running btsync.


#####Stopping btsync

    sudo btsyncctl stop


#####Other commands

If the first argument passed to btsyncctl isn't `start`, `stop`, or `status` the arguments will be passed along to the btsync binary. Therefore, you can pass arguments that btsync supports, for example `sudo btsyncctl --help` and `sudo btsyncctl --dump-sample-config`.


#####Get read/write access to the synced files


To access, add, or make changes to the files synced by btsync you'll want to make btsync's home directory group-writable and sticky:

    sudo chmod g+rwxs /home/btsync

And add yourself to the btsync user's group:

    sudo usermod -a -G btsync myusername

On some systems you may need to set your's and the btsync user's umasks to `0002`. You can do this by adding the following line to each `~/.bashrc`:

    umask 0002