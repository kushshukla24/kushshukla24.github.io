---
title: "Simple Web Server"
description : "Description goes here..."
date: 2024-01-01T20:31:41+05:30
tags: ["web", "server"]
draft: false
---
High level programming languages of today made it pretty easy to develop web applications on servers. But, it is always good to know how things works under the hood. Though, abstractions are good for productivity, they help you to do the work quickly and get to the Minimum Viable Product/Prototype (MVP) of your desired system running. However, when it comes to extracting every ounce of performance from the system, there is no substitute for knowing the fundamentals, understanding the system inside-out and ultimately exploiting the system for the use-case. In this post, we will delve deeper into a simple web server to understand the intricacies of this machinery. 

## What is a web server?
Web Server is a software programs that runs on a computer and listens to incoming requests from clients and responds to them. The clients can be a web browser, a mobile app or any other application that can make HTTP requests. The web server is responsible for handling the requests and sending back the response to the client. 

For our case here, we will be building a simple web server on an Ubuntu Linux machine that can handle a HTTP request, process it and send back the HTTP response. There will no asynchronous processing, no multi-threading, no caching, no load balancing, no security, no authentication, no authorization, no database, no file system, no logging, no monitoring, no metrics, no nothing. It will be a simple web server that can handle a single request at a time. This simplicity will help us to understand the fundamentals of a web server.

## What is a File Descriptor?
A file descriptor is a unique identifier or an integer that the operating system assigns to an open file or I/O (input/output) stream. In Unix-like operating systems, including Linux, file descriptors are used to represent access to files, sockets, pipes, and other input/output resources.

When a program creates a new file descriptor, the operating system assigns a non-negative integer as the file descriptor. The program can then use this integer to refer to the associated file or I/O stream. File descriptors are a fundamental concept in Unix-like systems and are used extensively in system calls and low-level I/O operations. A negative file descriptor is used to indicate an error.

For our web-server, we will accessing the socket file descriptor to read and write data.

## What is a Socket?
A socket is an endpoint of a two-way communication link between two programs running on the network. A socket is bound to a port number so that the TCP Layer can identify the application that data is destined to be sent to. An endpoint is a combination of an IP address and a port number. Every TCP connection can be uniquely identified by its two endpoints.

## What is TCP?
TCP (Transmission Control Protocol) is a standard that defines how to establish and maintain a network conversation through which application programs can exchange data. TCP works with the Internet Protocol (IP), which defines how computers send packets of data to each other. Together, TCP and IP are the basic rules defining the Internet. TCP is defined by the Internet Engineering Task Force (IETF) in the Request for Comment (RFC) standards document number 793.

# Developing a Simple Web Server

1. Create a server-side socket.
    Socket is created by making a **system call** to the OS. The OS returns a file descriptor that can be used to read and write data to the socket. The socket is created by specifying the domain, type and protocol. 
    
    The domain specifies the communication domain. For our case, we will be using the AF_INET domain which is the IPv4 Internet Protocol. 
    The type specifies the communication semantics. For our case, we will be using the SOCK_STREAM type which is a reliable, two-way, connection-based byte stream. 
    The protocol specifies the protocol to be used with the socket. For our case, we will be using the TCP protocol. 
    
    The socket is created by making a system call to the OS as shown below.

    ```c
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("Socket failed");
        exit(EXIT_FAILURE);
    }
    ```

