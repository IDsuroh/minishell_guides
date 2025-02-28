<!--
  SYSTEM FUNCTIONS DOCUMENTATION (v.o1)
  GitHub Markdown-Formatted
-->

# System Functions Documentation (v.o1)

![Terminal Banner](https://user-images.githubusercontent.com/placeholder-terminal.png)
<sup><sub>*Example placeholder image. Replace with your own.*</sub></sup>

<br />

## Table of Contents

1. [Introduction](#introduction)
2. [Standard I/O Library (`stdio.h`)](#standard-io-library-stdioh)
   - [printf](#printf)
3. [Standard Library (`stdlib.h`)](#standard-library-stdlibh)
   - [malloc, free, exit, getenv](#malloc-free-exit-getenv)
4. [Readline Functions (`readline/readline.h`, `readline/history.h`)](#readline-functions)
   - [readline, rl_clear_history, rl_on_new_line, rl_replace_line, rl_redisplay, add_history](#readline-rl_clear_history-rl_on_new_line-rl_replace_line-rl_redisplay-add_history)
5. [Signals (`signal.h`)](#signals-signalh)
   - [signal, sigaction, sigemptyset, sigaddset, kill](#signal-sigaction-sigemptyset-sigaddset-kill)
6. [Process and File Control (`unistd.h`, `sys/types.h`, `sys/wait.h`, `fcntl.h`, `sys/stat.h`)](#process-and-file-control)
   - [fork, wait, waitpid, wait3, wait4, execve](#fork-wait-waitpid-wait3-wait4-execve)
   - [write, access, open, read, close](#write-access-open-read-close)
   - [getcwd, chdir, stat, lstat, fstat, unlink](#getcwd-chdir-stat-lstat-fstat-unlink)
   - [dup, dup2, pipe](#dup-dup2-pipe)
7. [Directory Functions (`dirent.h`)](#directory-functions-direnth)
   - [opendir, readdir, closedir](#opendir-readdir-closedir)
8. [Terminal & Device Functions (`unistd.h`, `sys/ioctl.h`, `termios.h`, `termcap/curses)](#terminal--device-functions)
   - [isatty, ttyname, ttyslot, ioctl](#isatty-ttyname-ttyslot-ioctl)
   - [tcsetattr, tcgetattr](#tcsetattr-tcgetattr)
   - [tgetent, tgetflag, tgetnum, tgetstr, tgoto, tputs](#tgetent-tgetflag-tgetnum-tgetstr-tgoto-tputs)
9. [Error Handling (`errno.h` and related)](#error-handling-errnoh-and-related)
   - [strerror, perror](#strerror-perror)
10. [Conclusion](#conclusion)

---

## Introduction

This documentation (v.o1) describes **system-level functions** in C, grouped by the libraries or headers they originate from. Each section begins with a high-level overview of the library's purpose and relevant concepts, then explains each function with **proper usage** examples and **improper usage** pitfalls. The goal is to provide a clearer understanding of low-level C programming and operating system interactions.

---

## Standard I/O Library (`stdio.h`)

### Overview

- The `stdio.h` library provides functions for:
  - Reading from **standard input** (`stdin`)
  - Writing to **standard output** (`stdout`)
  - Formatted printing (`printf`, `fprintf`, etc.)

### `printf`

- **Header**: `#include <stdio.h>`
- **Description**: Prints formatted output to stdout.

**Proper Usage**:
```c
#include <stdio.h>

int main(void) {
    printf("Hello, %s!\n", "World"); // use correct format specifiers
    return 0;
}
```

**Improper Usage**:
```c
printf("Number: " 10); // Missing format specifier --> error
```

**Reason**: The compiler cannot parse `(\"Number: \" 10)` correctly, causing unexpected behavior.

---

## Standard Library (`stdlib.h`)

### Overview

- **Memory management**: `malloc`, `free`, etc.
- **Process termination**: `exit`
- **Environment variables**: `getenv`

### `malloc, free, exit, getenv`

#### `malloc`

- **Description**: Dynamically allocates the requested number of bytes and returns a pointer to the allocated memory.

**Proper Usage**:
```c
int *arr = malloc(10 * sizeof(int));
if (!arr) {
    perror("Failed to allocate");
    exit(EXIT_FAILURE);
}
```
**Reason**: Always check `malloc` return value for NULL.

#### `free`

- **Description**: Frees a previously allocated memory block.

**Proper Usage**:
```c
free(arr);
```

**Improper Usage**:
```c
free(arr);
free(arr); // Double free -> undefined behavior
```

#### `exit`

- **Description**: Terminates the current process immediately, returning a status code.

```c
exit(EXIT_SUCCESS);
```

#### `getenv`

- **Description**: Retrieves the value of an environment variable.

```c
char *path = getenv("PATH");
if (path) {
    printf("PATH: %s\n", path);
}
```

---

## Readline Functions

### Overview

These functions come from **`readline`** and **`history`** libraries. They provide command-line editing, history management, etc.

### `readline, rl_clear_history, rl_on_new_line, rl_replace_line, rl_redisplay, add_history`

- **`readline`**: Reads a line from stdin with support for line editing.
- **`rl_clear_history`**: Clears the in-memory command history.
- **`rl_on_new_line`**: Tells `readline` library that the cursor is on a new line.
- **`rl_replace_line`**: Replaces the current line in the editing buffer.
- **`rl_redisplay`**: Redisplays the current line after changes.
- **`add_history`**: Adds a string to the command history.

**Proper Usage**:
```c
#include <readline/readline.h>
#include <readline/history.h>

char *input = readline("myshell> ");
if (input) {
    add_history(input);
    // process input
}
```

---

## Signals (`signal.h`)

### Overview

**What are signals?**  
Signals are **software interrupts** sent to a process to indicate events like Ctrl+C (`SIGINT`), termination requests (`SIGTERM`), segmentation faults (`SIGSEGV`), etc. When a signal arrives, normal execution is interrupted, and a **signal handler** can take over if one is set.

### `signal, sigaction, sigemptyset, sigaddset, kill`

- **`signal(sig, handler)`**: Installs a simple handler function for `sig`.
- **`sigaction`**: A more configurable way to set handlers.
- **`sigemptyset`, `sigaddset`**: Helpers to build signal sets.
- **`kill(pid, sig)`**: Sends a signal to a process with a given PID.

**Proper Usage**:
```c
#include <signal.h>
#include <stdio.h>

void my_handler(int signum) {
    printf("Caught signal %d\n", signum);
}

int main() {
    signal(SIGINT, my_handler); // handle Ctrl+C
    while (1) { /* loop */ }
    return 0;
}
```

---

## Process and File Control

These functions reside in various headers:
- `unistd.h` (POSIX standard functions)
- `sys/wait.h` (waiting on processes)
- `fcntl.h` (file control flags)
- `sys/stat.h` (file status)

### `fork, wait, waitpid, wait3, wait4, execve`

- **`fork()`**: Creates a new child process.
- **`wait()`**, **`waitpid()`**: Wait for child processes to terminate.
- **`wait3()`, `wait4()`**: Like `wait`, but provides resource usage info (CPU time, memory usage).
- **`execve()`**: Replaces the current process with a new program.

**What is a child process?**  
After `fork()`, you have two processes running the same code but with different PIDs. The child inherits many attributes from the parent but can execute independently.

### `write, access, open, read, close`

- **`write(fd, buf, count)`**: Writes `count` bytes from `buf` to file descriptor `fd`.
- **`access(path, mode)`**: Checks if the process can access `path` with the given permissions.
- **`open(path, flags, mode)`**: Opens a file, returning a file descriptor.
- **`read(fd, buf, count)`**: Reads up to `count` bytes into `buf`.
- **`close(fd)`**: Closes a file descriptor.

### `getcwd, chdir, stat, lstat, fstat, unlink`

- **`getcwd(buf, size)`**: Gets the absolute path of the **current working directory** into `buf`.
- **`chdir(path)`**: Changes the current working directory.
- **`stat(path, &info)`**: Retrieves file info (follows symbolic links).
- **`lstat(path, &info)`**: Same as `stat` but **does not** follow links.
- **`fstat(fd, &info)`**: Gets file info from an open file descriptor.
- **`unlink(path)`**: Removes a filesystem link (deletes file if it’s the only link).

**What are symbolic links (symlinks)?**  
A special file that “points” to another file or directory.

### `dup, dup2, pipe`

- **`dup(fd)`**: Returns a new file descriptor referring to the same resource as `fd`.
- **`dup2(old_fd, new_fd)`**: Closes `new_fd` if necessary and makes it refer to the same open file as `old_fd`.
- **`pipe(pipefd)`**: Creates a unidirectional pipe (interprocess communication). `pipefd[0]` is read end, `pipefd[1]` is write end.

**How are `dup` and `dup2` different?**  
- `dup` picks the lowest available descriptor.
- `dup2` specifically reuses `new_fd`, closing it if open.

---

## Directory Functions (`dirent.h`)

### Overview

**Directory streams** allow reading the contents of a directory entry by entry.

### `opendir, readdir, closedir`

- **`opendir(path)`**: Opens a directory stream.
- **`readdir(dirp)`**: Returns the **next entry** (`struct dirent`) in the directory stream.
- **`closedir(dirp)`**: Closes the stream.

**What is the "next entry"?**  
Each call to `readdir` yields the next file or subdirectory name inside that directory.

---

## Terminal & Device Functions

### `isatty, ttyname, ttyslot, ioctl`

- **`isatty(fd)`**: Checks if `fd` is associated with a terminal device.
- **`ttyname(fd)`**: Returns the name (e.g. `/dev/pts/0`) of the terminal device.
- **`ttyslot()`**: Returns the slot number of the terminal (historical usage).
- **`ioctl(fd, request, ...)`**: Issues low-level device-specific I/O control commands.

**What is a terminal device?**  
Originally, a physical device (screen+keyboard). Modern OSes emulate this via pseudo-terminals (`/dev/pts/*`).

### `tcsetattr, tcgetattr`

- **Description**: Get/set the attributes of a terminal (e.g., canonical mode, echo, etc.).

```c
#include <termios.h>

struct termios oldt, newt;
tcgetattr(STDIN_FILENO, &oldt);
newt = oldt;
newt.c_lflag &= ~(ICANON | ECHO);
tcsetattr(STDIN_FILENO, TCSANOW, &newt);
```

### `tgetent, tgetflag, tgetnum, tgetstr, tgoto, tputs`

- **Termcap / curses**:
  - `tgetent(buffer, TERM)`: Loads the capabilities for the given `TERM` type.
  - `tgetflag(id)`: Checks for a boolean terminal capability.
  - `tgetnum(id)`: Retrieves a numeric capability (e.g., columns).
  - `tgetstr(id, area)`: Retrieves a string capability (e.g., clear screen sequence).
  - `tgoto(move_str, col, row)`: Builds a cursor-movement command.
  - `tputs(str, affcnt, putfunc)`: Outputs `str` (with padding) through `putfunc`.

**What is termcap?**  
A database describing how different terminals handle capabilities (cursor movement, screen clearing, etc.). `ncurses` or `termcap` might be used to handle these.

---

## Error Handling (`errno.h` and related)

### `strerror, perror`

- **`strerror(errno)`**: Returns a string describing the last error code.
- **`perror(msg)`**: Prints `msg`, then `strerror(errno)`.

**Usage**:
```c
FILE *fp = fopen("nonexistent.txt", "r");
if (!fp) {
    perror("fopen failed");
}
```

---

## Conclusion

This documentation (v.o1) provides **in-depth details** of common system calls and library functions in C:

- **Signals**: Handling software interrupts
- **Processes**: Creating (`fork`), executing (`execve`), waiting (`wait*`)
- **File I/O**: Opening, reading, writing, and closing descriptors
- **Directory**: Iterating over directory streams
- **Terminal**: Checking if a descriptor is a terminal, controlling attributes
- **Error Handling**: Using `perror` and `strerror` to diagnose failures

By understanding these fundamentals, you can build robust, low-level applications (like shells, system utilities, or networking daemons) in C. Always remember to **check return values** and handle **error cases** carefully.
