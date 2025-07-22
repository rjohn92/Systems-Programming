# ⚙️ Systems Programming in C — Week 11

## 📚 Lecture Notes: Filesystems and Directories

### stat, lstat, fstat, and Inodes

* **Inode:** A data structure that stores information about a file (metadata), not its name or content. Contains size, permissions, timestamps, and pointers to file data.
* **stat(path, \&buf):** Retrieves inode and metadata for a file or directory (follows symlinks).
* **lstat(path, \&buf):** Like stat, but does not follow symlinks (returns info about the link itself).
* **fstat(fd, \&buf):** Like stat, but works on an open file descriptor.

---

### Directories: mkdir, opendir, readdir, Directory Traversal

* **mkdir(path, mode):** Creates a new directory.
* **opendir(path):** Opens a directory for reading entries.
* **readdir(dirp):** Reads directory entries (`struct dirent`). Use in a loop to enumerate files/subdirectories.
* **Recursive traversal:** Recursively descend into directories to list all files (like `find`).

---

### Permissions, Ownership, chmod, chown

* **Permissions:** Read (r), write (w), execute (x) bits for user/group/other.
* **Ownership:** Each file has a user and group owner (UID, GID).
* **chmod(path, mode):** Change file permissions.
* **chown(path, uid, gid):** Change file ownership.
* **umask:** Sets default permissions for new files.

---

### Why It Matters (CNO/RE Context)

* File/directory metadata is essential for privilege escalation, file hiding, forensics, and persistence techniques.
* Malware/CNO tools often manipulate permissions, hide files in deep directories, or abuse symlinks.
* Many real-world vulnerabilities relate to incorrect directory traversal, permission checks, or ownership mistakes.

---

## 🧠 Key Concepts To Know

* How inodes enable hard links, but not soft (symbolic) links
* How to distinguish files, directories, symlinks with stat/lstat
* How recursive file walking is implemented (stack/queue or recursion)
* Security risks: symlink race conditions, world-writable directories, sticky bits

---

## ❓ Essential Questions

* How can you tell if a file is a directory or symlink using stat/lstat?
* What happens if you don’t handle “.” and “..” correctly in recursive traversal?
* Why is `chmod 777` dangerous? When (if ever) is it justified?
* What is the difference between owner, group, and other permissions?

---

## 🧪 Lab: Recursive Directory Walker

### **Instructions:**

1. **Write a C program:**

   * Accepts a directory path as a command-line argument.
   * Recursively visits all files and subdirectories under the path.
   * For each file, prints:

     * Name
     * File type (regular, directory, symlink)
     * Permissions (mode)
     * Size
     * Owner UID/GID
   * Uses `opendir()`, `readdir()`, `stat()`, and `lstat()`.
   * Skips “.” and “..” to avoid infinite loops.

   Example skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <dirent.h>
   #include <sys/stat.h>
   #include <string.h>

   void walk(const char *path) {
       struct dirent *entry;
       DIR *dir = opendir(path);
       if (!dir) return;
       while ((entry = readdir(dir)) != NULL) {
           if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0)
               continue;
           char fullpath[4096];
           snprintf(fullpath, sizeof(fullpath), "%s/%s", path, entry->d_name);
           struct stat st;
           lstat(fullpath, &st);
           printf("%s\t", fullpath);
           if (S_ISDIR(st.st_mode)) {
               printf("[DIR]\n");
               walk(fullpath); // recursive
           } else if (S_ISLNK(st.st_mode)) {
               printf("[SYMLINK]\n");
           } else if (S_ISREG(st.st_mode)) {
               printf("[FILE] Size: %ld Mode: %o Owner: %d/%d\n", st.st_size, st.st_mode & 0777, st.st_uid, st.st_gid);
           } else {
               printf("[OTHER]\n");
           }
       }
       closedir(dir);
   }

   int main(int argc, char *argv[]) {
       if (argc != 2) {
           printf("Usage: %s <directory>\n", argv[0]);
           return 1;
       }
       walk(argv[1]);
       return 0;
   }
   ```

2. **Compile and run:**

   ```sh
   gcc -o walker lab11.c
   ./walker /path/to/dir
   ```

3. **Check output:**

   * Ensure all files, directories, and symlinks are reported with correct metadata.

---

### **Expected Observables:**

* Recursively traverses and prints every file and directory
* Prints type, permissions, size, and ownership for each
* Skips "." and ".." to avoid infinite recursion

---

## 📖 Further Reading

* [Beej’s Guide: Directories and Files](https://beej.us/guide/bgipc/html/split/files.html)
* [Linux man page: stat()](https://man7.org/linux/man-pages/man2/stat.2.html)
* [Linux man page: readdir()](https://man7.org/linux/man-pages/man3/readdir.3.html)
* [Linux man page: chmod()](https://man7.org/linux/man-pages/man2/chmod.2.html)
* [Understanding inodes (DigitalOcean)](https://www.digitalocean.com/community/tutorials/inodes-usage-linux)

---

*Document your code, sample output, and any corner cases (e.g., symlinks, permissions errors) encountered during the walk.*
