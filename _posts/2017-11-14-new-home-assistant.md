---
title:        Home Automation with HASS.io and Home Assistant
author:       Eric Ditter
date:         2017-11-26
tags: [Home Automation, HASS.io]
excerpt_separator: <!--more-->
---

When it comes to managing a smart home I like to have everything in one places and up until a few weeks ago, I did that using [SmartThings](https://www.smartthings.com/). Recently I found out I needed to spend $100 to get the new hub to allow for some new features I wanted so I started looking at alternatives.  Somehow I ran across [Bruh Automation's](https://www.youtube.com/channel/UCLecVrux63S6aYiErxdiy4w){:target="_blank"} videos on YouTube where he talks about Home Assistant and it seemed like a good platform so I took one of my spare Raspberry Pi's and threw it on there and I was sold on it!

<!--more-->

After installing Raspian, getting the various LetsEncrypt & DuckDNS pieces setup, and setting up all the other Linux related things that I didn't really know how to do I was ready to get Home Assistant to talk to SmartThings.  I ran across [St. John Johnson's Github repo](https://github.com/stjohnjohnson/smartthings-mqtt-bridge){:target="_blank"} and it was using Docker so I figured, "hey that's a buzz word lately, let's see how it works".  Somehow that got hosed up and I couldn't uninstall it, couldn't run anything Docker and I have no idea why.  ¯\\_\(ツ\)\_/¯

So I figured I would start over but this time I was going to use Hass.io which I ran across while I was doing the installs on Raspian.  It took me 2 hours from the time I started flashing the sd card to the time it was essentially back to where I had it plus I had the SmartThings MQTT bridge working perfectly.

There were a few things I couldn't do the same as before because it is more locked down but I don't have a lot of experience with Linux in general so it was probably safer that way.  And for the most part, it's just done differently in Hass.io but it's still there, you just need to find it.

There are a few learning curve with Home Assistant in general, particularly with the [Yaml](http://yaml.org/) but there are tons of example configs out there so you can start to piece things together pretty quickly. The only drawback that I have really seen with using Hass.io is that there is a lot of existing documentation out there for Home Assistant but sometimes they don't directly apply to Hass.io installs.  Or there could be a list of step you need to follow and then hidden in there is "if you have Hass.io this is done for you" so it doesn't apply. It honestly just comes down to knowing what to look for which took a bit at first.

So that's where I am at.  So far it is a great platform and really easy to extend, I've already written a plugin to make up for the one thing I was missing from the switchover from SmartThings which was integrating [Life360](https://www.life360.com/) location tracking into it so I will be doing a quick write-up on that soon.

Want to learn more?

- [Home Assistant Getting Started](https://home-assistant.io/getting-started/)
- [Home Assistant Podcast](https://hasspodcast.io/feed/podcast/)
- [Bruh Automation YouTube channel](https://www.youtube.com/channel/UCLecVrux63S6aYiErxdiy4w)
