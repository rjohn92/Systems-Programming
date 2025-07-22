# ⚙️ Systems Programming in C — Week 2

## 📚 Lecture Notes: Processes and Program Execution

### Fork/Exec/Wait: Creating and Managing Processes

* **Process:** An instance of a running program (has its own memory, PID, open files, state).
* **fork():** System call that duplicates the calling process. The parent receives the child’s PID; the child receives 0. Both resume execution after the fork.
* **exec():** Replaces the current process image with a new program (e.g., `execve`, `execlp`, etc). Used after fork in the child to run a different program.
* **wait():** Parent blocks until one of its children terminates. Returns child’s PID and exit status.

**Typical flow:**

1. Parent calls `fork()`.
2. Child continues; may call `exec()` to run another program.
3. Parent calls `wait()` to reap child.

---

### Parent/Child Relationships & Process Hierarchy

* Every process (except `init`/`systemd`) has a parent.
* Children inherit environment, open FDs (unless set otherwise), and start as near-copies.
* The Linux process tree can be viewed with `pstree`.
* Parent can track children (PIDs), synchronize via `wait()`/`waitpid()`.

---

### Zombie and Orphan Processes

* **Zombie:** A process that has finished executing but whose parent hasn't read its exit status (i.e., not reaped with `wait()`). Remains in the process table (shows as "<defunct>").
* **Orphan:** A child whose parent has exited. Orphans are adopted by `init`/`systemd` (PID 1), which reaps them.
* Too many zombies = process table pollution; good hygiene is to always reap your children!

---

## 🧠 Key Concepts To Know

* Difference between process, thread, and program
* How `fork()`, `exec()`, and `wait()` interact
* What happens to a process when its parent dies (adoption by init)
* Real-world bugs: un-reaped zombies, runaway orphans
* CNO/RE tie-in: Malware/process hiding, privilege escalation via process tree confusion

---

## ❓ Essential Questions

* What does `fork()` actually duplicate, and what is unique to the child?
* What is the difference between running a command with `system()` vs `fork()` + `exec()`?
* How do zombies appear, and how do you clean them up?
* Why might a CNO/RE analyst care about process hierarchies?

---

## 🧪 Lab: Forking, Exec-ing, and Reaping Processes

### **Instructions:**

1. **Write a C program:**

   * Parent process forks one or more children (suggest: 2).
   * Each child executes the `ls` command (use `execlp` or `execvp`).
   * Parent waits for each child to finish (`wait()` or `waitpid()`).
   * Parent prints child PIDs and exit statuses.

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <sys/wait.h>

   int main() {
       for (int i = 0; i < 2; ++i) {
           pid_t pid = fork();
           if (pid == 0) {
               // Child: run ls
               execlp("ls", "ls", NULL);
               perror("exec failed");
               exit(1);
           } else if (pid > 0) {
               // Parent: continue loop
               printf("Forked child PID: %d\n", pid);
           } else {
               perror("fork failed");
           }
       }
       // Parent: reap children
       for (int i = 0; i < 2; ++i) {
           int status;
           pid_t child = wait(&status);
           printf("Reaped child %d, exit status: %d\n", child, WEXITSTATUS(status));
       }
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o forkexec lab2.c
   ./forkexec
   ```

3. **While running, use `ps` and `pstree` in another terminal:**

   * `ps -ef | grep forkexec`
   * `pstree -p <your_shell_pid>`
   * Observe parent and child processes.

4. **Try commenting out the wait loop.**

   * What happens to the children after they finish? Use `ps`/`pstree` to observe zombies.

---

### **Expected Observables:**

* Multiple child processes (running `ls`) visible in the process tree.
* With proper `wait()`, children are reaped and do not become zombies.
* If `wait()` is omitted, children become zombies ("<defunct>").
* Orphans, if any, are reparented to `init`/`systemd`.
* Ability to see process relationships with `ps`/`pstree`.

---

## 📖 Further Reading

* [Beej’s Guide: Processes & Fork](https://beej.us/guide/bgipc/html/split/fork.html)
* [Linux man page: fork()](https://man7.org/linux/man-pages/man2/fork.2.html)
* [Linux man page: execve()](https://man7.org/linux/man-pages/man2/execve.2.html)
* [Zombie Processes Explained (Linux Handbook)](https://linuxhandbook.com/zombie-process/)
* [Understanding pstree](https://linux.die.net/man/1/pstree)

---

*Document your code, terminal output, and a short summary of what you observed with and without proper child reaping.*
