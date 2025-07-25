# ⚙️ Systems Programming in C — Week 9

## 📚 Lecture Notes: Intro to Sockets and Networking

### Socket Concepts (AF\_INET, SOCK\_STREAM, SOCK\_DGRAM)

* **Socket:** An endpoint for network communication between two machines/processes.
* **Domains:**

  * `AF_INET` (IPv4 Internet)
  * `AF_INET6` (IPv6)
* **Types:**

  * `SOCK_STREAM` (TCP, reliable, connection-oriented)
  * `SOCK_DGRAM` (UDP, unreliable, connectionless)
* Sockets provide a standard way to send/receive data over the network, using file descriptor APIs.

---

### Core System Calls: socket(), bind(), listen(), accept(), connect()

* **socket(domain, type, protocol):** Create a new socket and return its FD.
* **bind(fd, addr, addrlen):** Assign a local address (and port) to a socket.
* **listen(fd, backlog):** Mark socket as a passive (server) socket, ready to accept connections.
* **accept(fd, ...):** Accept an incoming connection, returns a new socket FD for that connection.
* **connect(fd, addr, addrlen):** Client-side: connect socket to remote address/port.

---

### Simple TCP Echo Server/Client

* **Server:**

  * Creates a socket, binds it to an address/port, listens, and accepts clients.
  * For each client, reads data and writes it back (echo).
* **Client:**

  * Creates a socket, connects to the server, sends data, and prints response.

---

### Why It Matters (CNO/RE Context)

* Almost all CNO, malware, and RE tooling uses sockets for C2, exfiltration, scanning, and lateral movement.
* Understanding sockets is foundational for all modern network programming, both offensive and defensive.

---

## 🧠 Key Concepts To Know

* What each socket function does and its typical arguments
* How TCP differs from UDP in setup and reliability
* How sockets relate to file descriptors
* How to troubleshoot common socket errors (address in use, connection refused, etc.)

---

## ❓ Essential Questions

* What is the difference between server and client socket setup?
* What happens when you connect two sockets to the same port?
* How do you cleanly close sockets in C?
* How can a simple echo server become vulnerable to exploitation?

---

## 🧪 Lab: Build a Basic TCP Echo Server and Client

### **Instructions:**

1. **Write a C TCP echo server:**

   * Binds to `127.0.0.1:8888` (localhost, port 8888).
   * Accepts a single client, echoes data back until client closes connection.

   Example server skeleton:

   ```c
   // server.c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <unistd.h>
   #include <arpa/inet.h>

   int main() {
       int sockfd, clientfd;
       struct sockaddr_in addr;
       char buf[1024];
       sockfd = socket(AF_INET, SOCK_STREAM, 0);
       addr.sin_family = AF_INET;
       addr.sin_addr.s_addr = htonl(INADDR_ANY);
       addr.sin_port = htons(8888);
       bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
       listen(sockfd, 1);
       printf("Server listening on port 8888\n");
       clientfd = accept(sockfd, NULL, NULL);
       printf("Client connected!\n");
       while (1) {
           int n = read(clientfd, buf, sizeof(buf));
           if (n <= 0) break;
           write(clientfd, buf, n);
       }
       close(clientfd);
       close(sockfd);
       return 0;
   }
   ```

2. **Write a C TCP echo client:**

   * Connects to `127.0.0.1:8888`.
   * Reads lines from stdin, sends to server, prints server’s reply.

   Example client skeleton:

   ```c
   // client.c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <unistd.h>
   #include <arpa/inet.h>

   int main() {
       int sockfd;
       struct sockaddr_in addr;
       char buf[1024];
       sockfd = socket(AF_INET, SOCK_STREAM, 0);
       addr.sin_family = AF_INET;
       addr.sin_addr.s_addr = inet_addr("127.0.0.1");
       addr.sin_port = htons(8888);
       connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));
       printf("Connected to server. Type messages...\n");
       while (fgets(buf, sizeof(buf), stdin)) {
           write(sockfd, buf, strlen(buf));
           int n = read(sockfd, buf, sizeof(buf));
           if (n <= 0) break;
           buf[n] = '\0';
           printf("Echo: %s", buf);
       }
       close(sockfd);
       return 0;
   }
   ```

3. **Compile and run:**

   ```sh
   gcc -o server server.c
   gcc -o client client.c
   ./server
   # In another terminal
   ./client
   ```

---

### **Expected Observables:**

* Server prints “Client connected!” when client connects
* Messages typed in the client are echoed back by the server
* Both programs exit cleanly on EOF (Ctrl+D)

---

## 📖 Further Reading

* [Beej’s Guide: Sockets (Ch 5–6)](https://beej.us/guide/bgnet/html/split/clientserver.html)
* [Linux man page: socket()](https://man7.org/linux/man-pages/man2/socket.2.html)
* [Linux man page: bind()](https://man7.org/linux/man-pages/man2/bind.2.html)
* [Linux man page: accept()](https://man7.org/linux/man-pages/man2/accept.2.html)
* [TCP vs UDP Explained (Cloudflare)](https://www.cloudflare.com/learning/ddos/glossary/transmission-control-protocol-tcp/)

---

*Document your code, test output, and any issues encountered exchanging messages between server and client.*
