# ⚙️ Systems Programming in C — Week 12

## 📚 Lecture Notes: Defensive and Offensive Programming

### Safe Coding: Bounds Checking, Error Handling, `assert`

* **Bounds checking:** Always verify array/string indices to prevent out-of-bounds access.
* **Error handling:** Check all return values for system calls and library functions (e.g., `malloc`, `read`, `write`). Don’t assume success!
* **assert(condition):** Macro to test invariants at runtime; aborts if the condition is false. Use for debugging but remove from production code.

---

### Classic Bugs

* **Buffer overflow:** Writing beyond the end of an array or buffer, corrupting adjacent memory. Can lead to code execution or crashes.
* **Use-after-free:** Accessing memory after it has been freed; can cause crashes or enable exploitation.
* **TOCTOU (Time-Of-Check to Time-Of-Use):** Race condition where a file/resource changes between validation and usage (e.g., check permissions, then open file; attacker swaps the file in the gap).

---

### Detecting and Fixing Vulnerabilities with Valgrind

* **Valgrind:** Dynamic analysis tool for finding memory bugs (leaks, use-after-free, out-of-bounds).
* Usage: `valgrind ./myprog`
* Reports invalid reads/writes, leaks, and more. Essential for C/C++ security work.

---

### Why It Matters (CNO/RE Context)

* Buffer overflows, UAF, and TOCTOU are among the most exploited bugs in history (shellcode, privilege escalation, malware persistence).
* Secure coding and memory analysis skills are *baseline* for any serious CNO, RE, or exploit developer.

---

## 🧠 Key Concepts To Know

* Difference between stack and heap overflows
* How/why to use `assert`, and its limits
* Common places for UAF and TOCTOU bugs in systems code
* How to read and act on Valgrind output

---

## ❓ Essential Questions

* What is the difference between buffer overflow and use-after-free?
* How does TOCTOU enable privilege escalation or file hijacking?
* What coding habits prevent most classic C bugs?
* Why should you check *all* return values in C?

---

## 🧪 Lab: Write, Detect, and Fix a Vulnerable Program

### **Instructions:**

1. **Write a vulnerable C program:**

   * Example: A function that copies user input to a fixed-size buffer without bounds checking.
   * Intentionally free a pointer, then access it (UAF).

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>

   void vuln() {
       char buf[16];
       printf("Enter something: ");
       gets(buf); // Vulnerable: no bounds checking
       printf("You entered: %s\n", buf);
   }

   int main() {
       char *p = malloc(10);
       free(p);
       printf("Freed pointer: %p, value: %d\n", p, *p); // Use-after-free
       vuln();
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -g -o vulnprog lab12.c
   ./vulnprog
   ```

3. **Detect with Valgrind:**

   ```sh
   valgrind ./vulnprog
   ```

   * Observe and record output (invalid read/write, use-after-free, possible overflow).

4. **Patch the program:**

   * Replace `gets()` with `fgets(buf, sizeof(buf), stdin);`
   * Do not access freed memory.
   * Re-run with Valgrind; ensure all errors are gone.

---

### **Expected Observables:**

* Initial run: Valgrind reports memory errors (UAF, overflow, invalid reads/writes).
* After patch: Valgrind shows no memory errors; input is bounds-checked and safe.

---

## 📖 Further Reading

* [Beej’s Guide: Memory Safety](https://beej.us/guide/bgc/html/split/memdebug.html)
* [Valgrind Documentation](https://valgrind.org/docs/manual/manual.html)
* [Stack Overflow: Classic C Security Bugs](https://stackoverflow.com/questions/6441215/common-c-programming-mistakes)
* [TOCTOU Explained (OWASP)](https://owasp.org/www-community/attacks/Time_of_check_to_time_of_use_-_TOCTOU)
* [Safe Coding Practices for C (CERT Guide)](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard)

---

*Document your code, Valgrind output (before/after), and write a summary of what was fixed and why.*
