---
layout: post
title: "My janky home theater setup"
categories: etc
---

I've been iterating on our home theater setup for a few years now. Recently it
feels like it has stabilized somewhat so I think it's time to share my work with
the world. I am curious what people think and especially if people know of a
better way to do things than I do.

# Features

This home theater setup does a lot. Here is what I've integrated into one
system:

1. A receiver for controlling different HDMI inputs:
    * Xbox
    * Blu-Ray player (a PS3 at this time)
    * HTPC
    * Apple TV
2. A home server for
    * Data backups
    * Media sharing
    * Hosting simple web apps
3. Kodi, for playing local media
4. Steam, for local multiplayer games
5. Spotify, for music
6. My gaming desktop PC
7. All controlled primarily via smart phone

Here is how it all looks together:

<figure>
<img src="{{ site.baseurl }}/assets/theater.png">
<figcaption>
Ideally there would be a line from the IR blaster to the Blu-Ray player but...
read on.
</figcaption>
</figure>

# The easy part

Most of this is pretty low tech. If you ignore the HTPC and its attachments and
focus on the HDMI-connected portion, this is a simple receiver-based setup. Each
device has its own remote, you use the receiver's remote to switch between
inputs, and you use the TV's remote to turn the TV on and off. Done.

# HTPC

This is where it gets interesting. Inspired by Jeff Atwood's [HTPC
builds][htpc], I built a low-power HTPC of my own. This was about 5 years ago so
my hardware specs are probably irrelevant, but other than replacing a failed
drive the thing has stood up very well over the years. I don't have any plans to
replace it any time soon. If I was to build another one, the general components
I would look for are

1. A mid-range Intel CPU with good integrated graphics
2. Lotsa RAM
3. An SSD
4. A couple large NAS HDDs
5. Splurge on a nice high-efficiency PSU, or listen to Jeff and get an external
   PSU if size is an issue for you.

[htpc]: https://blog.codinghorror.com/the-2016-htpc-build/

The reasoning behind this is that this is primarily for streaming and network
storage. When I built this, 1080p was still perfectly acceptable and Intel's
integrated graphics were perfectly cabale of acheiving 1080p _video_ at 60fps.
These days 4K is the goal to aim for and guess what? Intel's integrated graphics
are still keeping up. You just don't need the cost, power, and clunkiness of a
dedicated graphics card in your HTPC.

The SSD is for the OS because you want this baby to start up instantly. The HDDs
are for media storage in RAID 1 because the best kind of backup is the one you
don't have to think about at all.

## OS

My OS of choice is Arch Linux. I definitely don't recommend this in general; you
have to be a bit of Linux nerd to enjoy Arch. You probably _could_ get by with
Windows, but I think Linux (Arch, specifically) has benefits:

1. Linux is the only OS I've used that runs as well on day 2000 as it did on day
   one. I don't know what it is about Mac OS and Windows that causes them to
   sloooowly degrade over the years until you're smashing your keyboard in
   frustration, but Linux doesn't have it. This isn't going to be your main PC,
   so you don't want to have to do regular maintenance on it.
2. Unless you have some very specific software in mind, Linux has the broadest
   software support. The main shortcoming used to be gaming, but with DXVK and
   Steam Play even that has virtually disappeared. So far Arch Linux has been
   able to satisfy all of my requirements.
3. Every OS is hackable, but for an HTPC it's especially important. It probably
   won't have a mouse and keyboard for input and the traditional desktop
   metaphor makes no sense for it. If you want it to behave like a typical
   set-top device you want tight control over the startup process and you want
   to customize the UI. Again, it's possible to do with other OSes but easier
   with Linux, especially a bare-bones distribution like Arch.
