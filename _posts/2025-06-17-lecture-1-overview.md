---
title: lecture_1_overview
description: Basic definition of network and structure of internet 
layout: post
date: 2025-06-17 06:16 -0700
categories: [Networking, Lectures]
tags: [ece438, networking]
---


# Overview

## Meaning of computer networks
- A series of **network elements** connected together, that implements a set of **protocols** for the **purpose of sharing resources** at the end hosts. 
## What do computer networks do?
- A computer network has one and only 1 task: delivering the data between end hosts
- So any computer network is basically a **data-delivery system**.
- Data delivery is enabled by:
  - chopping the data into packets
  - sending individual packets across the network
  - reconstructing the data from packets at the end host
- The idea is to use computer networks to allow processes on different hosts to exchange messages with logical equivalence of interprocess communication (IPC), which is the mechanism that allows processes on the same host to exchange messages.
- There is a clear seperation of concerns where computer networks just deliver the data and applications running on the hosts decode what to do with the data, allowing networks to be application agnostic.
- In these networks, there are millions of end points, some of which can be addressed by numbers and some which lie behind a virtual end point.
- Many networks came together to form a bigger network and the overall structure called the internet - the network of networks.

## Internet Structure

- The individual networks which make up the internet are called **Autonomous Systems (AS)**, which are the fundamental building blocks of the internet. An Autonomous System (AS) is a large network or a group of networks that operates under a unified routing policy and single administration.Every AS is assigned a unique identifier called an **Autonomous System Number (ASN)**, which is used to manage routing between these massive networks.

- Internet Service Providers (ISPs) have a three tier model based on their role and level of connectivity within the global internet.

### Tier 1 ISPs
- Serve as **backbone of the internet**. Own and manage the primary infrastructure, including vast networks of transcontinental and undersea fiber optic cables (which carry 95% of the traffic).
- They are the largest providers with a global reach, including companies like AT&T, Verizon, Sprint and NTT.
- A key characteristic of a Tier 1 ISP is that it can reach every other network on the internet without paying for transit, which is achieved through **settlement-free peering**, where they mutually agree to exchange traffic with other Tier 1 providers at no cost.

#### How are cables laid out in the ocean?

- First step is route planning and seabed survey where survey ships use SONAR and seismic methods to create high resolution maps of the ocean floor along intended path to find safest path.
- Then there is a cable laying process where cable carrying vessels begin the installation process carrying thousands of kilometers of fiber optic cables coiled in large tanks and slwly unreel the cable at 100-200kms per day, using a special plow on the seabed which digs a trench and lays cable.
  - In shallow waters, cables ae buried in trenches dug into the seabed and remotely operated vehicles used for post lay inspection.
  - In deep water risk of disturbance much lower so cables laid directly onto the surface of seabed.
- The cables are not laid under high tension and have some slack to acommodate minot shifts, including tectonic plate movements, and the cables have multiple layers of steel wire and waterproofing materials to withstand the harsh pressure and temperature.
- A small percent of traffic is also handled by satellite to account for places on land where getting a cable is hard, but this is not preferred because cables carrying capacity is in terabits per second while a satellite is usually in gigabit per second so cable is 1000x faster and more cost effective.

### Tier 2 ISPs

- Are smaller and operate on the national or regional scale. Have a hybrid model of connectivity, where they peer with other Tier 2 providers to exchange traffic for free but must purchase **IP transit** from a Tier 1 provider to gain access to the entire internet.
- Basically a tier 2 ISP pays a tier 1 provider to carry its data packets to and from the global internet.

### Tier 3 ISPs

- Local providers that are closest to the end-users, including businesses and residential customers. Their primary business is selling internet access to consumers.
- They almost exclusively purchase IP transit from Tier 2 or Tier 1 ISPs to connect their customers to the internet as they do nt have their own peering arrangements.

#### Peering vs Transit

- **Peering** is a reciprocal agreement where two networks agree to carry each other's traffic to their respective customers for free, to avoid transit fees.
- **Transit** is a commercial service where a smaller network pays a larger network to carry its traffic.

#### Internet Exchange Points (IXPs)
- An IXP is a physical location, often a data center, where multiple networks can interconnect and exchange traffic directly.
- These facilitate peering between a large number of ISPs, Content Delivery Networks (CDNs) and other major networks in one place.
- By connecting at an IXP, networks can reduce latency and costs by exchanging traffic directly instead of sending it through a third-party transit provider.

