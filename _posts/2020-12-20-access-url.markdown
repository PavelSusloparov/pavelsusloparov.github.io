---
layout: post
title: "What happens when you type URL in the browser."
date:   2020-12-20 15:15:36 -0500
categories: networking
---

Simple at first, the topic brings a lot of depth when we speak about each component.

On the high-level:
1. Enter URL to the browser window.
2. The browser looks up the domain name in Domain Name Server(DNS).
3. The browser initiates a TCP connection to the server.
4. The browser sends an HTTP request to the server.
5. The server handles the request and sends the HTTP response.
6. The browser displays HTML content.

So far, so good. Let's talk more about each category.

# Enter an URL to the browser window

## From the keypress

When you press your keyboard key, the key presses a switch, completing the circuit behind the key.
The key matrix is a grid of circuits underneath the keys.
Completing the circuit allows a tiny amount of information to flow through to the keyboard processor.
The keyboard processor task is to compare the circuit's location on the key matrix to the character map in its read-only memory.

The character map is a lookup table.
In addition to simple lookups, such as 'a',
the lookup table has combinations such as <Shift> + <>>, which results as a '.' in the browser URL.

Keep in mind that there are various keyboard types, which changes how the processor identifies a keypress.
Learn more about keyboards types [here](https://en.wikipedia.org/wiki/Computer_keyboard)

## From keyboard processor to screen

The signal has to carry to the screen next.
Laptops use internal connectors. Standalone keyboards use Bluetooth or USB or PS/2.
Next, the keyboard controller analyses the incoming signal and determines it is a system level command or not.
For our use case, it is not, and the controller determines the signal as content.
The browser can work with content signals to see the URL appearing in the browser window while typing.

# The browser looks up the domain name in Domain Name Server(DNS).

The browser has to resolve the human-readable hostname to the IP address.
For that, it needs a lookup table, so it tries to find it in multiple places.
If the entry exists, it knows the IP address right away and does not proceed to the next step.

1. It goes to the browser cache.
2. It goes to the OS local cache.
    - Browser reaches the local cache through API to `local resolver library,` which is part of OS.
3. It goes to router cache.
4. It goes to the local DNS server.
    - It is configured on the client machine. An example of a local DNS is [Google Public DNS](https://en.wikipedia.org/wiki/Google_Public_DNS).
Or, it is your internal service provider, like Comcast. And yes, they know about all websites which you visit.
5. From the local DNS, it goes to the root server.
    - This is a high-level domain server, like .com or .io
6. Root server responds to local DNS with the authoritative name server, which knows about the given hostname.
7. The authoritative name server responds with the end server's exact IP address back to the local DNS server.
8. Local DNS server calls the end server to verify and get the IP address back.
9. Local DNS responds to the client machine with the IP address of the end server.

# The browser initiates a TCP connection to the server.

## Protocols intro

Great, the browser knows the address of the end server. Now it wants to send a request to it to get a response with HTML content back.
However, before that, the client machine needs to establish a connection to the server.

The most common server-to-server communication protocol is *HyperText Transfer Protocol*(HTTP).
It knows how to send objects, like HTML files.
HTTP is an `application layer` protocol; it relies on the Transport layer protocol to carry packets through the network.

The most common `transport layer` protocol is *Transmission Control Protocol*(TCP).
It guarantees message delivery in order.

The most common use for our use case is a persistent connection.
In short, the browser opens a connection, send data, receives the response 
and keeps the connection open while some requests are coming from the browser.

## Establish the connection

The most popular way to establish a TCP connection is the Three-Way Handshake.

1. The client sends a TCP segment with the SYN flag set, and a sequence number is a random number in TCP segment headers(not HTTP headers :) ).
2. The server receives the TCP segment and sends a TCP segment back with SYN and ACK flags set, and a sequence number is a random number in TCP segment headers.
3. The client sends a TCP segment with the ACK flag set, and the sequence number is `sequence_number + 1` from the SYN-ACK TCP segment.

There are other ways, like [Finite State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) or `Simultaneous Connection Establishment.` 

# The browser sends an HTTP request to the server.

TCP protocol has headers for each TCP segment of information, which essentially metadata to get the request assembled on the receiver side.
Read more about TCP [here](https://en.wikipedia.org/wiki/Transmission_Control_Protocol)

Then `network layer` determines how to transfer the packet with information to the end server through a chain of routers.

There are multiple algorithms to keep routers lookup tables up-to-date about their neighbors.
Read more about the link-state routing protocol [here](https://en.wikipedia.org/wiki/Link-state_routing_protocol).
In short, it updates each router lookup table about it's neighbors and then, using Dijkstra's algorithm, finds the shortest path in a graph.

Distance-vector routing protocol [here](https://en.wikipedia.org/wiki/Distance-vector_routing_protocol)
In short, it is based on the Bellman-Ford algorithm,
which through the multiple broadcast repetitive actions update lookup tables based on the neighbor's lookup tables.

Then `datalink layer` needs to frame packets based on the type of the network you use, such as WiFi or LAN, or Ethernet.
Also, it provides control over the `physical layer,` which sends binary data.

With a sequence of hops through the routers, packets reach the server.

On the server-side, the layers go in reverse.
The `transport layer` labels packets(multiplexing), so the server can put packets together(de-multiplexing).

# The serve handles the request and sends an HTTP response.

It all depends on the server implementation.
It can be a standalone machine. It can be a set of machines, which play different roles,
such as Global Load Balancer, Regional Load Balancer, Application server, Database, and other application servers.
All of the listed components communicate through TCP and can be located in different data centers, essentially locations.
The communication process is very similar to what I described above.
Eventually, the application server does defined business logic and responds with the HTML object.

# The browser displays HTML content.

The browser shows the HTML object on the screen.
It needs to know how to interpret the set of HTML tags to visual representation.

## Data Object Model (DOM)

The server returns an HTML page in binary stream format with HTTP headers, which define metadata for the HTML object.
With headers help, the browser interprets the binary stream to readable text.

Then the browser creates a JavaScript object *Node* for every tag, such as <html>, <body>, <div>.
Also, it knows the relation between those Nodes' objects.

The relationships look like a tree structure. It is called the *Data Object Model*(DOM).
JavaScript does not understand what DOM is. It is not part of the JavaScript specifications.
DOM is a high-level WEB API, which provides capabilities to manipulate with it.

## Cascading Style Sheet (CSS)

HTML object can contain CSS. It gives property to an HTML tag.

The browser needs to apply a CSS style to a DOM element.
It creates *CSS Object Model*(CSSOM), which has similar to DOM tree structure.
There is a default version of the CSSOM based on the `user agent stylesheet,` which every browser has.
More about `user agent stylesheet` [here](https://www.w3.org/TR/CSS21/cascade.html#cascade).

If there is a custom CSS, it overrides the default value.

## Render a tree

Then browser combines DOM and CSSOM trees through the `Render-Tree` process.

## Paint

Browser prints the result of the `Render-Tree` process with each element.

1. It creates a layout.
    - The layout knows each node's size in pixels and position, which the element needs to be printed on the screen.
2. It creates layers for each element in the Render-Tree.
3. It paints layouts by drawing them on the screen.
    Inside each layer, the browser paints each pixel based on the layer property(coming from CSS). 
    The process is called `rasterization.`
    
## Compositing

This step draws the result of the paint to the screen.
In this operation, all layers are sent to the Graphics Processing Unit(GPU) to draw them on the screen.

# Outro

`google.com` opens in 400ms.
This period of time transfers 86kb of data and makes 11 hops to transfer each packet of information to the server.
Try `traceroute google.com` in your terminal to check how many hops you have.

Thank you for reading!
