# ⚙️ Systems Programming in C — Week 6

## 📚 Lecture Notes: Inter-Process Communication (Pipes, FIFOs)

### Anonymous Pipes (`pipe()`)

* A **pipe** is a unidirectional communication channel between two related processes (usually parent and child).
* Created with `int pipefd[2]; pipe(pipefd);`

  * `pipefd[0]` = read end, `pipefd[1]` = write end.
* Typically, one process closes the write end (after writing), the other closes the read end (after reading).

---

### Named Pipes (FIFOs, `mkfifo`)

* A **FIFO** (First-In, First-Out) or **named pipe** appears as a special file in the filesystem.
* Created using `mkfifo("fifo_file", 0666);`
* Enables communication between unrelated processes (not just parent/child) via the file system.

---

### Chaining Processes with Pipes (Simple Shell Pipelines)

* Shells use pipes to chain processes (e.g., `ls | grep foo`).
* Each process reads/writes to the correct end of a pipe, so output from one is input to another.
* In C: use `fork()`, `pipe()`, and `dup2()` to set up stdin/stdout redirection before calling `exec()`.

---

### Why It Matters (CNO/RE Context)

* Pipes/FIFOs are used for stealthy CNO comms, process control, and lateral movement tools.
* Understanding IPC lets you debug complex malware or implement command-and-control channels.

---

## 🧠 Key Concepts To Know

* How a pipe enables communication between parent and child (or unrelated, with FIFO)
* The difference between anonymous pipes and named pipes
* Use of `dup2()` for shell pipelines
* Real-world bugs: deadlocks, leaks if not closing unused pipe ends

---

## ❓ Essential Questions

* What happens if you forget to close the unused ends of a pipe?
* Why can’t you use an anonymous pipe for communication between two completely unrelated processes?
* How do named pipes differ from regular files? What happens if two writers use the same FIFO?

---

## 🧪 Lab: One-Way IPC with Pipe

### **Instructions:**

1. **Write a C program:**

   * Parent process creates a pipe.
   * Parent forks a child.
   * Parent writes a message to the pipe.
   * Child reads the message from the pipe and prints it.

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <string.h>

   int main() {
       int fd[2];
       if (pipe(fd) == -1) {
           perror("pipe failed");
           exit(1);
       }
       pid_t pid = fork();
       if (pid == 0) { // Child
           close(fd[1]); // Close unused write end
           char buf[100];
           ssize_t n = read(fd[0], buf, sizeof(buf));
           if (n > 0) {
               buf[n] = '\0';
               printf("Child received: %s\n", buf);
           }
           close(fd[0]);
       } else if (pid > 0) { // Parent
           close(fd[0]); // Close unused read end
           char *msg = "Hello from parent!";
           write(fd[1], msg, strlen(msg));
           close(fd[1]);
       } else {
           perror("fork failed");
       }
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o pipe_ipc lab6.c
   ./pipe_ipc
   ```

3. **For named pipes (bonus):**

   * Create a FIFO with `mkfifo myfifo`.
   * In one terminal: `cat < myfifo`
   * In another: `echo 'hello from FIFO' > myfifo`

---

### **Expected Observables:**

* Child prints the message sent by parent via pipe.
* No deadlocks; unused pipe ends are properly closed.
* (Bonus) Named pipe/FIFO works between unrelated shells.

---

## 📖 Further Reading

* [Beej’s Guide: Pipes and FIFOs](https://beej.us/guide/bgipc/html/split/pipes.html)
* [Linux man page: pipe()](https://man7.org/linux/man-pages/man2/pipe.2.html)
* [Linux man page: mkfifo()](https://man7.org/linux/man-pages/man3/mkfifo.3.html)
* [Understanding Shell Pipelines (LWN.net)](https://lwn.net/Articles/237922/)

---

*Document your code, terminal output, and a summary of how data flowed between processes.*
