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
- **Description**: Prints formatted output to `stdout`.

**Detailed Explanation**:
- **Formatting**: `printf` uses format specifiers (like `%d`, `%s`) to format different data types.
- **Return Value**: Returns the total number of characters written, or a negative value if an error occurs.
- **Buffering**: Usually line-buffered on a terminal; flushes on newline or buffer fill.
- **Common Mistakes**:
  - Mismatched format specifiers (e.g., using `%d` for a `double`).
  - Not including a newline (can lead to buffered output not appearing immediately).

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

- **Header**: `#include <stdlib.h>`
- **Description**: Dynamically allocates the requested number of bytes and returns a pointer to the allocated memory.
- **Detailed Explanation**:
  - **Memory Allocation**: Reserves a contiguous block of memory on the heap.
  - **Initialization**: Contents of allocated memory are **uninitialized**.
  - **Failure**: Returns `NULL` if allocation fails; always check.
  - **Alignment**: Usually aligned suitably for any data type.

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

- **Header**: `#include <stdlib.h>`
- **Description**: Frees a previously allocated memory block.
- **Detailed Explanation**:
  - **Ownership**: Only free memory that was allocated by `malloc`/`calloc`/`realloc`.
  - **Double Free**: Calling `free` on the same pointer more than once leads to undefined behavior.
  - **After Free**: The pointer becomes invalid; do not dereference.

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
- **Detailed Explanation**:
  - **Cleanup**: Calls functions registered with `atexit`, flushes I/O buffers.
  - **Exit Codes**: Conventionally `EXIT_SUCCESS (0)` means success, `EXIT_FAILURE` means error.

```c
exit(EXIT_SUCCESS);
```

#### `getenv`

- **Description**: Retrieves the value of an environment variable.
- **Detailed Explanation**:
  - **Return**: Returns a pointer to a string in the environment list.
  - **Mutable?**: Some implementations allow modifying environment variables via `setenv`, but `getenv` pointer shouldn’t be freed.
  - **Usage**: Often used to get `PATH`, `HOME`, etc.

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

#### `readline`
- **Header**: `#include <readline/readline.h>`
- **Description**: Reads a line from stdin with support for line editing.
- **Detailed Explanation**:
  - Provides user-friendly features like arrow key navigation, backspace, etc.
  - Returns a **dynamically allocated** string, which should be freed by the caller.
  - If EOF is reached, returns `NULL`.

```c
char *input = readline("myshell> ");
if (input) {
    // Use input
}
```

#### `rl_clear_history`
- **Description**: Clears the in-memory command history list.

#### `rl_on_new_line`
- **Description**: Tells readline library that the cursor is on a new line (often used after a signal).

#### `rl_replace_line`
- **Description**: Replaces the current line in the editing buffer with new text.

#### `rl_redisplay`
- **Description**: Forces readline to redisplay the current buffer.

#### `add_history`
- **Description**: Adds a string to the in-memory history.
- **Detailed Explanation**:
  - The string becomes part of the history; can be accessed by up/down arrows in subsequent `readline` calls.

