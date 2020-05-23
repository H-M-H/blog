---
title: "Cast your iPad's Screen to Linux"
date: 2020-05-23T10:07:55+02:00
draft: false
---

## Overview
This post is going to explain how to record the screen of your iPad and stream it to your Linux
system with latencies of about two seconds. Unfortunately Apple makes things quite a bit more
complicated than they have to be to keep you within their walled garden, so lets see how to break
free.

There are three components to the solution I found:
- An app to stream the screen of your iPad to a Real-Time Messaging Protocol (RTMP) server.
- An RTMP server that receives the stream and distributes it to clients that request the stream.
- A program to connect to the RTMP server and playback the stream.

## RTMP Server
The most complicated part in this setup is the RTMP server. The RTMP server we will use here is
nginx using the RTMP module. On Arch Linux you can install the packages `nginx-mainline
nginx-mainline-mod-rtmp`. On Ubuntu everything can be installed via: `apt install nginx
libnginx-mod-rtmp`. On other distributions you may need to build the RTMP module from source, to do
so refer to the official docs at https://github.com/arut/nginx-rtmp-module.

After the installation nginx needs to be configured for livestreaming. To do so the file
`/etc/nginx/nginx.conf` needs to be modified. If it does not yet exist, create it.
The following configuration should get you up and running, apply customizations if necessary.
```
# this tells nginx to load the RTMP module
# the path to it may vary depending on your distribution
# make sure it actually points to the RTMP module you installed
# also make sure it is placed somwhere at the top of this file
load_module "/usr/lib/nginx/modules/ngx_rtmp_module.so";

rtmp {
    server {

        listen 1935;

        chunk_size 4000;

        application live {
            # enable livestreaming
            live on;

            # no need to record the livestreams
            record off;

            # everyone is allowed to livestream to this server,
            # you may want to limit this to the IP of your iPad
            allow publish all;

            # only localhost can access and play any livestreams
            allow play 127.0.0.1;
            deny play all;
        }
    }
}
```
For documentation on all RTMP options refer to
https://github.com/arut/nginx-rtmp-module/wiki/Directives .

## The App
On your iPad an app is required to stream your screen via RTMP.  After extensive research the only
app I found to be acceptable is [Larix
Screencaster](https://apps.apple.com/us/app/larix-screencaster/id1477313005).  Unfortunately as of
version 1.0.5 it still contains a bug that cuts off a part of your screen if you try to enable
anything else than streaming in vertical/portrait mode. So make sure to set all your settings to
vertical!

The settings in the main menu should look similar to this:
![Larix Screencaster settings 1](/screencast-ipad-to-linux/larix_1.jpg)
To tell the app where to direct the video stream to, you need to add a new connection. Give it a name
and point it to the RTMP server on your computer by pasting a URL like
`rtmp://192.168.42.42:1935/live/ipad` into the URL field. Make sure to replace the IP address with
the actual IP address of your computer! After configuring the connection it should look similar to
this:
![Larix Screencaster settings 2](/screencast-ipad-to-linux/larix_2.jpg)
To start a stream from your iPad follow the instructions show in
https://www.youtube.com/watch?v=yvkkcRP8-2Y .

## Playing the Stream
To playback the stream captured on your iPad you need a program capable of playing RTMP streams. If you are
a friend of running everything in your terminal [mpv](https://mpv.io/) is the right choice for you.
To play the stream the command
`mpv rtmp://127.0.0.1:1935/live/ipad --profile=low-latency -demuxer-lavf-analyzeduration=5` can be used. To workaround Larix
Screencasters problems with non vertical streams mpv can be used to rotate the video with the
additional options `--video-rotate=90`. Sometimes it appears mpv does not recognize the video part
of the stream, consider increasing `-demuxer-lavf-analyzeduration` in that case.
Alternatively
[VLC media player](https://www.videolan.org/vlc/) is equally capable of playing RTMP streams. To
play the stream go to the menu Media and choose Open Network Stream. This will open a prompt for the
Network Stream to be opened, paste `rtmp://127.0.0.1:1935/live/ipad` into it. The stream should
start playing then. VLC can also rotate the video, the option can be found under Tools >
Effects and Filters > Video Effects > Geometry > Transform. I recommend mpv though as I could not
find an option for VLC to minimize playback latency.
![Fin.](/screencast-ipad-to-linux/fin.jpg)
