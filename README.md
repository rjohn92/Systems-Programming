# ⚙️ Systems Programming in C (CNO/RE Track) – Master Syllabus

## Overview

**Goal:**  
Learn to write, debug, and analyze C code that interfaces directly with the operating system.  
Gain practical skills for building, auditing, and reverse engineering tools, exploits, and system-level applications as required in CNO and RE roles.

---

## 🎯 Audience

This course is designed for:
- Aspiring CNO analysts, vulnerability researchers, and offensive cyber professionals (entry to intermediate)
- Security engineers and DevOps/SREs transitioning into lower-level systems or RE work
- CS students or professionals who have **basic programming** experience but need hands-on systems/OS knowledge
- Anyone preparing for roles that involve exploit development, malware analysis, or advanced system debugging

---

## 🏁 Prerequisites

To succeed in this course, you should **already be able to**:
- Write, debug, and compile basic C programs (variables, loops, functions, structs, arrays, pointers)
- Understand fundamental concepts of memory management (`malloc`, `free`)
- Use the Linux command line, `gcc`, and navigate/manipulate files/directories
- Use a basic debugger (gdb, lldb, or similar) to step through C code and view variables
- Read and edit text/code using a terminal editor (vim, nano, or VSCode with terminal)

**Recommended before starting:**  
- Complete a C Programming Fundamentals course or self-study syllabus (see: [Comprehensive C Syllabus](../C-Syllabus/README.md) or equivalent)

---

## 🗝️ What Skill Level Is Expected?

- **Beginner-to-intermediate:**  
  This is *not* an "introduction to programming" course — you should be comfortable with C syntax, pointers, and compiling simple programs.
- **You do NOT need to:**  
  - Have prior systems programming or reverse engineering experience
  - Know advanced OS internals (you will learn these here)
  - Have exploited real-world vulnerabilities before
- **You WILL be challenged to:**  
  - Write non-trivial, multi-file C programs
  - Debug and analyze memory/process issues
  - Use and interpret syscall traces, assembly listings, and OS-level tools

---

## 🏗️ Learning Outcomes

By the end of this course, you’ll be able to:
- Write C programs that use syscalls, manage processes, threads, files, and memory directly
- Analyze how your programs interact with the OS (and vice versa)
- Debug, trace, and analyze real binaries using modern tools (GDB, objdump, strace, Valgrind)
- Understand and find basic vulnerabilities, and build a foundation for CNO/RE tool development

---

## 🗓️ Weekly Breakdown
...
# ⚙️ Systems Programming in C (CNO/RE Track) – Master Syllabus

## Overview

**Goal:**  
Learn to write, debug, and analyze C code that interfaces directly with the operating system.  
Gain practical skills for building, auditing, and reverse engineering tools, exploits, and system-level applications as required in CNO and RE roles.

---

## 🗓️ Weekly Breakdown

---

### **Week 0: What Is Systems Programming?**
- Systems programming vs. app programming
- User space vs. kernel space
- What happens when you run a C program?
- *Lab:* Use `strace` to trace `ls`; draw a syscall flow diagram.

---

### **Week 1: Compilation, Linking, and the Process Model**
- Source to binary: preprocessing, compilation, linking
- Static vs. dynamic linking
- Anatomy of a Linux process: stack, heap, data, code
- *Lab:* Write, compile, and analyze a simple C binary with `objdump` and `readelf`.
- *Observable:* Identify `.text`, `.data`, and `.bss` sections in your binary.

---

### **Week 2: Processes and Program Execution**
- Fork/exec/wait: creating and managing processes
- Parent/child relationships, process hierarchy
- Zombie and orphan processes
- *Lab:* Write a C program that forks children, executes `ls`, and reaps orphans.
- *Observable:* Use `ps`/`pstree` to view your process tree.

---

### **Week 3: File Descriptors and System I/O**
- What is a file descriptor?
- System calls: `open`, `read`, `write`, `close`
- Duplicating FDs: `dup`, `dup2`, redirecting stdin/out
- *Lab:* Write a simple `tee` clone (copy stdin to a file and stdout).
- *Observable:* Redirect your program’s output with `<` and `>` in the shell.

---

### **Week 4: Memory and the Process Address Space**
- Stack vs. heap vs. data segment
- Dynamic memory: `malloc`, `free`, `brk`, `sbrk`, `mmap`
- Reading `/proc/self/maps`
- *Lab:* Write a C program that allocates, uses, and frees memory, then inspects its address space.
- *Observable:* Annotate your own memory map and find stack/heap/code regions.