**Proper Usage**:
```c
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

#### `signal`
- **Description**: Installs a simple handler function for `sig`.
- **Detailed**:
  - Some signals (e.g., `SIGKILL`, `SIGSTOP`) cannot be caught or ignored.
  - `signal` is less flexible than `sigaction`.

```c
signal(SIGINT, my_handler);
```

#### `sigaction`
- **Description**: A more configurable way to set handlers with flags, masks.
- **Detailed**:
  - Allows blocking other signals while the handler is running.
  - `sa_sigaction` can be used for advanced handlers that need extra info.

```c
struct sigaction sa;
sa.sa_handler = my_handler;
sigemptyset(&sa.sa_mask);
sa.sa_flags = 0;
sigaction(SIGINT, &sa, NULL);
```

#### `sigemptyset`, `sigaddset`
- **Description**: Used to build or modify signal sets.
- **Detailed**:
  - `sigemptyset(&set);` initializes empty set.
  - `sigaddset(&set, SIGTERM);` adds SIGTERM to that set.

#### `kill`
- **Description**: Sends a signal to a process with given PID.
- **Detailed**:
  - `kill(pid, 0)` can be used to check existence/permission of a process (no actual signal is sent).

```c
kill(pid, SIGTERM);
```

---

## Process and File Control

These functions reside in various headers:
- `unistd.h` (POSIX standard functions)
- `sys/wait.h` (waiting on processes)
- `fcntl.h` (file control flags)
- `sys/stat.h` (file status)

### `fork, wait, waitpid, wait3, wait4, execve`

#### `fork`
- **Description**: Creates a new child process.
- **Detailed**:
  - Returns `0` to the child, returns child PID to the parent, or `-1` on error.
  - Child inherits file descriptors, environment, etc., but has its own PID.

```c
pid_t pid = fork();
if (pid == 0) {
    // child
} else if (pid > 0) {
    // parent
} else {
    perror("fork failed");
}
```

#### `wait`, `waitpid`
- **Description**: Wait for a child process to terminate.
- **Detailed**:
  - `wait(&status);` waits for any child.
  - `waitpid(pid, &status, 0);` waits for a specific child.
  - Prevents zombies.

#### `wait3`, `wait4`
- **Description**: Like `wait`, but also provides **resource usage** in a `struct rusage`.
- **Detailed**:
  - CPU time, memory usage, etc. can be retrieved.

#### `execve`
- **Description**: Replaces the current process image with a new program.
- **Detailed**:
  - If successful, never returns.
  - If fails, returns `-1` and sets `errno`.

```c
char *argv[] = {"/bin/ls", "-l", NULL};
execve("/bin/ls", argv, envp);
```

### `write, access, open, read, close`

#### `write`
- **Description**: Writes `count` bytes from `buf` to file descriptor `fd`.
- **Detailed**:
  - Returns number of bytes written or `-1` on error.
  - May write fewer bytes than requested.

#### `access`
- **Description**: Checks accessibility of a file (read, write, execute).
- **Detailed**:
  - Not always reliable for security checks (TOCTOU race conditions).

#### `open`
- **Description**: Opens a file and returns a file descriptor.
- **Detailed**:
  - Flags like `O_RDONLY`, `O_WRONLY`, `O_CREAT` control behavior.
  - Mode is used if creating a file (e.g., `0664`).

#### `read`
- **Description**: Reads up to `count` bytes from `fd` into `buf`.
- **Detailed**:
  - Returns number of bytes read; `0` means EOF if reading from file.

#### `close`
- **Description**: Closes a file descriptor.
- **Detailed**:
  - Frees up the descriptor for reuse.

### `getcwd, chdir, stat, lstat, fstat, unlink`

#### `getcwd`
- **Description**: Gets the absolute path of the current working directory.
- **Detailed**:
  - Writes up to `size` bytes into `buf`.
  - If the path is longer than `size`, `NULL` is returned.

#### `chdir`
- **Description**: Changes the current working directory.
- **Detailed**:
  - Affects how relative paths are resolved.

#### `stat`, `lstat`, `fstat`
- **Description**: Retrieve file info (size, permissions, timestamps, etc.).
- **Detailed**:
  - `stat` follows symlinks, `lstat` does not.
  - `fstat` operates on an already open file descriptor.

#### `unlink`
- **Description**: Removes a filesystem link to a file.
- **Detailed**:
  - If this is the last link, the file data is freed once no process holds it open.

**What are symbolic links (symlinks)?**
- A file that points to another path.

### `dup, dup2, pipe`

#### `dup`
- **Description**: Returns a new file descriptor referring to the same resource.
- **Detailed**:
  - The new descriptor is the lowest-numbered available slot.

#### `dup2`
- **Description**: Forces duplication into a chosen descriptor number.
- **Detailed**:
  - If `new_fd` is open, it is silently closed first.

#### `pipe`
- **Description**: Creates a unidirectional data channel.
- **Detailed**:
  - `pipefd[0]` is read end, `pipefd[1]` is write end.

**How are `dup` and `dup2` different?**  
- `dup` picks the lowest available descriptor.
- `dup2` specifically reuses `new_fd`.

---

## Directory Functions (`dirent.h`)

### Overview

**Directory streams** allow reading the contents of a directory entry by entry.

### `opendir, readdir, closedir`

#### `opendir`
- **Description**: Opens a directory stream.
- **Detailed**:
  - Returns a pointer to a `DIR` structure if successful.
  - `NULL` on error.

#### `readdir`
- **Description**: Returns the **next entry** (`struct dirent`) in the directory.
- **Detailed**:
  - Each call yields one file/subdirectory name.
  - Returns `NULL` if no more entries.

#### `closedir`
- **Description**: Closes the directory stream.
- **Detailed**:
  - Frees resources associated with the `DIR*`.

**What is the "next entry"?**  
It’s each successive `struct dirent` from the directory listing.

---

## Terminal & Device Functions

### `isatty, ttyname, ttyslot, ioctl`

#### `isatty`
- **Description**: Checks if `fd` is associated with a terminal device.
- **Detailed**:
  - Returns nonzero if it is, `0` otherwise.

#### `ttyname`
- **Description**: Returns a string naming the terminal (e.g., `/dev/pts/0`).

#### `ttyslot`
- **Description**: Returns the slot number of the terminal (legacy concept).

#### `ioctl`
- **Description**: Performs device-specific I/O operations.
- **Detailed**:
  - Called with requests like `TIOCGWINSZ` to get terminal size.

**What is a terminal device?**  
Originally physical; now often a pseudo-terminal (`/dev/pts/*`) in a shell.

### `tcsetattr, tcgetattr`

- **Description**: Get/set terminal attributes (canonical mode, echo, etc.).
- **Detailed**:
  - `tcgetattr(fd, &termios_p)` obtains the current settings.
  - `tcsetattr(fd, TCSANOW, &termios_p)` updates them immediately.

```c
#include <termios.h>

struct termios oldt, newt;
tcgetattr(STDIN_FILENO, &oldt);
newt = oldt;
newt.c_lflag &= ~(ICANON | ECHO);
tcsetattr(STDIN_FILENO, TCSANOW, &newt);
```

### `tgetent, tgetflag, tgetnum, tgetstr, tgoto, tputs`

#### `tgetent`
- **Description**: Loads terminal capability data for a given `TERM`.
- **Detailed**:
  - Reads from the termcap/terminfo database.

#### `tgetflag`
- **Description**: Checks a boolean capability (e.g., whether the terminal can do auto-margins).

#### `tgetnum`
- **Description**: Retrieves a numeric capability (e.g., columns count `co`).

#### `tgetstr`
- **Description**: Retrieves a string capability (e.g., clear screen `cl`).

#### `tgoto`
- **Description**: Builds a cursor-movement command from a format string.

#### `tputs`
- **Description**: Outputs a string with appropriate padding.

**What is termcap?**  
A database describing how different terminals handle capabilities (cursor movement, screen clearing, etc.). `ncurses` or `termcap` might be used to handle these.

---

## Error Handling (`errno.h` and related)

### `strerror, perror`

#### `strerror`
- **Description**: Returns a string describing the last error code in `errno`.
- **Detailed**:
  - The returned pointer should not be freed.
  - This string is statically allocated or resides in a system buffer.

#### `perror`
- **Description**: Prints `msg`, then `strerror(errno)`.
- **Detailed**:
  - Typically used immediately after a failing syscall or library function.

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
