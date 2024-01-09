---
title: What is a VPN? How does a VPN work? A tutorial for absolute beginners.
description: A virtual private network (VPN) is a method to create a secure connection over an insecure network like public internet.
date: 2023-08-14 5:00:00 +0530
categories: [networking]
tags: [vpn, remote-access, virtual-private-network]  # TAG names should always be lowercase
render_with_liquid: false
# pin: true
---

![vpn-light](/assets/img/media/vpn-main-light.drawio.svg){: .light }

![vpn-dark](/assets/img/media/vpn-main-dark.drawio.svg){: .dark }

## What is a VPN?
Before we begin, let's understand some basic terms of networking.
### Network
In this context, a network is a group of computers connected with each other either physically (through [Ethernet cable](https://en.wikipedia.org/wiki/Ethernet_crossover_cable)) or through wireless (Wi-Fi or Bluetooth)
### Private Network
A private network is a network of computers connected locally or in a restricted local environment at your home or at any organization. A local area network (LAN) is an example where you use private address space of IP (IPv4/IPv6) addresses `192.168.0.0/16`, `172.16.0.0/12` and/or `10.0.0.0/8`. Simple put, the computers connected to this network are restricted to public access.

>On the other hand, the internet, an infrastructure of connected computer networks, is a **public network**. On internet, one computer or a network can communicate with other computer or network globally.
{: .prompt-info}

### Virtual Private Network
Now we know what is a private network, let's jump to the VPN. Quoting from [wikipedia](https://en.wikipedia.org/wiki/Virtual_private_network)
> VPN can extend a private network (one that disallows or restricts public access), in such a way that it enables users of that network to send and receive data across public networks as if the public networks' devices were directly connected to the private network.

It means an employee can work from home and still connect securely to the organization's private network. 
So this literally meant VPN is just a private network but virtual (not kidding!), with a feature that it encrypts the traffic between the employee's device and the organization's private network and therefore traffic remains private during the communication.  

Which lead us to the formal [definition of VPN](https://en.wikipedia.org/wiki/Virtual_private_network):
>A virtual private network (VPN) is a mechanism for creating a secure connection between a computing device and a computer network, or between two networks, using an insecure communication medium such as the public Internet.

## Why is the need of a VPN?
When you exchange data/information on the internet then it can be spoofed by ISP, hackers and government agencies. No one likes this eavesdropping. This is where a VPN comes in handy. A VPN is used to have a secure and encrypted connection over the internet from unauthorized entities.  

By using a VPN you can securely
+ connect two sites of an organization at different location over the internet  
+ connect a device (laptop, pc, tablet, phone) to the office/home network from outside/remote
+ browse internet and keep your identity private. 

## Types of VPN
### Remote Access
It allows to securely connect a computer remotely to a private network over the internet.
VPN is configured on the server side (VPN server) and the server listens for the incoming connection requests. The client computer requires a VPN app running to request a connection, for example openvpn client app. Once the connection is established encrypted data can be exchanged between the computer and the private network over the internet.

### Site-to-Site
It allows to securely connect two sites/offices over the internet. To establish and maintain a connection either a router or a firewall is configured on each sites.

## How does a VPN work?
In thins post, I will explain the remote access VPN. A VPN setup has two components, a **VPN server** and a **VPN client**.  

+ VPN client sits on your computer. You connect to the VPN using this client running in the background. Between the VPN server and computer, it creates a VPN tunnel (an encrypted connection that encapsulates data packets inside other data packets). 

+ When you request [https://haccks.com](https://haccks.com) in the browser, you are connecting to this site through your ISP (Internet Service Provider). This ISP assigns a unique public IP address to your computer. 


+ The VPN client will encrypt this request and send it to the VPN server.   

+ VPN server will decrypt this data and then connect to the the website on your behalf and act as a proxy: meaning [https://haccks.com](https://haccks.com) will see the IP address of the VPN server and not the IP address of your computer assigned by your ISP.  

+ The requested resource will be send back to the VPN server. It will then encrypt the resource and send it back to your computer.   

+ VPN client will decrypt this data and will show you the home page of [https://haccks.com](https://haccks.com)  

![vpn-remote-light](/assets/img/media/vpn-ra-1-light.drawio.svg){: .light }

![vpn-remote-dark](/assets/img/media/vpn-ra-1-dark.drawio.svg){: .dark }  

**But hey, how does a VPN help an employee sitting in Bangalore to remotely connect to a computer at office in Seattle?**  

It will be pretty much similar to the steps discussed above but, to make it simple I skipped one detail in the above steps:   

> *VPN server dynamically assigns a unique IP addresses to each of the client connected to it.*  
  
Let's say the VPN server is using the `10.8.0.0` series of IP addresses. When a computer is connected to this server then it will assign an IP address to that computer from this pool of IP address.    
So, if it assigns the address `10.8.0.1` to the employee's computer and the addresses `10.8.0.2`, `10.8.0.3`, `10.8.0.4` to computers at Seattle office then each devices can ping each other behaving like they are all connected to the same private network.  

You should note that each of these devices will also have IP addresses assigned to them by their respective routers at their ends. For example: `192.168.1.10` for device in Bangalore and `172.16.10.100`, `172.16.10.101`, `172.16.10.102` for devices at Seattle office. In the diagram below only the IP address assigned by VPN server to the devices are shown to make it simple.

![vpn-remote-light2](/assets/img/media/vpn-ra-2-light.drawio.svg){: .light }
![vpn-remote-dark2](/assets/img/media/vpn-ra-2-dark.drawio.svg){: .dark }

------------

Resources:
1. [Virtual private network](https://en.wikipedia.org/wiki/Virtual_private_network)
2. [What Is a VPN? - Virtual Private Network](https://www.cisco.com/c/en/us/products/security/vpn-endpoint-security-clients/what-is-vpn.html)