## Some core protocols

- The **Transmission Control Protocol (TCP)** and **Internet Protocol (IP)** are the foundational protocols of the internet.
- IP is responsible for addressing and routing data packets to their correct destination, while TCP ensures the reliable, ordered, and error-checked delivery of these packets.
- **Border Gateway Protocol (BGP)** is the routing protocol that makes the internet work by enabling data exchange between Autonomous Systems. BGP's primary function is to find the best path for data to travel across the vast network of Autonomous Systems. Allows routers to dynamically adapt to network failures, so if one path becomes unavailable, BGP finds an alternative route for traffic.

## Layering

- To manage complexity of modern networks, we use a structured approach called layering in whicih we break down the complex task of network communication into a series of smaller, more managable part or layers. Each layer has a specific responsibility and provides a service to the layer directly below it while relying on the services of the layer below. Is good because:
  - Allows for modular design, makes system easier to maintain and update as a change in the implementation of one layer's service such as updating a Wi-Fi protocol in link layer does not require changes to the other layer like the web browser in applicaion layer.
  - Provides an explicit structure which makes troubleshooting much more strainghtforward.
  - By defining standard interfaces between layers different systems and vendors can create products that work together effectively.

### Internet protocol Stack: Five layer model

- The five layer protocol stack of the internet is also called the TCP/IP model.

| Layers |
| ------------- |
| Application Layer |
| Transport Layer |
| Network Layer |
| Link Layer |
| Physical Layer |

#### Physical layer

- lowest layer of the protocol stack, deals with the physical connection between two network devices.
- Function: Its primary job is to transmit raw bits over a physical medium. It defines the physical and electrical specifications for the devices, including voltage levels, data rates, and types of connectors and cables used (eg: copper wire, fiber optic, and radio waves).
- Protocols: Specifications for ethernet cables, DSL, and encoding schemes used to turn bits into electrical or light signals.

#### Link layer

- responsible for the reliable transfer of data between two adjacent nodes connected on the same physical network.
- Function : Takes packets from network layer and encaptulates them into units called **frames**. Adds a header containing the physical (MAC) address of the source and destination devices on the local networks. Is also responsible for error detection and correction for data transmitted within a single network segment.
- Protocols: Ethernet, Wi-fi, Point to point protocol (PPP)

#### Network layer

- manages the delivery of data packets from the original source host to the final destination host, which may be on different networks across the globe
- Function: responsible for logical addressing and routing, assigns source and destination **IP addresses** to data segments to create packets (also called datagrams) and determines the best path for these packets to travel through the internetwork of routers
- Protocols: most important protocol at this layer is the **Internet protocol**. Routing protocols like the **Border Gateway Protocol (BGP)** also operate here to exchange routing information between networks.

#### Transport Layer

- ensures that data is transferred between the correct processes on the source and destination hosts
- Function : takes data from the application layer and breaks it down into smaller pieces called **segments**. adds a header with source and destination port numbers to ensure the data reaches the right application. handles error control, flow control, and ensures that data arrives in the correct sequence.
- Protocols: primarily two transport protocols, TCP which is connection oriented and provides reliable, ordered and error checked delivery of a stream of bytes, used for web browsing and email. UDP which is connectionless and offers faster low overhead data delivery but does not guarantee reliability or ordering, used for application like video streaming and online gaming where speed is more critical.

#### Application layer

- top layer and is the one that end-users and their applications interact with most directly
- Function: provides the protocols that allow software to send and receive information and present meaningful data to users, handles data representation and encoding.
- Protocols: has wide variety of protocols, like HTTP, FTP, SMTP, DNS.

### Encapsulation

The process by which data moves down through layers of the sending device is called encapsulation, at each layer a header (and sometimes a trailer) is added to the data from the layer above it.

1.  **Application Layer**: A user's application creates a **message** (e.g., an email).
2.  **Transport Layer**: The message is passed to the transport layer, which adds a TCP or UDP header to create a **segment**.
3.  **Network Layer**: The segment is passed to the network layer, which adds an IP header containing source and destination IP addresses, creating a **packet** or **datagram**.
4.  **Link Layer**: The packet is passed to the link layer, which adds a header and trailer with MAC addresses, creating a **frame**.
5.  **Physical Layer**: The frame is converted into bits and transmitted over the physical medium.

- When the data reaches the destination device, the reverse process, known as **decapsulation**, occurs.




