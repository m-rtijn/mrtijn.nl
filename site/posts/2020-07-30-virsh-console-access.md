---
title: Console access via virsh console for Ubuntu guest VMs
published_date: "2020-07-30 20:42:03 +0000"
layout: blog_post.liquid
is_draft: false
data:
  last_updated_date: 2020-07-30
  published_date: 2020-07-30
---
Sometimes it's a hassle to open up `virt-manager` for just running a quick command on
a guest VM. Luckily, `virsh` has the `console` command to attach to the serial console
of a guest. But because the serial console is not by default enabled on Ubuntu Server (at
least, not as far as I know), you have to enable it manually.

For this, I created a small `ttys0.service` file:

<pre>
[Unit]
Description=Serial Console Service

[Service]
ExecStart=/sbin/getty -L 115200 ttyS0 screen-256color
Restart=always

[Install]
WantedBy=multi-user.target
</pre>

Install the service, load it with `systemctl daemon-reload`, and be sure to enable it
so that it automatically starts after boot. And that's it! Now you can easily access the
console of your guest VM using `virsh console`.

