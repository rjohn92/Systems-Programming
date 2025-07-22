# ⚙️ Systems Programming in C — Week 4

## 📚 Lecture Notes: Memory and the Process Address Space

### Stack vs. Heap vs. Data Segment

* **Stack:** Region for function calls, local variables. Grows/shrinks as functions are called/returned. Fast, but limited in size; memory is managed automatically.
* **Heap:** Region for dynamically allocated memory (via `malloc`, `calloc`, etc.). Grows as needed, but must be manually managed (`free`). Larger, but slower and more error-prone (leaks, overflows).
* **Data Segment:**

  * **.data**: Global/static variables initialized at compile time.
  * **.bss**: Global/static variables uninitialized at compile time.
  * **.text**: The actual program code (read-only).

---

### Dynamic Memory Management

* **malloc(size):** Allocates `size` bytes on the heap, returns pointer. Contents uninitialized.
* **free(ptr):** Frees previously allocated memory.
* **brk/sbrk:** Legacy syscalls to change the "end" of the process data segment (heap). Used by malloc under the hood on some systems.
* **mmap:** More modern syscall to allocate (potentially large or custom-aligned) regions of memory anywhere in the process space. Used for big allocations, memory-mapped files, and shellcode.

---

### Reading `/proc/self/maps`

* On Linux, `/proc/self/maps` shows a live map of your process memory regions.
* Each line lists an address range, permissions (rwxp), offset, device, inode, and region name (if any).
* Typical regions: stack, heap, code (.text), shared libraries, mapped files, etc.

Example excerpt:

```
00400000-0040b000 r-xp 00000000 fd:01 393235     /home/user/a.out
0060a000-0060b000 rw-p 0000a000 fd:01 393235     /home/user/a.out
00e42000-00e63000 rw-p 00000000 00:00 0          [heap]
7fff549f5000-7fff549f6000 rw-p 00000000 00:00 0  [stack]
```

---

## 🧠 Key Concepts To Know

* Difference between stack and heap allocations (lifetime, usage, risks)
* How global variables are mapped to .data or .bss
* When and why to use `malloc`/`free` versus stack arrays
* How to interpret `/proc/self/maps` and recognize stack, heap, and code regions
* Security impact: buffer overflows, use-after-free, heap spraying (CNO/RE relevance)

---

## ❓ Essential Questions

* What happens if you forget to free heap memory? If you free it twice?
* Why might you use `mmap` instead of `malloc`?
* What information does `/proc/self/maps` reveal that is useful for exploit dev or reverse engineering?
* How does stack overflow differ from heap overflow?

---

## 🧪 Lab: Memory Allocation and Memory Map Inspection

### **Instructions:**

1. **Write a C program:**

   * Declare a global variable, a static variable in `main()`, and a local (stack) variable.
   * Use `malloc()` to allocate a buffer on the heap.
   * Print the addresses of each variable and the heap pointer.
   * Use `free()` on the heap allocation.
   * Use `getchar()` to pause the program (so you can inspect its memory map externally).

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>

   int global_var = 1;

   int main() {
       static int static_var = 2;
       int stack_var = 3;
       int *heap_var = malloc(100);
       printf("global_var: %p\n", (void*)&global_var);
       printf("static_var: %p\n", (void*)&static_var);
       printf("stack_var: %p\n", (void*)&stack_var);
       printf("heap_var: %p\n", (void*)heap_var);
       getchar(); // pause
       free(heap_var);
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o memlab lab4.c
   ./memlab
   ```

3. **In another terminal, inspect memory map:**

   ```sh
   cat /proc/$(pidof memlab)/maps
   ```

   * Find regions: stack, heap, code (.text), data (.data/.bss), shared libraries.

4. **Annotate:**

   * Match printed addresses to their respective regions in `/proc/self/maps`.
   * Make notes/sketch: which variables are in stack, heap, data segment, or code region?

---

### **Expected Observables:**

* Output prints variable addresses — should be spread across different regions
* `cat /proc/<pid>/maps` clearly shows \[heap], \[stack], code, and data regions
* You can confidently identify at least stack, heap, code (.text), and data segments in the live memory map

---

## 📖 Further Reading

* [Beej’s Guide: Memory Layout](https://beej.us/guide/bgipc/html/split/memory.html)
* [Linux man page: malloc()](https://man7.org/linux/man-pages/man3/malloc.3.html)
* [Linux man page: mmap()](https://man7.org/linux/man-pages/man2/mmap.2.html)
* [Understanding /proc/self/maps (Linux)](https://man7.org/linux/man-pages/man5/proc.5.html)
* [Stack vs. Heap (GeeksForGeeks)](https://www.geeksforgeeks.org/stack-vs-heap-memory-allocation/)

---

*Document your code, terminal output, and a summary of your variable addresses and process memory map.*
