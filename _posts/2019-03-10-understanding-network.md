---
title: Understanding Network
categories:
- Backend
- Node
tags:
- network
- tcp
- udp
- network layers
- subnet
- route table
- AWS
- VPC
- gateway
- network interfaces
- DNS
- NAT
description: Understanding some key components of networking on Linux and on Internet,
  with AWS VPC architecture
date: 2019-03-10
author_profile: true
classes: wide
---

# Understanding Network

After learning about AWS VPC, I tried to understand how network works in general, and this post summarizes what I learned about network in general and the similarities with AWS VPC.

# Basics

## Packets

All data is sent over on the internet in **packets**. It contains two parts: a *header* and a *payload/body.* The header contains identifying information such as the origin and destination host and basic protocol, and the payload contains the data.

## Conceptual Layers

There are two models of the network conceptual layers. OSI(Open Systems Interconnect) model and TCP/IP model. Although TCP/IP model, aka Internet protocol suite has been widely adopted.

### OSI Model

OSI model defines 7 separate layers:

- Application
- Presentation
- Session
- Transport
- Network
- Data Link
- Physical

But let's see the TCP/IP model as well and compare those two models.

### TCP/IP Model

- Application

    Transmit user data between applications, and this is where users most often interact with.

    Some common protocols in this layers are HTTP, HTTPS, SSL, FTP.

    It is basically the Application and Presentation layer in OSI model.

- Transport

    Responsible for communication between processes, it build up connections.

    Common protocols in this layers are TCP and UDP.

    It is basically Session and Transport layer in OSI model.

- Internet

    Transport data from node to node in a network, this layer is aware of the endpoints of the connections.

    IP addresses are defined in this layer, such as IPv4 and IPv6.

    It corresponds with the Network layer in OSI model.

- Link

    You can roughly see it as the hardware related layer for now if you are not trying to understand the hardware and interface of network.

    It corresponds with the Datalink and Physical layer.

# Subnets

Every computer is in a subnet, it is a connected group of hosts, and the hosts in a subnet can talk to each other directly.

![LAN](/assets/images/content/network-1-lan.jpg)

## Subnet Masks and CIDR Notation

You can define a subnet with a network prefix and a subnet mask, for example, *10.23.2.0 and 255.255.255.0.*

It might be confusing at this point just to look at those two IP addresses, but before we  go into netmasks and subnets, it's very helpful to understand how to translate the IP address into binary format.

### Binary, Bit, Byte

We know that computer essentially works based on binary, here is a quick refresher on converting between decimal and binary with two examples.

Say we want to convert 27 to binary, what we need to to is to break this number down into powers of two:

![binary](/assets/images/content/network-2-binary.jpg)

### Netmasks

Now let's convert 10.23.2.0 and 255.255.255.0 to binary

10.23.2.0:

`00001010 00010111 00000010 00000000`

255.255.255.0:

`11111111 11111111 11111111 00000000`

You can change any number in the last 8 bits because the match positions in netmask is `0`. With exceptions of all 0s and all 1s, if you are specifying subnet on AWS, AWS reserves the first four and the last IP address for internal networking purposes.

### CIDR Notation

Once you understand how to read the subnet with the netmask, it's easy to understand how CIDR notation works.

In above example, we use 255.255.255.0 to specify which position of bits have to be fixed. The equivalent CIDR notation is `10.23.2.0/24`, the 24 at the end means the first 24 bits are 1.

So the quick way to understand how many addresses are available within a subnet is to use `2 ^ (32 - n)` for `XXXX/n`

# Route Table

A route table is used to determine where network traffic is directed. On linux, use `route -n` to see a result like this:

![linux route table](/assets/images/content/network-screenshot-route-table.png)

For the first row, there is a G in the Flags column, which means the communication for this network must be sent through the gateway in the gateway column, which is 10.23.2.1. For second row, there is no G in the flags, that means the network is directly connected in some way.

## Route Table on AWS

public route table has target is connected with the Internet Gateway

![aws route table 3](/assets/images/content/network-screeshot-routetable-aws-3.png)

`0.0.0.0/0` represents any address on the Internet.

# Network Interface

To connect the physical and internet layer, there is the network interface. You can run `ifconfig`, it will list network interface information, you will see things like *eth0*, *wlan0* or *lo0*.

## Localhost

There is a special interface, the *lo* interface, it stands for **loopback** or **localhost interface**. It is a virtual network interface to connect applications and processes on a single computer to other applications. The IP address range of *127.0.0.0-127.255.255.255* is used for localhost, and usually we use *127.0.0.1* to represent.

If you see the content in `/etc/hosts`, you should see a line:

`127.0.0.1	localhost`

localhost is the host name of 127.0.0.1

### Localhost VS 0.0.0.0

Sometimes you will see people use `0.0.0.0`  to connect local applications, but those are two different things. Localhost is a virtual network interface, the loopback interface, whereas `0.0.0.0` is an address used to refer to all IP address on the same machine, it will try to connect to every available interface.  If you check the AWS route table config screeshot, you should see for public route table, the destination `0.0.0.0/0` is used to represent the internet.

# DNS

## What is DNS

DNS stands for Domain Name System, it is basically the phonebook of the Internet. Human uses domain name to go to a website(like www.abc.com), but behind the scenes, DNS finds the matching IP address for the domain name so that computers can access the right resources

## How does DNS work

### DNS Server Types

