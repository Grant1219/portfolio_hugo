+++
title = "Still not the year of the Linux desktop"
description = ""
tags = ["linux"]
categories = ["Linux"]
date = 2022-04-09T13:50:54-07:00
draft = false
+++

I reinstalled my desktop workstation for the first time in a few years, and I ended up spending more time on it than I expected (shocking right?). This is pretty much the expectation when using Linux, but still a few things surprised me that I thought were solved problems at this point. I tried both Fedora 36 and Debian 11, and decided to stick with Debian (which is what I was using before) after a few days.

Originally I was reinstalling in attempt to troubleshoot an issue I was having with a system crash, so the goal was to test different setups and see if I could reproduce the issue. It turned out to be an issue with my GPU hardware, and swapping in another card fixed it (for now). That's not the point of this post though, and I would rather not get into the sad state of graphics card prices...

### The Fedora vs Debian installer

The Fedora installer is more user friendly than the Debian installer, although in "expert" mode the Debian installer gives you quite a bit of freedom. The one huge issue I ran into with Debian was the partitioner. Fedora detected my disk, asked to open my encrypted volume, and then allowed me to easily assign partitions/volumes to mount points. After finishing the Fedora install, it booted perfectly, and the partitions I asked it not to touch were kept. This **was not** the case with the Debian installer, and I actually went down quite a rabbit hole figuring out how to make it work. This was all because I didn't want to wipe my whole drive, and keep my `/home` partition intact. Anyone using the "happy path" where you can wipe the whole disk wouldn't even notice this edge case, but I think it's important to bring up.

**How to properly detect and use existing partitions with Debian (including crypto volumes)**

When the installer boots, select the keyboard layout then press Alt+F2 to switch over to the terminal. This is required to install some additional packages to read crypto volumes.

```
anna-install cryptsetup-udeb
anna-install crypto-dm-modules
anna-install crypto-modules
```

You can verify they have been "staged" to install later by switching to the terminal output with Alt+F4. Then go back to the installer with Alt+F1. I didn't know about this `anna-install` command at all until I went digging for answers, but apparently it waits until after the package manager is setup to download and install packages.

If not using a crypto volume, this step can be skipped, but I was so I needed to open the volume. This has to be done before the disk detection/partition step.

```
cryptsetup luksOpen /dev/nvme0n1p3 <volume-name>
```

If using LVM and for some reason the volumes don't automatically activate, this command will activate them:
```
vgscan
vgchange -a y
```

The Debian partitioner should now show all the partitions/volumes, allowing ones to be selected for certain mount points, and provide the choice to either wipe or keep the data. It's a shame this doesn't happen through prompts like the Fedora installer (am I just using it wrong?).

This last step is what tripped me up the first attempt, and also only applies to crypto volumes. If you don't recreate the `/etc/crypttab` file, GRUB will assume there is just a normal non-encrypted disk, and the system **will not boot** after the installation completes.

```
printf "<volume-name>\tUUID=%s\tnone\tluks\n" "$(cryptsetup luksUUID /dev/nvme0n1p3)" >> /target/etc/crypttab
```

After this the GRUB configuration and installation can be run, and hopefully everything will boot as expected. I already knew how to setup crypto volumes, configure LVM, and install GRUB myself, but the fact I had to do all this during the Debian install felt strange and not well supported.

These tricks I learned thanks to a very helpful forum thread here (along with a lot more detail):

https://forums.debian.net/viewtopic.php?p=609446&sid=34acce60be601962e011e7a7d6aefe89#p609446

### GNOME, KDE Plasma, Sway?

I have been using i3 for years now, and still really enjoy it, but I made a point this week of trying out some new desktop and window managers. If I always use the same thing and never change my setup, I won't know what else is available and might miss an opportunity to upgrade or improve my workflow.

Wayland has been the replacement for X11 for a while now, and all three of the DMs/WMs I tried have Wayland support. Unfortunately after some testing and research, I decided to go back to X11 because many games don't yet run on Wayland, and the graphics driver support isn't great either (although I heard it's much better than it was).

**GNOME**

I actually tried the default GNOME that comes with Fedora 36 (GNOME 40?), and then the "classic" GNOME variant. The latest GNOME is honestly terrible. I don't know if the intention was to design it for tablets or maybe laptops, but it seriously feels like a dumbed down phone OS. The "classic" option was marginally better, but still used the same limiting settings windows and other applications that I just could not deal with. I understand the developers are trying to make it easy to use, but it just seems like they've gone too far (similar to the reason Apple OSes annoy me). I couldn't even figure out how to apply a different desktop wallpaper to each screen.

{{< figure src="/images/gnome.png" alt="GNOME desktop" >}}


**KDE Plasma**

KDE Plasma 5 supports Wayland and X11. I remember using KDE for a while way back when I used to run Kubuntu, and I recall it being fairly polished (even with the crazy 3D cube desktop enabled, remember that?). KDE Plasma didn't really seem polished to me though, despite being a decade newer. Maybe it was the default theme, or the translucent menu that was difficult to read, but everything kind of felt clunky and jumbled... not great. I know I can just change the theme, color scheme, or whatever else, but the defaults discouraged me right away. I decided to bail and reinstall within an hour or so. Sorry KDE...

Another bit that annoyed me which I only realized afterwards is that KDE dumped tons of config files into my `~/.config` directory, and I had to manually clean them up later. There just seems to be so many layers and interdependencies in KDE or GNOME desktop environments, and it all just feels fragile to me.

{{< figure src="/images/kde_plasma.png" alt="KDE desktop" >}}

**Sway**

Sway is the replacement for i3 that supports Wayland. i3 is apparently very tightly coupled with X11, so there really isn't a migration path to Wayland for it. The configuration is meant to be mostly compatible, with some notable differences in naming which it will complain about obviously enough for you to fix via trial and error. For the most part Sway worked well, and if Wayland eventually does become more mature I wouldn't mind it. Read more about the differences here: https://github.com/swaywm/sway/wiki/Differences-from-i3

It seems like after trying all the latest desktop options I really can't do better than a tiling window manager like i3 or Sway. The minimal theming and extremely quick window navigation just can't be beat.

{{< figure src="/images/sway.png" alt="Sway window manager" >}}

### Reinstalling all my applications

After testing Fedora and eventually going back to Debian and i3, it was time to reinstall everything and hopefully have a working system again. I know many people have automated workstation setup scripts, or other ways of restoring their setups, but I usually just keep a list of programs I use and reinstall them as needed. This is a good exercise for me in figuring out what I really need, and what I just install by habit or don't need anymore. It also gives me a chance to discover and learn about new applications that might work even better for me.

The other problem I've had with automation scripts is they inevitably break because things change, and then it just becomes another thing to maintain/fix. How often does one usually reinstall their workstation? This was the first time in a couple years for me, and I definitely came across issues where packages have been deprecated, or dependency names change and the package doesn't even install (looking at you Discord). One really strange discovery is that the latest Thunderbird depends shared libraries from a deprecated `libdbus-glib-1-2` package.

Anyway, several days later of testing and messing around with different installs left me nearly back where I started, and with a few new tips on configuring i3 and the rest of my system. I'll leave the links to some of the blogs I referenced below.

https://ericren.me/posts/2019-01-27-minimal-ubuntu-tiling-wm-setup/

https://major.io/2021/07/12/tray-icons-in-i3/

All in all, although the default "happy path" for Linux installs has definitely come a long way over the years. As soon as one departs from the defaults for any distribution, Linux still becomes an advanced user only experience very quickly. That's fine by me, but I still thinks it's far from "the year of the Linux desktop".
