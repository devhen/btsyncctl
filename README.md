btsyncctl
===========

[BitTorrent Sync](http://www.bittorrent.com/sync) is a powerful cross-platform file sharing application. Think of it as a decentralized version of Dropbox with no charges (it's free), no limits, and no middle-man.

**UPDATE:** BitTorrent Sync is now [Resilio Sync](https://www.resilio.com/platforms/desktop/). The executable changed from `btsync` to `rslsync`. `btsyncctl` works with both.

`btsyncctl` is a simple bash script meant to automate the process of starting, stopping, and checking the status of the Resilio Sync application (`rslsync`) running on a Linux desktop or server. `rslsync` runs as a non-privileged user and any user with adequate sudo privileges can use `btsyncctl` to pass it basic controls like `start`, `stop`, and `status`.

### Installing rslsync and btsyncctl


##### Add btsyncctl to your system

    git clone https://github.com/devhen/btsyncctl.git
    sudo cp btsyncctl/btsyncctl /usr/bin/
    sudo chmod +x /usr/bin/btsyncctl


##### Create the user that btsync will run as

On RHEL / CentOS / Fedora:

    sudo adduser rslsync

On Debian / Ubuntu:

    sudo adduser rslsync --disabled-password


##### Download rslsync

Get rslsync for your system architecture (i386 or x64) from here: <https://www.resilio.com/platforms/desktop/>

Repositories for Debian, Ubuntu, Redhat, CentOS, and Fedora can be found here: <https://help.resilio.com/hc/en-us/articles/206178924>

Put the btsync binary somewhere it can be executed by your btsync user, such as `~/bin`:

    sudo su -l rslsync
    mkdir ~/bin
    cp /path/to/rslsync ~/bin/


##### Create a rslsync config file

    sudo su -l rslsync
    btsyncctl --dump-sample-config > /home/rslsync/rslsync.conf

Customize your config file as you see fit. Most settings can be left on their defaults but I recommend setting `device_name` (to your hostname, for example), and setting `force_https` to `true`.

##### Start rslsync

    sudo btsyncctl start


##### Check rslsync status

    sudo btsyncctl status

This shows whether rslsync is running and if so, the version number, architecture (32-bit or 64-bit), listening ports (if you have `netstat` installed), time since it was started, and how much memory its using.


##### Secure the WebUI port


The default WebUI port is 8888. If you are running rslsync on your local computer you should be able to access the WebUI right away by going to:

    https://localhost:8888

If you are running rslsync on a remote server you'll probably want to open the WebUI port for your IP address only.

On RHEL 7 / CentOS 7 / Fedora with firewalld:

    sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="YOUR.IP.ADDR.HERE" port port="8888" protocol="tcp" accept' --permanent
    sudo firewall-cmd --reload

On Debian / Ubuntu with ufw:

    sudo ufw allow from YOUR.IP.ADDR.HERE to any port 8888 proto tcp

You should now be able to access the WebUI by going to:

    https://yourdomain.com:8888

The first time you access the WebUI it will prompt you to set the WebUI username and password.


##### Open the listening port


Without your listening port open to the public, the rslsync network may have to provide a "relay server" to peers that are unable to connect to your rslsync server. Using relay servers can slow down transfer speeds so for best performance you should open rslsync's listening port by setting `listening_port` in rslsync.conf to an available port and then opening that port in your firewall.

On RHEL 7 / CentOS 7 / Fedora with firewalld:

    sudo firewall-cmd --add-port=12345/tcp --permanent
    sudo firewall-cmd --add-port=12345/udp --permanent
    sudo firewall-cmd --reload

On Debian / Ubuntu with ufw:

    sudo ufw allow 12345

If you are behind a router you should enable UPnP in rslsync.conf by setting `use_upnp` to `true`. If UPnP is not enabled on your router you will need to manually forward the listening port to the IP of the machine running rslsync.


##### Stopping rslsync

    sudo btsyncctl stop


##### Other commands

If the first argument passed to btsyncctl isn't `start`, `stop`, or `status` the arguments will be passed along to the rslsync binary. Therefore, you can pass arguments that rslsync supports, for example `sudo btsyncctl --help` and `sudo btsyncctl --dump-sample-config`.


##### Get read/write access to the synced files


To access, add, or make changes to the files synced by rslsync you'll want to make rslsync's home directory group-writable and sticky:

    sudo chmod g+rwxs /home/rslsync

And then add yourself to the rslsync user's group. You'll need to logout and back in, or start a new shell, for this to take effect:

    sudo usermod -a -G rslsync myusername

To change the default permissions rslsync uses for new files, set the umask by adding this line to the rslsync user's `~/.bashrc`:

    umask 0007
