# ⚙️ Systems Programming in C — Week 5

## 📚 Lecture Notes: Signals and Signal Handling

### What are Signals?

* **Signals** are asynchronous notifications sent to a process by the kernel or another process to notify it of events (like interrupts, segmentation faults, or child process termination).
* Used for inter-process communication, process control, and event notification.
* Each signal has a unique number. View all with:

  ```sh
  kill -l
  ```

---

### Common Signals

* **SIGINT (2):** Interrupt from keyboard (Ctrl+C)
* **SIGSEGV (11):** Segmentation fault (invalid memory access)
* **SIGCHLD (17/20/18):** Child process has stopped or terminated
* **SIGTERM (15):** Termination signal (default for `kill`)
* **SIGKILL (9):** Kill signal (cannot be caught/ignored)

---

### Catching and Ignoring Signals: `signal()`, `sigaction()`

* **signal(signum, handler):** Simple way to set a signal handler. Portable but limited.
* **sigaction(signum, struct sigaction, oldact):** Preferred, more flexible and reliable. Allows more control (e.g., restartable syscalls).
* **Catching a signal:** Register a function (handler) to run when a signal is received.
* **Ignoring a signal:** Set handler to `SIG_IGN`.

---

### Why It Matters (CNO/RE Context)

* Signal handling is used for malware persistence (e.g., ignoring termination), hiding, or controlling processes.
* Exploit and shellcode writers often use/abuse signals for privilege escalation or crash handling.
* Signal bugs (double-free on SIGSEGV, unhandled SIGCHLD) can create vulnerabilities.

---

## 🧠 Key Concepts To Know

* Difference between synchronous (e.g., SIGSEGV) and asynchronous (e.g., SIGINT) signals
* Which signals can/cannot be caught or ignored
* How signals are queued and delivered by the kernel
* Impact of signal handlers on program flow and security

---

## ❓ Essential Questions

* What happens if a signal is sent but no handler is set?
* How do you ensure your handler is safe (reentrant, minimal side effects)?
* What is the difference between `signal()` and `sigaction()`?
* Why is handling `SIGCHLD` important in multi-process programs?

---

## 🧪 Lab: Handling Ctrl+C and Segmentation Faults

### **Instructions:**

1. **Write a C program:**

   * Registers handlers for `SIGINT` (Ctrl+C) and `SIGSEGV` (segfault)
   * On `SIGINT`, prints a message and exits cleanly
   * On `SIGSEGV`, prints an error and exits (bonus: try to recover)
   * Main loop: waits for input (e.g., with `getchar()` or a sleep)

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <signal.h>
   #include <unistd.h>

   void sigint_handler(int signum) {
       write(1, "Caught SIGINT (Ctrl+C)! Exiting...\n", 33);
       exit(0);
   }
   void sigsegv_handler(int signum) {
       write(2, "Segmentation fault caught!\n", 26);
       exit(1);
   }
   int main() {
       signal(SIGINT, sigint_handler);
       signal(SIGSEGV, sigsegv_handler);
       printf("Running. Press Ctrl+C or trigger a segfault.\n");
       getchar(); // Wait for input
       // Uncomment to test segfault:
       // int *p = NULL; *p = 1;
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o siglab lab5.c
   ./siglab
   ```

3. **Test:**

   * Press Ctrl+C: should print message and exit.
   * Uncomment segfault line, rerun: should print segfault message and exit.

---

### **Expected Observables:**

* Pressing Ctrl+C triggers handler, prints message, program exits cleanly
* Causing a segmentation fault triggers handler, prints error message
* Program does not terminate with default OS message if handlers are set

---

## 📖 Further Reading

* [Beej’s Guide: Signals](https://beej.us/guide/bgipc/html/split/signals.html)
* [Linux man page: signal()](https://man7.org/linux/man-pages/man2/signal.2.html)
* [Linux man page: sigaction()](https://man7.org/linux/man-pages/man2/sigaction.2.html)
* [The Linux Programming Interface: Signals (Sample)](https://man7.org/tlpi/)
* [Signal Safety and Pitfalls (LWN.net)](https://lwn.net/Articles/248911/)

---

*Document your code, terminal output, and what you learned about custom signal handlers.*
