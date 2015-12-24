# routetor
A socket server for routing specific source addresses through Tor

## Overview

`routetor` was originally written for use with the [Optiv fork of Cuckoo Sandbox](https://github.com/spender-sandbox/cuckoo-modified). Tor is a great way to give your sandbox VM a pseudo-anonymous, live internet connection. If you'd like to use Tor with your sandbox, here are a few things to keep in mind:

- Most corporate, university, and other shared networks have policies prohibiting the use of Tor for security reasons. Tor can help keep what you are doing hidden, but it will be very obvious to most network administrators when you use Tor. Get permission if necessary.  

- Only DNS and TCP traffic can be routed through Tor, the rest will be blocked if configured properly. As a result, malware might not behave the same way over Tor as it would over a normal internet connection. However, most malware makes use of TCP.

- It is possible for a server operator (i.e. malware operator) to recognize that a client is using Tor by checking the IP address against the public list of exit nodes, which may warn attackers that a sample is being analyzed. The Optiv fork of Cuckoo provides a per-analysis toggle for Tor. Use it wisely.
 
Although Tor may keep your connection anonymous, the content and configuration of your VM may reveal your identity.

For more on what Tor is, and how it works, view [the overview](https://www.torproject.org/about/overview).

## Installation

To install Tor, follow the guide option 2, **not** option 1, [here](https://www.torproject.org/docs/debian.html.en) 

Once Tor is installed, edit its configuration file

    $ sudo nano /etc/tor/torrc

Add the following lines to the bottom:

    TransListenAddress 192.168.56.1
    TransPort 9040
    DNSListenAddress 192.168.56.1
    DNSPort 5353

Note: `192.168.56.1` is the default address for the first VirtualBox host-only adaptor (`vboxnet0`). Adjust as necessary for your virtualization setup to match Cuckoo's result server IP address.

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
