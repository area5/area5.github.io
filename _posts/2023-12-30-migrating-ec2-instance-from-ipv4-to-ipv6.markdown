---
title:  "Migrating EC2 Instance from IPv4 to IPv6"
tags: AWS EC2 IPv4 IPv6 VPC Subnet
---

Starting February 1, 2024, AWS will charge $0.005 per hour for each public IPv4 addresses ([official blog](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)). A public IP address is an unique Internet identifier assigned to a service and allows the service to be reachable on the Internet anywhere in the word. I have a small WebSocket server (for a [minesweeper game](https://8cells.co/) I wrote) that runs on a `t4g.nano` instance that is directly connected to the Internet with a public IPv4 address. This writing describes the steps I took to migrate the instance from IPv4 stack to IPv6-only stack and lessons learned.

# Enabling VPC IPv6 CIDR block

To assign an IPv6 address to an instance, the VPC and Subnet in which the instance resides must have an IPv6 CIDR block.

The following steps adds an IPv6 CIDR block to a VPC:

1. In the VPC Console, select "Your VPCs"
1. Select the VPC of the instance, and click "Actions", and "Edit CIDRs"
1. Select "Add new IPv6 CIDR", and use "Amazon-provided IPv6 CIDR block" and save the settings.

Next, we need to make sure the subnet also has a IPv6 CIDR block:

1. In the VPC Console, select "Subnets"
1. Select correct subnet, and click "Actions", and "Edit IPv6 CIDRs"
1. Add a IPv6 CIDR

Once the VPC and subnet has an IPv6 CIDR block, an instance can be launched with a public IPv6 address. In the Instance Launch wizard, click "Edit" in "Network settings", choose "Disable" for "Auto-assign public IP", and choose "Enable" for "Auto-assign IPv6 IP".

The assigned IPv6 address is a public address, so no NAT gateway is required.

# Updating domain name record

I have a DNS hostname associated with the server. I need to add an [AAAA record](https://www.cloudflare.com/learning/dns/dns-records/dns-aaaa-record/) for the new IPv6 address and remove the existing [A record](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/) that's associated with the old IPv4 address of the instance.

The process is simple. The public IPv6 address can be found in EC2 Console under "IPv6 address" for the instance.

# Ping

Use the command `ping6` to ping the IPv6 address and hostname. The `ping` command does not work for me on my Mac.

During initial testing, I tried to ping the IPv6 address with ping command and the command thought I was pinging a hostname. Apparently there is a host out there with a weird long hostname that is assigned an IPv4 address, which really confused me.

# IPv6 support

Due to the fact IPv6 has been a standard since 1998, I thought the transition would be seamless and easy. That's far from true. This sections discusses support of different parts.

## Browser support

My server is a web socket server, so users interact with it in a web browser. All major browsers support IPv6.

I have no issues connecting to my server (https://8cells.co) in a browser. However, if you run into issues connecting, please comment. I'd like to hear about it.

## User IP address assignment

This is for my computer that I want to connect to the server. My experience shows that ISPs will assign an IPv6 to my devices. For example, I have no issue getting an address from wifi at home from AT&T and on my phone from Verizon.

However, this is not true for all wifi connections, especially public ones. I was not able to get an IPv6 address using the Mountain View public wifi, also from the public wifi at a coffee place I went to. If an user cannot get an IPv6 address, then my server it not reachable.

## Ubuntu Software Repo

I needed to install some packages on the server, which runs on Ubuntu. Not all software repo mirrors URLs have IPv6 support, e.g., https://askubuntu.com/questions/1437952/unable-to-apt-get-update-on-ipv6-only-ubuntu-server With some research, I was able to use mirrors that support IPv6.

Snap on Ubuntu also does not support IPv6.

## AWS

Although AWS is pushing for IPv6 adoption, work is needed to make tools easier for adoption. For one, EC2 Instance Connect still requires an IPv4 address associated with the instance at the time of this writing.

# Conclusion

Current support for IPv6 can still be improved, but I hope this push by AWS will accelerate global adoption effort.
