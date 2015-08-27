# routetor
Routes VM traffic through TOR for the Optiv fork of Cuckoo Sandbox

## Overview

The Onion Router (TOR) is a great way to give your sandbox VM a pseudo-annonymous, like internet connection. If you'd like to use TOR with your sandbox, here are a few things to keep in mind:

Most corporate, university, and other shared networks have policies prohibiting the use of TOR for security reasons. TOR may help keep what you are doing hidden, but it will be very obvious to most network administrators when you use TOR. Get permission if necessary.  

Only DNS and TCP traffic can be routed through TOR, the rest will be blocked if configured properly. As a result, malware might not behave the same way over TOR as it would over a normal internet connection. However, most malware makes use of TCP.

It is possible for a server operator (i.e malware oporator) to recognize that a client is using TOR by checking the IP address against the public list of exit nodes, which may warn attackers that a sample is being analyzed. The Optiv fork of Cuckoo provides a per-analysis toggle for TOR. use it wisely
 
Although TOR may keep your connection anonymous, the content and configuration of your VM may reveal your identity 

For more on what TOR is, and how it works, view [the overview](https://www.torproject.org/about/overview).

To install TOR, follow the guide option 2, **not** option 1, [here](https://www.torproject.org/docs/debian.html.en) 

Once TOR is installed, edit its config file

    $ sudo nano /etc/tor/torrc

Add the following lines to the bottom:

    TransListenAddress 192.168.56.1
    TransPort 9040
    DNSListenAddress 192.168.56.1
    DNSPort 5353

Then restart TOR:

    $ sudo service tor restart

Install the scripts that Cuckoo will use to enable and disable TOR routing per analysis:

    $ git clone https://github.com/seanthegeek/routetor.git
    $ cd routetor
    $ sudo cp *tor* /usr/sbin

`routetor` uses a UNIX socket client/server model, so that `root` can take privileged actions on behalf of the `cuckoo` user which runs the client scripts.

Schedule the `routetor` server to run at system boot

    $ sudo crontab -e

Add the line:

    @reboot /usr/sbin/routetor

Start `routetor` for the first time:

    $ sudo /usr/sbin/routetor &