1. DNS Recursor

    It is designed to receive queries from client machines, and making additional requests.

    It can be thought of as the librarian you can ask to find a book in a library

2. Root Nameserver

    It serves as a reference to other more specific locations, like TLD nameserver below.

    It can be thought of an index in a library that points to different racks of books.

3. TLD Nameserver

    Top Level Domain server hosts the last portion of a hostname, such as for example.com, the TLD server is *.com*

    It can be thought of as a specific rack of books in a library.

4. Authoritative Nameserver

    It is the last stop in the query process, it holds the requested record and it will return the IP address to the DNS recursor.

### The DNS Lookup Process

![DNS lookup](/assets/images/content/network-3-DNS.jpg)

### DNS Record Types

DNS records is also known as *zone files,* they are instructions in authoritative DNS servers with the matching IP address. All DNS records have a TTL(time to live) attribute indicates how often a DNS server will refresh that record.

Common type of DNS records are:

- A record: IPv4 addresses
- AAAA record: IPv6 addresses
- CNAME record: Canonical Name record, it forwards one domain to another domain, maps an alias name to a true domain name, it does not provide an IP address.
- MX record: email server

## Tools

    dig google.com
    dig [TYPE] domain.com # CNAME, A... type
    dig @8.8.8.8 example.com
    # 8.8.8.8 is google's DNS server
    dig +trace domain.com
    dig +short domain

    dig -x 172.2.2.2 # make a reverse DNS query
    # similar to
    host google.com

## AWS Route53

It is a highly available and scalable cloud DNS web service.

![aws route53](/assets/images/content/network-screeshot-aws-route53.png)

# Transport Layer

Transport layer bridge the gap between raw packets of internet layer and the refined needs of applications. The popular protocols in this layer are TCP and UDP, and TCP is the most popular protocols.

## TCP

- TCP breaks down and assemble raw packets in the correct order to form data stream, and it will retry if the connection fails.
- it provides for multiple network applications on one machine by utilizing **ports**
- it establish connection by three-way handshaking

![tcp 3 way handshake](/assets/images/content/network-4-tcp.jpg)

- usually who acts as the server listens on famous ports like 80
- superuser use port 1-1023, user process uses port number > 1024.
- some famous ports including port 22 for ssh, port 80 for http, port 443 for https
- TCP is more reliable and complex compared to UDP. for UDP, it transport only for single messages, there is no data stream.
- TCP ports 999 and UDP port 999 are different

## Tools

There is a tool called *netcat/nc* for reading and from and writing to network connections using TCP or UDP. For example:

    nc -l [PORT] # start a server
    nc [IP] [PORT] # acts like a client
    # you can type anything on the client side,
    # and you will see it gets transmitted to the server side

Another trick to send large file between computers in the same network(even without internet) is:

    nv -l 8080 > file
    cat file | nc [IP] 8080

# Network Address Translation

NAT is the most commonly way to share a single IP address with a private network, a variant of NAT is known as IP masquerading.

## How NAT works

1. Assume there is a LAN with a subnet
2. One of the hosts on the private network wants to make request to the Internet
3. Hosts on private network need to go through router to connect to the Internet, router actually intercept the request from the host
4. Router uses the information on the original request from the host, and start its own connection to the internet
5. Once router obtains the connection to the target server, it fakes the connection established message back to the host with the original request

## AWS NAT Gateway

For AWS, the instances you launch into a private subnet in a VPC is not able to communicate with the Internet through the Internet Gateway (remember in route table section, we have a public route table to associate public subnet with Internet Gateway, but we didn't do anything for private route table). If those instance needs to install software, we will need to add NAT Gateway to make it happen

# Other Concepts And Tools

- MAC: Media Access Control. This is an unique identifier for each device
- DHCP: Dynamic Host Configuration Protocol. This will automatically provides IP and other configuration for the network
- SSH VS SSL:

    SSH stands for Secure Shell, SSL stands for secure socket layer. SSL usually applies with HTTPS to add encryption.

- Other common network tools:

        # checks if you can reach a host and its latency
        ping [HOST]

        # tells the path a packet takes to get to a destination
        traceroute [HOST]

        # make HTTP requests
        curl [HOST] [-H --data...]

        # view network packets being sent and received
        tcpdump

# Configure a Secure Web App in AWS

![aws vpc](/assets/images/content/network-5-aws.jpg)

To see the detailed steps to create the structure, check this [post](https://shuofeng.dev//deploy-a-more-secure-rails-app-on-aws-with-vpc-part1/) and related concepts [here](https://shuofeng.dev//aws-vpc-concepts/).

# Useful links

- [networking zine](https://jvns.ca/networking-zine.pdf)
- [Bits, Bytes, Building With Binary](https://medium.com/basecs/bits-bytes-building-with-binary-13cb4289aafa)
- [An Introduction to Networking Terminology, Interfaces, and Protocols](https://www.digitalocean.com/community/tutorials/an-introduction-to-networking-terminology-interfaces-and-protocols)
- [Understanding IP Addresses, Subnets, and CIDR Notation for Networking](https://www.digitalocean.com/community/tutorials/understanding-ip-addresses-subnets-and-cidr-notation-for-networking)
- [What is DNS? | How DNS works](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [StackOverflow Question about localhost vs 0.0.0.0](https://serverfault.com/questions/876698/whats-the-difference-in-localhost-vs-0-0-0-0)
- [Bite Size Networking](https://wizardzines.com/zines/bite-size-networking/)
-
