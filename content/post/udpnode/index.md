+++
author = "Hussam Al-Hertani"
title = "UDPNode: A C++ class for UDP Communication"
date = "2024-08-17"
draft = false
description = "UDPNode"
[params]
  math = true
tags = [
    "C++",
    "C"
]
categories = [
    "C++",
    "C"
]
+++

A couple of months ago, I decided to pursue few IOT project ideas that could benefit from basic UDP (User Datagram Protocol) communication. The MQTT and CoAP protocols are definitely more robust and secure. But there are some situations where the basic UDP protocol is sufficient. The POSIX C socket API built into Linux is quite complete; if a bit raw. And understanding said socket API; even with good command of the C programming language, can a bit daunting. Thankfully  [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/) does a great job of covering all aspects of socket programming. So I decided to embark on a journey of learning Linux Network / socket programming. My goal was to wrap just enough socket programming functionality within a C++ class to facilitate basic UDP communication. The result was the [UDPNode class](https://www.github.com/halherta/udpnode). 

### The UDPNode class

The UDPNode class facilitates UDP transmission. When a UDPNode object is instantiated, it's bound to an IP address and port on which it listens. A receive event loop is initialized as a thread. The purpose of the receive event loop is to capture datagrams sent from other UDPNodes and place them in a receive queue. The received datagrams can be extracted from the receive queue at a later time for processing. The same object can also transmit a datagram to other IP address and port combinations. All received datagrams must adhere to a specific structure that is serialized / deserialized in a JSON format.

This datagram structure is shown below:

```cpp
struct rxDatagram{
    unsigned int srcport;   // Source port number.
    std::string srcipaddr;  // Source IP address.
    time_t time_stamp;      // Timestamp of the received datagram.
    std::string msg;        // Message content.
    unsigned int crc_checksum;  // CRC checksum of the message.
    bool jointhread;        // Flag to indicate if the thread should join.
};
```

The `msg` string is the actual UDP message that's being pushed around. Each UDPNode can receive and transmit these datagrams. Here's an example of an application that primarily listens for datagrams:

```cpp
#include <iostream>
#include "UDPNode.h"


int main(void){

    UDPNode node(3490, ipv6,1024,5,true);
    node.startRxLoop();
    usleep(10000000);
    if(node.rxDataAvailable()){
        unsigned int datagramsavailable = node.rxDataQueueSize();
        for( int i = 0 ; i < datagramsavailable ; i++){
            node.printDatagram(node.readRxDatagramFromQueue());
            usleep(100000);
        }
    }
    node.endRxLoop();
    return 0;
}
```

The above program instantiates a UDP node object on port 3490, using IPv6. It sets the maximum message length to 1024 bytes, the maximum message queue size to 5 and enables debug messages. The program starts the receive thread and waits 10 seconds. It then prints the contents of all datagrams available in the receive queue. Finally, it stops the receives thread before exiting the program.

Here's another program that transmits datagrams:

```cpp
#include <iostream>
#include <unistd.h>
#include "UDPNode.h"


int main(void){
    UDPNode node(5590, ipv6, 1024, 5,true);
    
    node.tx(3490, ipv6,"::1","Attitude is the \"little thing\" that makes a big difference");
    node.tx(3490, ipv6,"::1","Life is too short to spend another day at war with yourself");
    node.tx(3490, ipv6,"::1","The best time to plant a tree was 20 years ago. The second best time is now");
    node.tx(3490, ipv6,"::1","Be happy for this moment. This moment is your life");
    node.tx(3490, ipv6,"::1","A good laugh and a long sleep are the two best cures for anythin");
    node.tx(3490, ipv6,"::1","The sun is a daily reminder that we too can rise again from the darkness, that we too can shine our own light");
    node.tx(3490, ipv6,"::1","Today is a good day to try");
    node.tx(3490, ipv6,"::1","The only person you are destined to become is the person you decide to be");
    
    return 0;
}
```

This program instantiates a UDP node object that listens on port 5590, using IPv6. It sets the maximum message length to 1024 bytes, the maximum message queue size to 5 and enables debug messages. It then sends 8 messages to the listening UDP node on port 3490. If you compile both programs above and run them, you'll witness how they interact with each other. Be sure to check out the [github repository](https://www.github.com/halherta/udpnode) for compilation instructions.


This class has not been rigourously tested, so feel free to submit pull requests for adding functionality or fixing bugs!