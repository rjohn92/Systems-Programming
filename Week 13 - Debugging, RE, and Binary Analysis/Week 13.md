# ⚙️ Systems Programming in C — Week 13

## 📚 Lecture Notes: Debugging, RE, and Binary Analysis

### GDB: Breakpoints, Watchpoints, Stack/Frame Inspection

* **GDB:** The GNU Debugger, used for interactive debugging of C programs (step through code, inspect variables, set breakpoints, etc.).
* **Breakpoint:** Pauses execution at a specified line or function.
* **Watchpoint:** Pauses execution when a variable changes.
* **Stack/frame inspection:** View call stack, function arguments, and local variables at any point in execution.
* **Usage:**

  ```sh
  gdb ./myprog
  (gdb) break main
  (gdb) run
  (gdb) next / step / continue
  (gdb) print var
  (gdb) backtrace
  (gdb) watch var
  ```

---

### objdump, readelf, and Symbol Tables

* **objdump -d:** Disassembles binary code to assembly (shows instructions at each address).
* **objdump -t:** Displays symbol table (functions, variables, their addresses and types).
* **readelf -h/-S/-s:** Inspects ELF headers, sections, and symbols.
* These tools help tie C source to compiled binary, and are essential for RE, exploit dev, and debugging.

---

### RE Tie-in: Finding main(), Entry Point, and Stack Variables

* **main():** Can be found in symbol table and disassembly (may have an underscore or address prefix).
* **Entry point:** ELF binaries have an entry address (`readelf -h` shows Entry point address).
* **Stack variables:** Appear as references to offsets from `rbp`/`esp` (x86/x64) in assembly.
* Use GDB to step into functions, observe stack frames, and see how arguments/locals are managed.

---

### Why It Matters (CNO/RE Context)

* Being able to debug, disassemble, and analyze binaries is foundational for reverse engineering, exploit development, and incident response.
* Finding function boundaries, stack layouts, and data flows enables bug hunting, patching, and code auditing.

---

## 🧠 Key Concepts To Know

* How to set/clear breakpoints and watchpoints in GDB
* How to step through code line by line, or instruction by instruction
* How to find function addresses in the symbol table
* How stack frames are managed in assembly and how to map locals/args

---

## ❓ Essential Questions

* What is the difference between stepping with `next` vs `step` in GDB?
* How can you tell where a function starts and ends in assembly?
* What does the stack look like before/after a function call?
* How can missing symbols (stripped binaries) make RE harder?

---

## 🧪 Lab: Debug and Analyze a C Binary

### **Instructions:**

1. **Write a simple C program:**

   * Defines at least one function that calls another (with local variables).
   * Prints some values.
   * Example:

   ```c
   // lab13.c
   #include <stdio.h>
   int square(int x) {
       return x * x;
   }
   int main() {
       int a = 5;
       int b = square(a);
       printf("a=%d b=%d\n", a, b);
       return 0;
   }
   ```

2. **Compile with debug symbols:**

   ```sh
   gcc -g -o lab13 lab13.c
   ```

3. **Debug with GDB:**

   * Set breakpoints at `main` and `square`.
   * Step through calls, print variables, inspect stack frames.
   * Use `backtrace` to show the call stack.

4. **Disassemble with objdump:**

   ```sh
   objdump -d lab13 | less
   objdump -t lab13 | grep square
   readelf -h lab13
   ```

   * Find addresses of functions, entry point, symbol names.
   * Observe how the `square` function appears in assembly (look for `call` instruction).

---

### **Expected Observables:**

* Ability to set/trigger breakpoints and step through functions
* See local variables on the stack, stack frame layout in GDB/assembly
* Locate `main`, `square`, and entry point in symbol table and disassembly
* Document how a function call (`call square`) appears in assembly

---

## 📖 Further Reading

* [GDB Manual (GNU)](https://www.gnu.org/software/gdb/documentation/)
* [Beej’s Guide: Debugging](https://beej.us/guide/bgc/html/split/debug.html)
* [Linux man page: objdump](https://man7.org/linux/man-pages/man1/objdump.1.html)
* [Linux man page: readelf](https://man7.org/linux/man-pages/man1/readelf.1.html)
* [How to Read x86 Assembly for C Programmers (b47ch)](https://b47ch.github.io/c-to-assembly.html)

---

*Document your code, GDB and objdump output, and a write-up showing where function calls and local variables appear in assembly.*
