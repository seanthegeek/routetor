# routetor
Routes VM traffic through Tor for the [Optiv fork of Cuckoo Sandbox](https://github.com/seanthegeek/routetor/edit/master/README.md)

## Overview

Tor is a great way to give your sandbox VM a pseudo-annonymous, live internet connection. If you'd like to use TOR with your sandbox, here are a few things to keep in mind:

Most corporate, university, and other shared networks have policies prohibiting the use of TOR for security reasons. Tor may help keep what you are doing hidden, but it will be very obvious to most network administrators when you use TOR. Get permission if necessary.  

Only DNS and TCP traffic can be routed through Tor, the rest will be blocked if configured properly. As a result, malware might not behave the same way over Tor as it would over a normal internet connection. However, most malware makes use of TCP.

It is possible for a server operator (i.e malware oporator) to recognize that a client is using Tor by checking the IP address against the public list of exit nodes, which may warn attackers that a sample is being analyzed. The Optiv fork of Cuckoo provides a per-analysis toggle for TOR. use it wisely
 
Although TOR may keep your connection anonymous, the content and configuration of your VM may reveal your identity 

For more on what TOR is, and how it works, view [the overview](https://www.torproject.org/about/overview).


## Installation

To install Tor, follow the guide option 2, **not** option 1, [here](https://www.torproject.org/docs/debian.html.en) 

Once Tor is installed, edit its config file

    $ sudo nano /etc/tor/torrc

Add the following lines to the bottom:

    TransListenAddress 192.168.56.1
    TransPort 9040
    DNSListenAddress 192.168.56.1
    DNSPort 5353

Note: `192.168.56.1` is the default address for the first VirtualBox host-only adaptor (`vboxnet0`). Adjust as nessessary for your virtualization setup to match Cuckoo's result server IP address.

Then restart Tor:

    $ sudo service tor restart

Install the scripts that Cuckoo will use to enable and disable Tor routing per analysis:

    $ git clone https://github.com/seanthegeek/routetor.git
    $ cd routetor
    $ sudo cp *tor* /usr/sbin

`routetor` uses a UNIX socket client/server model, so that `root` can take privileged actions on behalf of the `cuckoo` user, which runs the client scripts.

Schedule the `routetor` server to run at system boot

    $ sudo crontab -e

Add the line:

    @reboot /usr/sbin/routetor

Start `routetor` for the first time:

    $ sudo /usr/sbin/routetor &
