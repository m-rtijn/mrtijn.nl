+++
title = "Automatically mount NFS shares over Wireguard at boot"
date = "2022-08-14T12:00:00+0200"
+++

I use [Wireguard](https://www.wireguard.com/) to remotely connect to some self-hosted services.
One of these services is a simple fileserver via [NFS](https://en.wikipedia.org/wiki/Network_File_System).
I however ran into a problem when I tried to automatically mount NFS shares over Wireguard via fstab.
In this post, I will show what my problem was and how I fixed it.

## Prerequisites

I am assuming that you have a server and a client, which are connected via
Wireguard. The client runs a Linux-distribution with systemd and automatically
starts Wireguard at boot by use of the `wg-quick@wg0.service` systemd service.

The client needs to be able to mount NFS shares. On Debian-based distributions,
the `nfs-common` package has to be installed for this.
The server has made at least one NFS share available to the client.

## Manual mounting

Mounting an NFS share over Wireguard works the same as mounting a share that
is made available via on a LAN. Let's say we're connected via Wireguard to
our server, and this server has `10.0.0.2` as its IP in the Wireguard VPN. This
server has made the share `/nfs/exampleshare` available to us. Mounting this share
manually to `/mnt/nfs` is done in the same manner as for a local share:

```
# mount 10.0.0.2:/nfs/exampleshare /mnt/nfs
```

## Mounting at boot

You can put the NFS share in fstab, but when I tried this I ran into a problem:
*fstab automounting is done before Wireguard is set up*.
So, I figured out that you can create your own `.mount` systemd units.
That way, you can customize *when* your NFS share is mounted. Systemd actually
uses these `.mount` units to mount devices and shares according to the entries
in `/etc/fstab`.

So first, comment out any line regarding the NFS share in your `/etc/fstab`.
Then, create `/etc/systemd/system/mnt-nfs.mount` with the following content:

```
[Unit]
Description=Mount 10.0.0.2:/nfs/exampleshare on boot
Documentation=man:systemd.mount(5) man:systemd.unit(5) man:mount.nfs(8) man:nfs(5)
After=network.target wg-quick@wg0.service
Requires=wg-quick@wg0.service

[Mount]
Where=/mnt/nfs
What=10.0.0.2:/nfs/exampleshare
Type=nfs
Options=defaults,auto

[Install]
WantedBy=multi-user.target
```

You can supply your usual mounting options by adjusting `Options=` to your needs.
Furthermore, make sure that you use the correct name of your Wireguard interface.
I am using `wg0`, but you might have set it up differently.

After writing this systemd unit, load it with `systemctl daemon-reload`. Then you
can enable it using `systemctl enable mnt-nfs.mount` so that it actually is executed at boot.

If you have multiple NFS shares you want to mount, you can obviously create multiple
`.mount` units.