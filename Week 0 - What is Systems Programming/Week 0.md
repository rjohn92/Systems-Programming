# ⚙️ Systems Programming in C — Week 0

## 📚 Lecture Notes: What Is Systems Programming?

**Systems programming** is the art and science of writing software that interacts closely with the underlying operating system (OS) and hardware.  
Unlike application programming (where you focus on *what* your program does for a user), systems programming is about *how* things work “under the hood.”  
You’ll use the OS as a set of low-level tools: handling files, memory, processes, network, and interacting with the kernel via system calls.

---

### Why Learn Systems Programming?

- **CNO/RE Relevance:**  
  Nearly all exploits, implants, and reverse engineering tasks require understanding how binaries interact with the OS at the syscall level.
- **Tooling:**  
  Tools like debuggers, loaders, and malware must manipulate processes, memory, and files — not just use high-level APIs.
- **Security:**  
  Most vulnerabilities (buffer overflows, race conditions, privilege escalation) happen at the system interface, not in business logic.

---

### Systems Programming vs. Application Programming

|                | **Systems Programming**                           | **Application Programming**          |
|----------------|--------------------------------------------------|--------------------------------------|
| **Main Goal**  | Interact with/manage OS/hardware directly        | Provide end-user features/function   |
| **Examples**   | Shells, daemons, device drivers, servers         | Word processors, games, web apps     |
| **Language**   | C, C++, Rust, Assembly                           | C, C++, Python, Java, JS, etc.       |
| **Skills**     | Memory mgmt, syscalls, signals, concurrency      | UI/UX, business logic, frameworks    |
| **Failure**    | Often = crash, data loss, security bug           | Usually = user error, recoverable    |

---

### User Space vs. Kernel Space

- **User Space:**  
  Where normal programs run. Cannot directly access hardware or critical OS resources.  
  *Must* ask the kernel (via system calls) to do things like read files, allocate memory, open network connections.
- **Kernel Space:**  
  The privileged core of the OS. Can access all hardware, manage all memory/processes/devices.  
  User programs “cross the barrier” into kernel space via *syscalls* (e.g., `open`, `read`, `write`, `execve`).

*Why does this matter?*  
- You can only attack, defend, or analyze at the system level if you know where these boundaries are.

---

### What Happens When You Run a C Program? *(A High-Level Flow)*

1. **Compilation:**  
   Source code → object file(s) → linked into an executable binary (ELF on Linux)
2. **Execution:**  
   - You type `./myprog` in the shell.
   - The shell calls the `execve()` syscall to launch your program.
   - The kernel loads your program’s code and data into memory, sets up the stack/heap, and starts execution at `main()`.
   - Any time your code calls things like `printf`, `fopen`, or `malloc`, those eventually use system calls to interact with the OS.
3. **Running:**  
   - Your program lives in *user space*, making syscalls into *kernel space* when it needs privileged operations.
   - When your program exits (via `exit()`), it tells the kernel to clean up its memory and process entry.

---

## 🧠 Key Concepts To Know

- **System Call (syscall):**  
  A controlled “doorway” for user programs to ask the kernel to perform privileged actions (e.g., reading a file, creating a process).
- **User space vs. kernel space:**  
  Why these are separated, and what happens at the boundary.
- **Process:**  
  A running program — memory, code, data, and state, as tracked by the OS.
- **strace:**  
  A tool to “watch” a program as it makes syscalls; invaluable for debugging, RE, and exploit dev.

---

## ❓ Essential Questions

- What’s the difference between user space and kernel space? Why does it matter?
- What is a system call, and how does your C program use them (even if you never call them directly)?
- Why are shells, debuggers, and CNO tooling always written in systems languages like C?
- What does `strace` show you that normal debugging tools do not?

---

## 🧪 Lab: Trace a Program With strace

**Goal:**  
See, in real time, how even a simple command-line tool (`ls`) interacts with the OS via syscalls.

**Instructions:**

1. **Run a simple command with `strace`:**
   ```sh
   strace ls
```
## Observe the output:

Each line shows a system call: its name, arguments, and return value.

Look for common syscalls like openat, read, write, close, execve.

## Diagram the flow:

Sketch or annotate the sequence: e.g., execve("ls") → openat() on directories → multiple read() and write() calls.

Note when the program switches from user code to kernel and back.

## Expected Observables:

A list of syscalls called by ls

Understanding which syscalls interact with the filesystem, which with the terminal

Basic “mental map” of how a user program talks to the OS

## Bonus:
Try strace -c ls to get a syscall summary/count.

## 📖 Further Reading
Beej's Guide to C Programming (Systems/OS Basics)

Linux man page: strace

What is a system call? (TLPI)

Why User/Kernel Mode? (LWN.net)

Document your process: What did you expect, what surprised you, and what syscalls seemed most “critical” for the program?
