---
layout: post
title:  "Setting up HPN SSH on Debian Stretch"
date:   2017-11-10 17:45:35 +0100
categories: ssh hpn server
---

Before I dive into the guts of this article, let me explain to you in short
why I felt the need to setup HPN-SSH in the first place. If you don't care,
you can of course always skip ahead ;)

For quite some time we have been using ZFS now at our company. We have set up
two storage pools that are mirrored via DRBD and on top of that is another (main)
storage pool. This is combined with an overly complicated scheme of snapshot
creation and rotation, just to make our employees sleep well at night.

We decided that two servers are not enough redundancy and set up a third instance
that should house all snapshots ever made and store them for good before they
were being destroyed on the first pair of servers.

Of course creating the snapshot data took quite some time. But when it was finished
I was looking forward to transfer the data over the 10GBit connection between our servers
and enjoy ridicoulus transfer speeds. But that was not the case; in fact, the transfer
speed was disappointingly low.

So I conducted some research and found out about HPN SSH.

HPN SSH is basically just a patch that alters some of the OpenSSH source code to
avoid certain limitations and thereby boosting transfer speeds. Setting it up
is actually pretty easy, so let me explain to you how to do it.

## How to Setup HPN SSH

We are using Debian Stretch on our servers, so the following instructions are tested only on this
very operating system.

### Get the sources

We could download the source code from the OpenSSH project website. But actually we won't. Instead
we will stick with the version that is being shipped with Debian by default. So to get the
source code, fire up a terminal and enter as a non root user preferrably:

```bash
apt source openssh-server
```

This will download the source. To make sure that you can actually build the program, you
need all the dependencies installed:

```bash
apt build-dep openssh-server
```

Right now there is no way to download the patch from the command line, so you have to
head over to [Sourceforge](https://sourceforge.net/projects/hpnssh/files/?source=navbar) and
grab your copy there. Click on the version that matches your OpenSSH version and download
the file that is called

```bash
openssh-X_X_PX-hpn-XX.XX.diff
```

Now go ahead and delete the source folder of openssh, as it contains a version that
has potentially been altered by the Debian project. Then extract the original sources.

```bash
rm -rf openssh-X.XpX
tar -xzf openssh-X.XpX.orig.tar.gz
```


### Compile and install

Now let's apply the patch and build!

```bash
cd openssh-X.XpX
cp /path/to/patch.diff .
patch -p1 < openssh-X_X_PX-hpn-XX.XX.diff
./configure
make
```

After that is done (could take a while), you can safely remove the original version
of OpenSSH and install the new one.

```bash
sudo apt purge openssh-server
sudo make install
```

Don't forget to start the server!

```bash
/usr/local/sbin/sshd
```

When you reconnect to your server, you will notice that the fingerprints have changed.
This is normal, you can safely delete the old fingerprints and accept the new ones.

That's it! Remember to enable autostart of your new ssh server, for example by creating
a service file for systemd. I will explain how to create one in a later tutorial.

HPN SSH made file transfers two to four times as fast on our servers. I hope
that your experiences will be at least this good, or maybe even better :)
