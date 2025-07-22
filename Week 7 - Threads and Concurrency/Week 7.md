# ⚙️ Systems Programming in C — Week 7

## 📚 Lecture Notes: Threads and Concurrency

### What are Threads?

* **Thread:** A lightweight unit of execution within a process. Threads share memory, code, and file descriptors, but have their own stack and CPU state.
* **Multithreading:** Running multiple threads in a single process. Used to handle multiple tasks simultaneously or to improve performance on multicore systems.
* **Why use threads?** Parallelism (CPU-bound), responsiveness (I/O-bound), and to simplify complex asynchronous logic.

---

### pthreads (POSIX Threads)

* The standard threading API on Unix-like systems.
* Key functions:

  * `pthread_create()` — Start a new thread.
  * `pthread_join()` — Wait for a thread to finish.
  * `pthread_exit()` — Terminate a thread.
* All threads in a process can access global and heap memory (danger: race conditions!).

---

### Race Conditions, Mutexes, and Condition Variables

* **Race condition:** Two or more threads access shared data at the same time, and the result depends on the timing. Leads to bugs and security holes.
* **Mutex (mutual exclusion):** Used to lock critical sections so only one thread can access shared data at a time. Key functions:

  * `pthread_mutex_init()`, `pthread_mutex_lock()`, `pthread_mutex_unlock()`
* **Condition variable:** Allows threads to wait (block) until a certain condition is true. Used for signaling between threads.

---

### Thread Safety: Best Practices

* Always protect shared data with mutexes or atomic operations.
* Minimize the amount of code in critical sections.
* Avoid deadlocks by locking/unlocking in consistent order.
* Never call non-thread-safe library functions from multiple threads (check man pages).

---

### Why It Matters (CNO/RE Context)

* Threading bugs are a major source of vulnerabilities (race conditions, TOCTOU bugs, data corruption).
* Malware and CNO tools use threads for efficiency and stealth (e.g., background keyloggers, packet sniffers).
* Reverse engineers often need to understand thread behavior to analyze multi-threaded malware or implants.

---

## 🧠 Key Concepts To Know

* How `pthread_create()` launches a thread and what parameters it takes.
* How/when to use mutexes to protect shared data.
* What is a race condition, and how to reproduce and fix it.
* What happens if a thread is not joined (detached threads, memory leaks).

---

## ❓ Essential Questions

* What makes threading hard to debug and test?
* How can you tell if a function or variable is safe to use in multiple threads?
* What’s the difference between concurrency (threads) and parallelism (multiple CPUs)?
* What is a deadlock, and how can you avoid it?

---

## 🧪 Lab: Multi-Threaded Counter (Race Condition and Mutex Fix)

### **Instructions:**

1. **Write a C program:**

   * Create a global counter variable.
   * Start 5 threads; each thread increments the counter 100,000 times.
   * Print the final value of the counter.
   * Run first **without** a mutex, then add a mutex to protect the counter.
   * Observe incorrect vs. correct results.

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <pthread.h>

   #define NTHREADS 5
   #define NINCR 100000
   int counter = 0;
   // pthread_mutex_t lock; // Uncomment for mutex version

   void *worker(void *arg) {
       for (int i = 0; i < NINCR; ++i) {
           // pthread_mutex_lock(&lock); // Uncomment for mutex
           counter++;
           // pthread_mutex_unlock(&lock); // Uncomment for mutex
       }
       return NULL;
   }

   int main() {
       pthread_t threads[NTHREADS];
       // pthread_mutex_init(&lock, NULL); // Uncomment for mutex
       for (int i = 0; i < NTHREADS; ++i)
           pthread_create(&threads[i], NULL, worker, NULL);
       for (int i = 0; i < NTHREADS; ++i)
           pthread_join(threads[i], NULL);
       printf("Final counter: %d\n", counter);
       // pthread_mutex_destroy(&lock); // Uncomment for mutex
       return 0;
   }
   ```

2. **Compile and run (no mutex):**

   ```sh
   gcc -pthread -o race_counter lab7.c
   ./race_counter
   ```

   * Repeat multiple times; the final counter will likely be **wrong**.

3. **Add the mutex (uncomment code), recompile, and rerun:**

   * The final counter will now always be correct (5 \* 100,000 = 500,000).

---

### **Expected Observables:**

* Without mutex: final counter is incorrect (race condition)
* With mutex: final counter is correct (race eliminated)
* Demonstrates the need for thread safety in concurrent code

---

## 📖 Further Reading

* [Beej’s Guide: Threads](https://beej.us/guide/bgipc/html/split/pthreads.html)
* [Linux man page: pthreads](https://man7.org/linux/man-pages/man7/pthreads.7.html)
* [Linux man page: pthread\_mutex\_lock](https://man7.org/linux/man-pages/man3/pthread_mutex_lock.3p.html)
* [Race Conditions Explained (GeeksForGeeks)](https://www.geeksforgeeks.org/race-condition-in-process-synchronization/)

---

*Document your code, terminal output, and a short write-up comparing the race vs. mutex runs.*
