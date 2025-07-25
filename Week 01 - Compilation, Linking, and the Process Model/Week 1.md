# ⚙️ Systems Programming in C — Week 1

## 📚 Lecture Notes: Compilation, Linking, and the Process Model

### Source to Binary: How C Code Becomes a Program

**1. Preprocessing:**

* Handles directives like `#include`, `#define`, and `#ifdef`.
* Output: expanded source code file.

**2. Compilation:**

* Translates preprocessed C source (`.c`) to assembly (`.s`), then to object code (`.o`).
* Checks syntax, produces errors/warnings.

**3. Assembling:**

* Converts assembly into machine code/object files (`.o`).
* Each `.c` → `.o` if you have multiple files.

**4. Linking:**

* Combines one or more object files (`.o`) into a single executable (`a.out` or named).
* Resolves external symbols (e.g., standard library functions).

  * **Static linking:** Copies library code into your executable.
  * **Dynamic linking:** Leaves references to external libraries (e.g., `libc.so`), loaded at runtime.

**Summary Diagram:**

```
hello.c ──[preprocess]──> hello.i ──[compile]──> hello.s ──[assemble]──> hello.o
                  └───────────────────────────────────────────┬───────┘
                                               [link]         ↓
                                                        hello (ELF binary)
```

---

### Static vs. Dynamic Linking

|                   | **Static Linking**                                                                     | **Dynamic Linking**                       |
| ----------------- | -------------------------------------------------------------------------------------- | ----------------------------------------- |
| **What**          | Library code is copied into the binary                                                 | Binary contains references to shared libs |
| **Pros**          | Fully self-contained, portable                                                         | Smaller binaries, libraries updatable     |
| **Cons**          | Larger binaries, can't update library bugs                                             | Dependency on shared libs at runtime      |
| **CNO/RE Tie-In** | Statically-linked malware is harder to patch; dynamic linkage can be hijacked/replaced |                                           |

---

### Anatomy of a Linux Process

A process in memory is divided into key segments:

* **.text**: Compiled program instructions (code)
* **.data**: Global/static variables, initialized at compile time
* **.bss**: Global/static variables, uninitialized (zero-filled at startup)
* **Heap**: Dynamic memory (`malloc`/`free`)
* **Stack**: Local variables, function call frames

**Diagram: Typical Process Memory Layout**

```
High Memory Addresses
+------------------------+  <- Stack (grows downward)
|      Stack             |
+------------------------+
|      Heap              |  <- Expands upward with malloc()
+------------------------+
|   .bss (uninitialized) |
+------------------------+
|   .data (initialized)  |
+------------------------+
|   .text (code)         |
+------------------------+
Low Memory Addresses
```

---

## 🧠 Key Concepts To Know

* **Object file:** A compiled but not yet linked file (`.o`), machine code plus metadata.
* **Executable (ELF):** The final, runnable binary on Linux (ELF = Executable and Linkable Format).
* **Symbol:** A named function or variable that may be referenced across files or libraries.
* **.text/.data/.bss:** Main sections in ELF and memory; each with specific contents and security implications.

---

## ❓ Essential Questions

* What is the difference between static and dynamic linking? When would you use each?
* What is an ELF binary, and what are its main segments?
* How do the stack, heap, data, and code (.text) regions differ at runtime?
* Why does knowing the process layout matter for security, exploits, or debugging?

---

## 🧪 Lab: Write, Compile, and Analyze a Simple C Binary

### **Instructions:**

1. **Write a simple C program:**

   ```c
   #include <stdio.h>
   int global_init = 42;
   int global_uninit;
   int main() {
       int local = 7;
       printf("Hello, world! %d\n", local + global_init);
       return 0;
   }
   ```

   Save as `lab1.c`.

2. **Compile the program:**

   ```sh
   gcc -o lab1 lab1.c
   ```

3. **Use ****\`\`**** to analyze:**

   ```sh
   objdump -h lab1
   ```

   * Look for `.text`, `.data`, `.bss` in the output.

4. **Use ****\`\`**** to inspect ELF segments:**

   ```sh
   readelf -S lab1
   ```

   * Identify the same sections and their sizes.

5. **Bonus:**

   * Try making `global_uninit` initialized (set to a value) and recompile. See what changes in `.bss` and `.data`.

---

### **Expected Observables:**

* Identify and explain the `.text`, `.data`, and `.bss` sections in your program using `objdump` and `readelf`.
* Show which variables end up in which segment:

  * `global_init` → `.data`
  * `global_uninit` → `.bss`
  * `main` function and code → `.text`
* Understand how process memory layout influences exploitation and reverse engineering.

---

## 📖 Further Reading

* [Beej’s Guide to C Programming: Compilation & Linking](https://beej.us/guide/bgc/html/split/compiling.html)
* [ELF: The Executable and Linkable Format](https://linux.die.net/man/5/elf)
* [objdump man page](https://man7.org/linux/man-pages/man1/objdump.1.html)
* [readelf man page](https://man7.org/linux/man-pages/man1/readelf.1.html)
* [x86 Process Memory Layout (GeeksForGeeks)](https://www.geeksforgeeks.org/memory-layout-of-c-program/)

---

*Document your steps: screenshots, terminal output, and brief notes on what you learned about your program's binary structure.*
