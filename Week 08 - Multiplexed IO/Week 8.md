# ⚙️ Systems Programming in C — Week 8

## 📚 Lecture Notes: Multiplexed I/O (`select`, `poll`, `epoll`)

### Why Multiplex? The C10K Problem

* **Multiplexed I/O:** A technique that lets a program monitor many file descriptors (FDs) at once and act when any become "ready" (readable/writable).
* **The C10K Problem:** How do you efficiently handle 10,000+ simultaneous connections in one process? Blocking on one FD per process/thread doesn't scale.
* **Key idea:** Use a single event loop to handle multiple connections, minimizing the overhead of threads/processes.

---

### `select()`, `poll()`, and `epoll()`

* **select(fd\_set, ...):** Classic, portable; can monitor up to FD\_SETSIZE FDs. Fills sets for read/write/exception events. Can be inefficient for large FD sets.
* **poll(struct pollfd\[], ...):** Similar idea but with an array, not bitmasks. Handles more FDs, simpler API for dynamic sets.
* **epoll:** Linux-specific, scales to thousands of FDs. Uses a kernel-managed event list, best for high-performance servers.

---

### Non-Blocking I/O and Readiness

* **Non-blocking I/O:** A file descriptor is set to non-blocking mode (using `fcntl`). Reads/writes return immediately (may fail if not ready).
* **Readiness:** Multiplexing APIs (select, poll, epoll) report when FDs are ready for I/O without blocking. Allows single-threaded event-driven programming.

---

### Why It Matters (CNO/RE Context)

* Multiplexed I/O is used in high-performance servers, malware C2 channels, keyloggers, and reverse shells.
* RE analysts often encounter event loops and must understand how multiplexing influences code flow, concurrency, and detection.

---

## 🧠 Key Concepts To Know

* What an FD set is and how `select()` operates on it
* Limitations of `select()` (FD\_SETSIZE, O(n) scan)
* Differences between select, poll, and epoll (tradeoffs)
* How to set an FD to non-blocking and why

---

## ❓ Essential Questions

* Why not just use one thread/process per connection?
* How does `select()` notify your program which FD is ready?
* What happens if you don't handle all ready FDs promptly?
* Where are race conditions and security bugs possible in event loops?

---

## 🧪 Lab: Simple TCP Server Using select() for Multiple Clients

### **Instructions:**

1. **Write a C server:**

   * Listens on a TCP port (e.g., 9090).
   * Accepts multiple clients and echoes data back to each.
   * Uses `select()` to monitor server socket (for new connections) and all active client sockets (for incoming data).
   * Handles multiple clients in a single process (no threads/forks).

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <unistd.h>
   #include <arpa/inet.h>
   #include <sys/socket.h>
   #include <sys/select.h>

   #define PORT 9090
   #define MAX_CLIENTS 10

   int main() {
       int listen_fd, new_fd, client_fds[MAX_CLIENTS], max_fd, activity, i;
       struct sockaddr_in addr;
       char buf[1024];
       fd_set readfds;
       listen_fd = socket(AF_INET, SOCK_STREAM, 0);
       addr.sin_family = AF_INET;
       addr.sin_addr.s_addr = INADDR_ANY;
       addr.sin_port = htons(PORT);
       bind(listen_fd, (struct sockaddr*)&addr, sizeof(addr));
       listen(listen_fd, 5);
       for (i = 0; i < MAX_CLIENTS; i++) client_fds[i] = 0;
       while (1) {
           FD_ZERO(&readfds);
           FD_SET(listen_fd, &readfds);
           max_fd = listen_fd;
           for (i = 0; i < MAX_CLIENTS; i++) {
               if (client_fds[i] > 0) FD_SET(client_fds[i], &readfds);
               if (client_fds[i] > max_fd) max_fd = client_fds[i];
           }
           activity = select(max_fd + 1, &readfds, NULL, NULL, NULL);
           if (FD_ISSET(listen_fd, &readfds)) {
               new_fd = accept(listen_fd, NULL, NULL);
               for (i = 0; i < MAX_CLIENTS; i++) {
                   if (client_fds[i] == 0) {
                       client_fds[i] = new_fd;
                       break;
                   }
               }
           }
           for (i = 0; i < MAX_CLIENTS; i++) {
               int sd = client_fds[i];
               if (sd > 0 && FD_ISSET(sd, &readfds)) {
                   int n = read(sd, buf, sizeof(buf));
                   if (n <= 0) {
                       close(sd); client_fds[i] = 0;
                   } else {
                       write(sd, buf, n);
                   }
               }
           }
       }
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o select_server lab8.c
   ./select_server
   ```

3. **Connect multiple clients (in separate terminals):**

   ```sh
   nc localhost 9090
   # Type messages, see them echoed back
   ```

---

### **Expected Observables:**

* Multiple clients can connect to the server at once
* Each client’s input is echoed independently
* Only a single server process is running (use `ps`, `lsof`)

---

## 📖 Further Reading

* [Beej’s Guide: select() and poll()](https://beej.us/guide/bgnet/html/split/select.html)
* [Linux man page: select()](https://man7.org/linux/man-pages/man2/select.2.html)
* [Linux man page: epoll()](https://man7.org/linux/man-pages/man7/epoll.7.html)
* [The C10K Problem](https://www.kegel.com/c10k.html)
* [Event-driven programming in C (IBM)](https://developer.ibm.com/articles/au-advancedsocket/)

---

*Document your code, testing with multiple clients, and any issues or performance observations you encounter.*
