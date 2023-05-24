---
title: What is a VPN? How does it work?
description: A virtual private network (VPN) is a method to create a secure connection over an insecure network like public internet.
date: 2023-04-30 00:40:00 +0530
categories: [networking]
tags: [vpn, remote-access, virtual-private-network]     # TAG names should always be lowercase
render_with_liquid: false
---


Recently I setup a VPN for one of my client. Prior to that I was among those who used to use VPN services

Let's break it down further
## What is a VPN?
Before we begin, let's understand some basic terms of networking.
### Network
In this context, a network is a group of computers connected with each other either physically (via [Ethernet cable](https://en.wikipedia.org/wiki/Ethernet_crossover_cable)) or wirelessly  (Wi-Fi or Bluetooth)
### Private Network
A private network is a network of computers connected locally or in a restricted local environment at your home or at any organization. A local area network (LAN) is an example where you use private address space of IP (IPv4/IPv6) addresses `192.168.0.0/16`, `172.16.0.0/12` and/or `10.0.0.0/8`. Simple put, the computers connected to this network are restricted to public access.

>While the internet, an infrastructure of connected computer networks, is a **public network**. On internet, one computer or a network can communicate with other computer or network globally.
{: .prompt-info}

### Virtual Private Network
Now we know what is a private network, let's jump to the VPN. Quoting from [wikipedia](https://en.wikipedia.org/wiki/Virtual_private_network)
> VPN can extend a private network (one that disallows or restricts public access), in such a way that it enables users of that network to send and receive data across public networks as if the public networks' devices were directly connected to the private network.

It means an employee can work from home and still connect securely to the organization's private network. 
So this literally meant VPN is just a private network but virtual (not kidding!), with a feature that it encrypts the traffic between the employee's device and the organization's private network and therefore traffic remains private during the communication.  

Which lead us to the formal [definition of VPN](https://en.wikipedia.org/wiki/Virtual_private_network):
>A virtual private network (VPN) is a mechanism for creating a secure connection between a computing device and a computer network, or between two networks, using an insecure communication medium such as the public Internet.

## Why do we need a VPN?

## Types of VPN
### Remote Access

### Site-to-Site

## How does a VPN work?
TLS protocol for remote access. IPSec for site-to-site
full tunnel and split tunnel?
A self explanatory diagram is must

------------

Resources:
1. [Virtual private network](https://en.wikipedia.org/wiki/Virtual_private_network)
2. [What Is a VPN? - Virtual Private Network](https://www.cisco.com/c/en/us/products/security/vpn-endpoint-security-clients/what-is-vpn.html)