---

### **Week 5: Signals and Signal Handling**
- What are signals? Signal numbers (`kill -l`)
- Catching and ignoring signals: `signal()`, `sigaction()`
- Common signals: `SIGINT`, `SIGSEGV`, `SIGCHLD`
- *Lab:* Write a program that handles Ctrl+C and segmentation faults.
- *Observable:* Demonstrate handling `SIGINT` (print message, exit cleanly).

---

### **Week 6: Inter-Process Communication (Pipes, FIFOs)**
- Anonymous pipes (`pipe()`)
- Named pipes (FIFOs, `mkfifo`)
- Chaining processes with pipes (e.g., implementing a simple shell pipeline)
- *Lab:* Parent writes to a pipe, child reads from it; build a one-way IPC.
- *Observable:* Pass messages between parent and child using a pipe.

---

### **Week 7: Threads and Concurrency**
- `pthread_create`, `pthread_join`
- Race conditions, mutexes, condition variables
- Data sharing and thread safety
- *Lab:* Write a multi-threaded counter with and without mutexes.
- *Observable:* Show (un)safe output with/without synchronization.

---

### **Week 8: Multiplexed I/O (select, poll, epoll)**
- Why multiplex? The C10K problem
- `select()`, basics of `poll()` and `epoll()`
- Non-blocking I/O and readiness
- *Lab:* Write a simple server that uses `select()` to handle multiple file descriptors.
- *Observable:* Show multiple clients handled in one process.

---

### **Week 9: Intro to Sockets and Networking**
- Socket concepts (AF_INET, SOCK_STREAM, SOCK_DGRAM)
- `socket()`, `bind()`, `listen()`, `accept()`, `connect()`
- Simple TCP echo server/client
- *Lab:* Build a basic TCP echo server and client in C.
- *Observable:* Exchange messages between two terminals.

---

### **Week 10: Advanced Process and Memory Manipulation**
- `execve()` and custom environments
- Using `ptrace()` for debugging/tracing (intro only)
- Code injection: mmap + shellcode (demo, not weaponized)
- *Lab:* Use `ptrace` to trace/modify a running process.
- *Observable:* Demonstrate setting/modifying a variable at runtime.

---

### **Week 11: Filesystems and Directories**
- `stat`, `lstat`, `fstat`, understanding inode
- `mkdir`, `opendir`, `readdir`, directory traversal
- Permissions, ownership, `chmod`, `chown`
- *Lab:* Write a recursive directory walker.
- *Observable:* Print all files in a directory tree with metadata.

---

### **Week 12: Defensive and Offensive Programming**
- Safe coding: bounds checking, error handling, assert
- Classic bugs: buffer overflow, use-after-free, TOCTOU
- Detecting and fixing vulnerabilities with Valgrind
- *Lab:* Write a vulnerable program, then patch it.
- *Observable:* Show valgrind output before/after fix.

---

### **Week 13: Debugging, RE, and Binary Analysis**
- GDB: breakpoints, watchpoints, stack/frame inspection
- `objdump`, `readelf`, and symbol tables
- RE tie-in: find main(), entry point, and stack variables
- *Lab:* Debug and step through a compiled C binary, then examine with `objdump`.
- *Observable:* Document how a function call appears in assembly.

---

### **Week 14: Capstone Project and Integration**
- Build a CNO-style tool:
  - Options: mini shell, syscall tracer, mini RAT, vulnerable server, etc.
- Document all syscalls used, and perform basic RE walkthrough (GDB/objdump)
- *Lab/Project:* Complete and document a substantial systems program.
- *Observable:* README + annotated source + screenshots/logs of usage and analysis.

---

## 📚 Supplementary Topics / Appendices

- Using Makefiles and multi-file projects
- Advanced: `clone()`, namespaces, chroot, seccomp (Linux hardening)
- CNO/RE extra: ELF header parsing, intro to Ghidra, function signatures

---

## 📝 How To Use This Syllabus

- Treat each week as a standalone module — review, lab, observable outcome.
- Focus on code **and** analysis: always ask “what did the OS do when my code ran?”
- Annotate your labs with writeups, screenshots, and lessons learned.
- Tie in to CNO/RE: after each module, note “how could an attacker, defender, or analyst use/misuse this?”

---

*If you want expanded week-by-week lectures, lab instructions, or deep dives, just ask for a specific week!*
