# ⚙️ Systems Programming in C — Week 10

## 📚 Lecture Notes: Advanced Process and Memory Manipulation

### `execve()` and Custom Environments

* **execve(path, argv, envp):** Replaces the current process image with a new program. Used to launch programs with custom arguments and environment variables.
* **Environment variables** can be passed as the third argument. Allows precise control for child processes, malware, or RE test harnesses.
* Example: `execve("/bin/ls", argv, envp);`

---

### Using `ptrace()` for Debugging/Tracing (Intro Only)

* **ptrace():** System call allowing one process (the tracer) to observe and control another (the tracee). Used by debuggers like `gdb`.
* Capabilities: inspect/change memory/registers, single-step instructions, set breakpoints, catch syscalls.
* **Security/RE note:** ptrace is heavily used for debugging, exploit dev, anti-debug, and malware analysis.

---

### Code Injection: `mmap` + Shellcode (Demo, Not Weaponized)

* **mmap():** Allocate a new memory region, potentially with both write and execute permissions. Used to load/run custom code (shellcode, plugins, or code injection attacks).
* **Shellcode:** Raw machine code injected into a process (demonstrate benign usage, e.g., allocating and modifying memory).
* **Security warning:** This is for educational/demo purposes only.

---

### Why It Matters (CNO/RE Context)

* Tools/exploits often use `execve` for privilege escalation, pivoting, or sandbox escape.
* ptrace powers most RE tooling (debuggers, dynamic analysis, malware unpackers).
* Memory manipulation (mmap, shellcode) is foundational for exploit dev, reverse shells, and AV evasion.

---

## 🧠 Key Concepts To Know

* How `execve` differs from `system()` and why it matters for security.
* What ptrace can and cannot do (tracing, modifying memory, catching syscalls).
* Why memory permissions (RWX) are a critical attack/defense surface.
* Common anti-debug and anti-tracing techniques in malware.

---

## ❓ Essential Questions

* What are the risks of allowing write+execute memory in a process?
* How does `ptrace` let a debugger change program behavior at runtime?
* Why is using `execve` safer and more flexible than `system()`?
* How do advanced exploits use mmap/shellcode in real world scenarios?

---

## 🧪 Lab: Trace and Modify a Running Process with ptrace

### **Instructions:**

1. **Write a C program (tracee):**

   * Declares an integer variable and prints its value in a loop.
   * Sleeps 1s per iteration, so you have time to attach and inspect.

   Example:

   ```c
   // victim.c
   #include <stdio.h>
   #include <unistd.h>
   int secret = 42;
   int main() {
       for (int i = 0; i < 10; ++i) {
           printf("Iteration %d, secret = %d\n", i, secret);
           sleep(1);
       }
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o victim victim.c
   ./victim
   # Leave running, note PID (or use pgrep victim)
   ```

3. **Write a ptrace tool (tracer):**

   * Attach to the running process.
   * Locate and modify the `secret` variable (see man 2 ptrace for PEEK/POKE usage).
   * Print a message when modification succeeds.

   Example (partial outline):

   ```c
   // tracer.c
   #include <stdio.h>
   #include <sys/ptrace.h>
   #include <sys/wait.h>
   #include <sys/types.h>
   #include <unistd.h>
   #include <stdlib.h>
   #include <errno.h>

   int main(int argc, char *argv[]) {
       if (argc != 3) {
           printf("Usage: %s <PID> <new_value>\n", argv[0]);
           return 1;
       }
       pid_t pid = atoi(argv[1]);
       long newval = atol(argv[2]);
       if (ptrace(PTRACE_ATTACH, pid, NULL, NULL) == -1) {
           perror("ptrace attach");
           return 1;
       }
       waitpid(pid, NULL, 0);
       // Find address of secret (may require nm, objdump, or hardcode address)
       // Use ptrace(PTRACE_PEEKDATA/POKEDATA) to read/write variable
       // Example: ptrace(PTRACE_POKEDATA, pid, addr_of_secret, newval);
       ptrace(PTRACE_DETACH, pid, NULL, NULL);
       printf("Modified variable in process %d\n", pid);
       return 0;
   }
   ```

   *(You'll need to find the address of `secret` using nm/objdump.)*

4. **Demonstrate:**

   * Run victim, attach tracer, change variable, observe value change in victim’s output.

---

### **Expected Observables:**

* The target variable (`secret`) changes value mid-execution due to tracer’s intervention.
* You can watch memory being modified in a live process using ptrace.
* (Bonus) Try using `mmap` to allocate and write to executable memory in your own process (for further study).

---

## 📖 Further Reading

* [Beej’s Guide: Advanced Process Tricks](https://beej.us/guide/bgipc/html/split/ptrace.html)
* [Linux man page: ptrace()](https://man7.org/linux/man-pages/man2/ptrace.2.html)
* [Linux man page: execve()](https://man7.org/linux/man-pages/man2/execve.2.html)
* [Understanding Code Injection (LWN.net)](https://lwn.net/Articles/244053/)
* [Memory Permissions and Shellcode (GeeksForGeeks)](https://www.geeksforgeeks.org/shellcode-in-c/)

---

*Document your code, terminal output, and a summary of how you used ptrace to inspect/modify live process memory.*
