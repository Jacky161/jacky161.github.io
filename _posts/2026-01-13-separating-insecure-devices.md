---
title:  "Separating Insecure Devices in your Network + OpenWRT Ramble"
excerpt: "Why IoT devices can cause security risks, and how to mitigate them. I also ramble about confusing OpenWRT things."
date: 2026-01-13
categories: networking
toc: true
---

# Preamble

These days, it seems like everything is connected to the internet. The world of the Internet of Things is constantly growing, from smart fridges, cars, your thermostat, maybe your toilet someday!? An issue arises when considering that many IoT devices are not kept very secure by manufacturers, leading to them being a potential entrypoint into your network for hackers. Just as an example of the insecurity of these devices, this [article](https://www.malwarebytes.com/blog/news/2025/06/thousands-of-private-camera-feeds-found-online-make-sure-yours-isnt-one-of-them) from Malwarebytes found thousands of public camera feeds from smart cameras. Creepy.

In a traditional network, all devices are unified under one subnet (range of IPs), say `192.168.1.0/24`. This is CIDR notation which represents the range of IP addresses from `192.168.1.0 - 192.168.1.255`.

{% capture consumer_network_diagram %}
![Standard Consumer Network Diagram]({{ site.url }}{{ site.baseurl }}/assets/images/2026-01-13-separating-insecure-devices-basic-network.png)
{% endcapture %}

<figure>
  {{ consumer_network_diagram | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>A typical consumer home network.</figcaption>
</figure>

Here we can see a basic consumer network, likely similar to one that you may have at home. Devices are connected via a router, likely provided by your ISP, which broadcasts a wireless network as well as providing ethernet ports for wired devices. All devices connected to the network will have an IP address within your local subnet. With a setup like this, any device connected to your network can talk to each other. Your smart thermostat at `192.168.1.42` can talk to your computer at `192.168.1.20` if it wanted to. Even especially suspicious devices, like `192.168.1.67` could talk to anyone!

## VLANs and Network Separation

VLANs are a way to separate your network into different logical segments. For example, I could create a VLAN dedicated to my IoT devices and another dedicated to my servers, and setup a rule such that those two VLANs cannot talk to each other. That way, if one of my IoT devices is compromised, they can't try to hack into my server from there. VLANs are associated with different subnets. So we could assign my servers the subnet `192.168.10.0/24`, whereas my IoT devices might live on `192.168.20.0/24`.

{% capture basic_vlan_diagram %}
![Basic VLAN Diagram]({{ site.url }}{{ site.baseurl }}/assets/images/2026-01-13-separating-insecure-devices-rvlan-physical.png)
{% endcapture %}

<figure>
  {{ basic_vlan_diagram | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Diagram from <a href="https://www.practicalnetworking.net/stand-alone/routing-between-vlans/">practicalnetworking.net</a></figcaption>
</figure>

This diagram shows how a network switch can be divided into different VLANs. This requires a special switch called a "Managed" switch which can operate on Layer 3 of the OSI model. If a computer on VLAN 20 wants to talk to a computer on VLAN 30, they must go through a router which may or may not allow the request based on firewall rules. In contrast, devices on the same VLAN do not need to go through a router to talk to each other and can communicate directly.

## The Catch

VLANs sound great! We can separate less secure devices from our network, minimizing our exposure, while still being able to use them. However, in a typical consumer network with just a single router, it turns out that setting up VLANs is a bit overkill. We can accomplish essentially the same effect in OpenWRT using separate bridges for certain ethernet ports and SSIDs (different wifi networks). Each bridge can be assigned a subnet and we can use firewall rules to manage how traffic should be handled. Sounds like exactly what we want!

# Separating with OpenWRT

This [guide](https://openwrt.org/docs/guide-user/network/wifi/guestwifi/configuration_webinterface) from OpenWRT can be followed to setup a separate SSID for an IoT network. Here, I'll highlight some additional details that are not so obvious when following the guide.

## Attaching Ethernet Devices

The guide above doesn't mention how to attach specific ethernet ports to your new bridge. You can do this by simply going into Network --> Interfaces --> Devices. Then select the `br-lan` bridge and remove the ethernet port you want to connect to your new bridge. Then select your new bridge and add it there instead. Simple!

## Bridges and Interfaces

The guide has you create a bridge for your new network and then add an interface onto that with a static address. A bridge can be thought of as like a network switch, connecting many devices together. The devices in our bridge are our IoT devices. We also connect an interface to our bridge. This interface connects the router itself to our bridge. This is distinct from the default `br-lan` bridge, which links our normal SSID, and all the LAN ports on our router by default in one bridge. This is why devices connected to your router can all talk to each other by default. They are bridged together. Creating a new bridge severs that link.

## Firewall

When setting up the firewall rules, this is where you set what you want devices on each firewall zone to be allowed to access.

{% capture openwrt_firewall_rules %}
![OpenWRT Basic Firewall Rules]({{ site.url }}{{ site.baseurl }}/assets/images/2026-01-13-separating-insecure-devices-firewall-rules.png)
{% endcapture %}

<figure>
  {{ openwrt_firewall_rules | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>My firewall zone rules for a separate LAN and IoT network.</figcaption>
</figure>

Here are some firewall rules that I've set up as an example. Let's go through the columns and talk through them.

The `Zone ⇒ Forwards` column tells us which zones are allowed to communicate with each other. We can see that the `lan` zone is allowed to talk to the `wan` zone as well as the `iot` zone. The `wan` zone has traffic rejected which makes sense as we don't need outside devices from the internet being able to forward packets to our home devices. However, if I visit a website like Google, then the firewall knows to allow traffic from Google back to my device, since my device was the one to contact them first. The `iot` zone can access `wan` but can't access anything else. This means that `iot` devices can't initiate connections to my other devices unless they initiate the connection first. The reason I set `lan` devices being able to talk to `iot` is so that for example, I can view the footage from my surveillance camera since my device is initiating the connection, but my camera can't talk to my devices on its own.

The next three columns are `Input`, `Output`, and `Intra zone forward`. `Input` refers to whether devices are allowed to talk to the router itself, like the router's web interface. `Output` refers to whether the router can talk to devices in this zone. You usually want this to be allowed since the router should be able to communicate with all devices on your network. Finally, `Intra zone forward` refers to whether the router allows devices within the zone to talk to each other. However, this may not work as you expect. If you set this to reject, the router will not forward any packets between devices on that zone. However, if those devices are within the same bridge (for example, connected to the same IoT SSID), they will still be able to talk to each other since they don't need to use the router in the first place.

# Conclusion

Hopefully you know a thing or two more about VLANs, firewalls, and network bridges now! I sure learned a lot when trying to figure this out myself. I decided to write this because I personally found it very confusing when trying to figure out what all this stuff meant myself. I hope that you will be inspired to take steps to secure your own network and separate out all those suspicious little IoT gadgets.
