# ⚙️ Systems Programming in C — Week 3

## 📚 Lecture Notes: File Descriptors and System I/O

### What is a File Descriptor?

* An **integer handle** that refers to an open file, socket, or device (in Linux, everything is a file!).
* Managed by the kernel; every process has its own file descriptor table.
* Standard FDs: `0` = stdin, `1` = stdout, `2` = stderr.

---

### System Calls: `open`, `read`, `write`, `close`

* **open(path, flags, mode)**: Opens a file and returns a new file descriptor.
* **read(fd, buf, count)**: Reads up to `count` bytes from `fd` into `buf`.
* **write(fd, buf, count)**: Writes `count` bytes from `buf` to `fd`.
* **close(fd)**: Releases the file descriptor.
* All errors are reported as `-1`, with details in `errno`.

---

### Duplicating FDs: `dup`, `dup2`, Redirecting stdin/out

* **dup(fd)**: Duplicates an existing FD; returns a new FD pointing to the same file.
* **dup2(oldfd, newfd)**: Duplicates `oldfd` onto `newfd`, closing `newfd` first if needed. Used for redirection.
* **Redirection example:** In a child process, `dup2` is used to make stdout (fd 1) point to a file (for output redirection).

---

### Why It Matters (CNO/RE Context)

* File descriptors are fundamental to malware, CNO tools, and RE—stealthy tools often abuse FDs for persistence, hiding, or data exfiltration.
* Understanding redirection and FDs is essential for writing shells, keyloggers, backdoors, and loggers.

---

## 🧠 Key Concepts To Know

* Difference between high-level I/O (e.g., `fopen`) and low-level (system calls).
* How FDs are managed per-process.
* Relationship between FDs and Unix pipelines/redirection.
* How tools like `tee`, `cat`, and shells use FDs to multiplex input/output.

---

## ❓ Essential Questions

* What are the standard file descriptors, and how are they used?
* How does the kernel manage FDs for each process?
* What is the relationship between `open`/`read`/`write`/`close` and higher-level I/O in C?
* How does `dup2` enable output redirection or pipe chaining?

---

## 🧪 Lab: Write a Simple tee Clone

### **Instructions:**

1. **Write a C program:**

   * Reads from stdin (fd 0), writes each line both to stdout (fd 1) and to a file.
   * Opens the output file using `open()` (create/truncate).
   * Uses `read()` and `write()` (not `fgets`/`fprintf`).
   * Handles errors gracefully.

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <fcntl.h>
   #include <unistd.h>

   int main(int argc, char *argv[]) {
       if (argc != 2) {
           write(2, "Usage: tee_clone <outfile>\n", 28);
           exit(1);
       }
       int fd_out = open(argv[1], O_WRONLY | O_CREAT | O_TRUNC, 0644);
       if (fd_out < 0) {
           perror("open failed");
           exit(1);
       }
       char buf[1024];
       ssize_t n;
       while ((n = read(0, buf, sizeof(buf))) > 0) {
           write(1, buf, n);      // stdout
           write(fd_out, buf, n); // file
       }
       close(fd_out);
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o tee_clone lab3.c
   echo "Hello" | ./tee_clone output.txt
   cat output.txt
   ```

3. **Test shell redirection:**

   * Try redirecting stdin from a file:

     ```sh
     ./tee_clone copy.txt < input.txt
     ```
   * Try redirecting stdout to another file:

     ```sh
     echo "abc" | ./tee_clone out1.txt > out2.txt
     ```
   * Both `out1.txt` and `out2.txt` should contain the data.

---

### **Expected Observables:**

* Data from stdin is written both to stdout and to the specified file.
* Shell redirection (`<` and `>`) works as expected.
* If you use `ps` while the program runs, you can see open FDs via `/proc/<pid>/fd`.

---

## 📖 Further Reading

* [Beej’s Guide: File I/O](https://beej.us/guide/bgipc/html/split/fileio.html)
* [Linux man page: open()](https://man7.org/linux/man-pages/man2/open.2.html)
* [Linux man page: dup2()](https://man7.org/linux/man-pages/man2/dup2.2.html)
* [How Unix Pipes Work (LWN.net)](https://lwn.net/Articles/627601/)
* [UNIX I/O Redirection Explained (GeeksForGeeks)](https://www.geeksforgeeks.org/i-o-redirection-in-linux/)

---

*Document your code, terminal output, and a summary of how your program used FDs and redirection.*
