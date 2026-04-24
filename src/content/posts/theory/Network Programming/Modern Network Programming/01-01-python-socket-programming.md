---
title: '[Ch01 - 01] Python Socket Programming Basics'
published: 2026-04-24
description: 'Study notes on Python socket programming basics, including Socket API, TCP/UDP fundamentals, and client-server architecture.'
image: ''
tags: [Network Programming, Python, Socket, TCP, UDP, Theory]
series: 'Network Programming'
category: 'Theory'
draft: false
---

## What is Socket?
- A socket is an endpoint used to se
- A TCP connection (connection-oriented) is uniquely identified by:
  (source IP, source port, destination IP, destination port, protocol)
- UDP is connectionless, which means:
  + No handshake  
  + No guarantee of delivery
  + No ordering of packets

## Python socket programming
### 1. Socket API
  `socket()` : create socket
  ```python
  import socket
  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  ```
  `bind()` : Assign IP address and port to server
   ```python
  import socket
  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

  # bind() expects a tuple (host, port)
  s.bind(("0.0.0.0",9999)) 
  ```
  `listen()` : Put the socket into listening mode
  ```python
  import socket
  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

  s.bind(("0.0.0.0",9999))

  # the parameter of listen is called backlog
  s.listen(5) # Here, backlog = 5, it defines the maximum length of the pending connection queue (actual behavior depends on OS implementation)
  ```
  `accept()` : Accept a connection from a client
  ```python
  import socket
  s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

  s.bind(("0.0.0.0",9999))
  s.listen(5)
  
  # accept() blocks until a client connects
  # it returns a new socket for communication and the client's address (IP, port)
  client_socket, addr = s.accept()

  """
    + client_socket : the new socket used to send and receive data with that specific client
    + addr : the client's address, addr's format usually is (client_ip, client_port) 
  """
  ```
  `connect()` : establish a connection to a remote server (TCP only) (client side)
  ```python
  client.connect(("0.0.0.0", 9999))
  ```
### 2. Server vs Client
| Category              | Server                                              | Client                    |
| --------------------- | --------------------------------------------------- | ------------------------- |
| Role                  | Provides services                                   | Requests services         |
| Connection initiation | Passive (waits)                                     | Active (initiates)        |
| Core APIs             | `bind()`, `listen()`, `accept()`                          | `connect()`                 |
| Data exchange         | `recv()`/`recvfrom()`, `send()`/`sendto()`                                      | `send()`/`sendto()`, `recv()`/`recvfrom()`            |
| Concurrency           | Handles multiple clients (threading, async, select) | Usually single connection |
| State management      | Can be stateful or stateless                        | Usually stateless         |
| Port usage            | Binds to a fixed port                               | Uses ephemeral port       |
| Lifecycle             | Long-running process                                | Short-lived process       |
| Scalability           | Requires load balancing, optimization               | Not a concern             |
| Security risks        | DDoS, RCE, injection                                | MITM, spoofing            |
| Examples              | Web server, SSH server                              | Browser, curl             |

### 3. TCP server and client flow 
#### Server flow: 
  1. listens for incoming connections ( `listen()` )
  2. accepts a connection ( `accept()` )
  3. receives data
  4. sends response
  5. closes the connection
 
 Below is a representation of this process in Python:
  ```python
  import socket
  server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  server.bind(("0.0.0.0", 9999))
  server.listen(5)

  while True:
    client, addr = server.accept()

    data = client.recv(1024)
    client.send(b"Hello from server")

    client.close()
  ```
#### Client flow :
  1. establishes a connection ( `connect()` )
  2. sends data
  3. receives response
  4. closes the connection

 Below is a representation of this process in Python:
  ```python
  import socket

  client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  client.connect(("127.0.0.1", 9999))

  client.send(b"Hello server")
  data = client.recv(1024)

  client.close()
  ```
  

### 4. UDP server and client flow:
#### Server flow: 
  1. waits for incoming datagrams
  2. receives data
  3. sends response

 Below is a representation of this process in Python:
  ```python
  import socket
  
  server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
  server.bind(("0.0.0.0", 9999))

  while True:
    data, addr = server.recvfrom(1024)
    server.sendto(b"Hello", addr)
  ```
#### Client flow:
  1. sends datagram
  2. receives response

 Below is a representation of this process in Python:
  ```python
  import socket

  client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

  client.sendto(b"Hello", ("127.0.0.1", 9999))
  data, _ = client.recvfrom(1024)
  client.close()
  ```

#### UDP Note:
  1. `recvfrom(1024)`:
     - Specifies the maximum number of bytes to read
     - Receives a complete datagram
     - If the buffer is smaller than the datagram, the data may be truncated
     - Returns (data, address)
  2. UDP:
     - The server does not maintain prior knowledge of clients
     - The client does not establish a connection with the server
     - Communication is based on independent packets (datagrams)

### 5. Blocking behavior:
Blocking behavior applies to both TCP and UDP sockets. By default, sockets in Python operate in blocking mode.
  - `accept()` blocks until a connection arrives (TCP only)
  - `recv()` / `recvfrom()` block until data is received
  - `send()` / `sendto()` may block if the system buffer is full

#### Notes:
  - Blocking means the program pauses execution until the operation completes
  - This behavior is controlled by the socket configuration, not the protocol (TCP or UDP)
  - Both TCP and UDP sockets can be configured as blocking or non-blocking
  - Each `recvfrom()` call corresponds to a single datagram