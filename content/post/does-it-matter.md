+++
date = '2025-02-03T12:05:14+02:00'
draft = false
title = 'Does It Matter?'
tags = ['life', 'smart home', 'technology', 'productivity', 'zigbee', 'matter', 'z-wave', 'home-assistant', 'home-automation']
+++

## Introduction and the final thoughts
Long time no see! I've been on vacation and finally got some time to start automating my home. I've been thinking about it for a while. But I never thought that there is no standard way!
<!--more-->
Yes, there is a [Matter](https://buildwithmatter.com) standard. But it's not widely adopted yet. And there is [Z-Wave](https://www.z-wave.com) and [Zigbee](https://zigbeealliance.org).
And then there is an `ecosystem` around it. I mean, you can't just buy a device and expect it to work with everything. You need a hub, a bridge, a gateway, or a controller or a dedicated app. And you need to think about the compatibility.
I am not even saying there is IR, RF, Bluetooth etc.
I played with Home Assistant, and I like it. I can use Python, and it is actually more than just a home automation system. 
But I wanted my wife to do some basic things without me. 

And I wanted to have a simple way to add new devices.
And while many things obvious to me, they are not for her. And I don't want to spend time explaining how to do things.
I can expose devices to HomeKit, but due to limited supported features it's not always the best way.
For example, my Bosch dishwasher propagates as a switch list, which is more than enough. But I wanted to see the status and the remaining time.

I could create a template sensor, but it's not the best way since AFAIK HomeKit cannot recognize time sensors etc. They are all just a humidity sensor or a switch :)
I might dig dipper though. Same for my Vaillant boiler. Many modes are incorrectly propagated. And I don't want to create a custom component for everything.
And there is sometimes fails in sending commands (that is probably on Vaillant side, but still).
I also tried Homey Pro. It is a beautiful device. But sometimes I felt cheated. Official apps not working. Common devices not supported. So basically it is a proprietary expensive (yes, a lot of antennas) system, but it still relies on community support.
And community is a little bit toxic, because they are not happy with the company.
But when something work - I LIKE TO USE IT.

On the other side of the problem is the standard zoo. Not every company tries to make their Zigbee/etc. device compatible with everything. 
Many while-label devices are not even recognized by the hub cause Tuya (for example) creates non-standard variations of devices.
And such Chinese companies are a big share of the market. And while products might be great considering the price, I do not want to use their apps/clouds.
And now there is Matter. And I am excited. Maybe. Cause list of supported devices is not that big. And I am not sure if it will be soon. 
Unlike custom integrations it still lacks many features for specific devices, like configuration options for switches, specific thermostats modes etc.

And the most annoying part is the abuse of Wi-Fi. I am not against Wi-Fi devices (it is a must-have for a Camera/Media devices, right?), but my door sensor do not need a charging cable or battery replacement every month.
And while Thread is a good radio - number of devices is just ridiculous.
So right now I will probably stick to HA, Homey and Homekit (HA and Homey as bridges) and experiment with this setup more.