2. Bind the socket to an address.
    Socket Address belongs to a family, has a port number and an IP address. The binding of a socket to an address is done by making a *bind* **system call** to the OS. The OS returns a file descriptor that can be used to detect whether the bind system call succeeded/failed.

    In the code below, we are binding the socket '''server_fd''' to the address '''address''' which is of address family IPv4 (AF_INET) and listens to any IPv4 address on PORT.

    ```c
    struct sockaddr_in address;
    int address_len = sizeof(address);
	
    address.sin_family = AF_INET; //ipv4
    // this is listening on all interfaces (ethernet, wifi, 5G) and its a bad idea
    // because you are exposing your application to all the interfaces unintentionally
    address.sin_addr.s_addr = INADDR_ANY; // listen 0.0.0.0 interfaces 
    // PORT is an integer
    // byte order of integer is anything the host describes
    // anything on internet is network order 
    address.sin_port = htons(PORT); 

    // specific flag that allow you to listen to existing used port
    // SO_REUSEADDR is a socket option that tells the kernel to reuse a local socket in 
    // TIME_WAIT state, without waiting for its natural timeout to expire.
    // this will not listen, this is just a record
    if (bind(server_fd, (struct sockaddr *)&address, address_len) < 0) {
        perror("Bind failed");
        exit(EXIT_FAILURE);
    }
    ```

3. Listen for incoming connections.
    Now, that the socket is created and bound to an address, we can start listening for incoming connections. This is done by making a *listen* **system call** to the OS. The OS returns a file descriptor that can be used to detect whether the listen system call succeeded/failed.

    ```c
    // Creates a queue (accept, receive, send) of incoming connections
    // listen for clients with 10 backlog (10 connections in accept queue)
    if (listen(server_fd, 10) < 0) {
        perror("Listen failed");
        exit(EXIT_FAILURE);
    }
    ```

4. On an infinite loop
    1. Accept the incoming connection.
        ```c
        // Accept a client connection client_fd == connection
        // this blocks
        // if the accept queue is empty, we are stuck here.. 
        if ((client_fd = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&address_len)) < 0) {
            perror("Accept failed");
            exit(EXIT_FAILURE);
        }
        ```
    2. Read the data from the OS receive queue to the Application Buffer.
        ```c
        // read data from the OS receive buffer to the application (buffer)
        //this is essentially reading the HTTP request
        //don't bite more than you chew APP_MAX_BUFFER
        read(client_fd, buffer, APP_MAX_BUFFER);
        ```
    3. Write the data from the Application Buffer to the OS send queue.
        ```c
        //write to the socket
        //send queue os
        write(client_fd, http_response, strlen(http_response));
        ```
    4. Close the connection.
        ```c
        //close the client socket (terminate the TCP connection)
        close(client_fd);
        ```


With the above server code in execution, we can now make a HTTP request to the server and get the response back. The server will be able to handle only one request at a time. Lets understand the flow of the request and response.

1. Client makes a HTTP request to the server by specifiying the IP address and the port number.
2. The network identifies the target machine through the IP address.
3. Data from the Client request is copied to Network Interface Card (NIC) buffer of the server machine.
4. On the server machine, the kernel will finish the handshake, it will put Syn in sync queue and responds with the syn ack and then client will do ack, to establish a full fledged connection. The OS kepts the connection from the client on the accept queue.
5. The server application picks the incoming connection because the server socket is listening on the given port number from the accept queue. 
6. NIC copies the data from its buffer to the OS receive buffer.
7. The server application picks the request data from the recieve buffer and copies the data to the Application Buffer.
8. The server application processes the request and generates the response.
9. The server application copies the response data to the Application Buffer.
10. The server application copies the response data from the Application Buffer to the OS send buffer.
11. NIC copies the data from the OS send buffer to its buffer.  
12. NIC copies the data from its buffer to the Client machine.
13. The Server closes the connection.


3 copy operations are happening here on each direction.
NIC Buffer -> OS Receive Buffer -> Application Buffer 
Application Buffer -> OS Send Buffer -> NIC Buffer

Performance can be improved by reducing the number of copy operations. This can be done by using a technique called **Zero Copy**. In this technique, the data is copied directly from the NIC buffer to the Application Buffer and vice-versa. This is done by using a technique called **Memory Mapping**. 

Memory Mapping is a technique where the OS maps the NIC buffer to the Application Buffer. This way, the data is copied directly from the NIC buffer to the Application Buffer and vice-versa. This reduces the number of copy operations from 3 to 1.



