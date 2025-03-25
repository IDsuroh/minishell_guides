<!--
  Explanation of the Functions of minishell
  GitHub Markdown-Formatted
-->

```txt
============================================================
=    EXPLANATION OF THE FUNCTIONS OF MINISHELL (ASCII)    =
============================================================
```

<br />

## Table of Contents

1. [Introduction](#introduction)
2. [Standard I/O Library (`stdio.h`)](#standard-io-library-stdioh)
   - [printf](#printf)
3. [Standard Library (`stdlib.h`)](#standard-library-stdlibh)
   - [malloc, free, exit, getenv](#malloc-free-exit-getenv)
4. [Readline Functions (`readline/readline.h`, `readline/history.h`)](#readline-functions-readlinereadlineh-readlinehistoryh))
   - [readline, rl_clear_history, rl_on_new_line, rl_replace_line, rl_redisplay, add_history](#readline-rl_clear_history-rl_on_new_line-rl_replace_line-rl_redisplay-add_history)
5. [Signals (`signal.h`)](#signals-signalh)
   - [signal, sigaction, sigemptyset, sigaddset, kill](#signal-sigaction-sigemptyset-sigaddset-kill)
6. [Process and File Control (`unistd.h`, `sys/types.h`, `sys/wait.h`, `fcntl.h`, `sys/stat.h`)](#process-management-functions-documentation)
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

This documentation (v.o1) describes **system-level functions** relevant to **minishell**, grouped by the libraries or headers they originate from. Each section begins with a high-level overview of the library's purpose and relevant concepts, then explains each function with **proper usage** examples and **improper usage** pitfalls. The goal is to provide a clearer understanding of low-level C programming and operating system interactions, specifically for a shell-like environment.

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

## Readline Functions (`readline/readline.h`, `readline/history.h`)

### Overview

The **Readline library** provides functionalities for command-line editing and history management. It's widely used in interactive programs, particularly shells, to allow users to comfortably edit commands, manage command histories, and enhance usability.

### `readline, rl_clear_history, rl_on_new_line, rl_replace_line, rl_redisplay, add_history`

### Functions Explained

#### `readline`

- **Header**: `#include <readline/readline.h>`
- **Description**: Reads user input from standard input with enhanced editing capabilities.
- **Behavior**:
  - Supports line editing (e.g., cursor movement, deletion).
  - Returns a dynamically allocated string, which must be manually freed by the caller.
  - Returns `NULL` upon reaching EOF (`Ctrl+D`).

**Proper Usage Example:**
```c
#include <readline/readline.h>
#include <stdlib.h>

int main(void) {
    char *input = readline("minishell> ");
    if (!input) {
        printf("EOF received, exiting.\n");
        exit(EXIT_SUCCESS);
    }
    printf("You entered: %s\n", input);
    free(input);
    return 0;
}
```

**Common Mistakes:**
- Forgetting to free memory allocated by `readline`.
- Not checking for NULL return value (EOF handling).

#### `add_history`

- **Header**: `#include <readline/history.h>`
- **Description**: Adds a line to the history list, enabling retrieval via arrow keys in subsequent calls.

**Proper Usage Example:**
```c
#include <readline/readline.h>
#include <readline/history.h>
#include <stdlib.h>

int main(void) {
    char *input = readline("minishell> ");
    if (input && *input) {
        add_history(input);
    }
    free(input);
    return 0;
}
```

**Common Mistakes:**
- Adding empty or null strings to history.

#### `rl_clear_history`

- **Header**: `#include <readline/history.h>`
- **Description**: Clears all entries in the history, typically used when restarting or resetting command input history.

**Proper Usage Example:**
```c
#include <readline/history.h>

void clear_shell_history(void) {
    rl_clear_history();
    printf("History cleared successfully.\n");
}
```

#### `rl_on_new_line`

- **Header**: `#include <readline/readline.h>`
- **Description**: Informs readline library that the cursor has moved to a new line, frequently used after signal handling to reposition the cursor correctly.

**Proper Usage Example (Signal Handling):**
```c
#include <readline/readline.h>
#include <signal.h>

void sigint_handler(int sig) {
    rl_on_new_line();
    rl_replace_line("", 0);
    rl_redisplay();
}

int main(void) {
    signal(SIGINT, sigint_handler);
    char *input;
    while ((input = readline("minishell> "))) {
        free(input);
    }
    return 0;
}
```

#### `rl_replace_line`

- **Header**: `#include <readline/readline.h>`
- **Description**: Replaces the content of the readline buffer with the given string.

**Proper Usage Example:**
```c
#include <readline/readline.h>

void replace_current_line(const char *new_line) {
    rl_replace_line(new_line, 0);
    rl_redisplay();
}
```

#### `rl_redisplay`

- **Header**: `#include <readline/readline.h>`
- **Description**: Forces readline to refresh and redisplay the current editing buffer. Essential after modifications to the buffer or cursor position.

**Proper Usage Example:**
```c
#include <readline/readline.h>

void refresh_display(void) {
    rl_redisplay();
}
```

**General Best Practices**:
- Always handle memory allocation carefully (especially strings returned by `readline`).
- Properly manage history to enhance user experience.
- Implement thorough signal handling to gracefully manage interruptions and user actions.


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

#### `sigemptyset`
- **Description**: Initializes a signal set (sigset_t) to exclude all signals. Commonly used before explicitly adding signals to a set.
- **Detailed**:
  - Sets the given signal set to be empty (no signals included).
  - Essential step before manipulating signal masks using sigaddset.

```c
#include <signal.h>
#include <stdio.h>

int main(void) {
    sigset_t set;

    if (sigemptyset(&set) == -1) {
        perror("sigemptyset failed");
        return 1;
    }

    // Signal set is now empty; you can add signals explicitly
    return 0;
}
```

**Common Mistakes**:
Forgetting to initialize the signal set before adding signals, leading to unpredictable behavior.
Not checking the return value for error handling.


#### `sigaddset`

- **Description**: Adds a specific signal (e.g., SIGINT, SIGTERM) to a signal set.
- **Detailed**:
  - Enables the addition of specific signals to a previously initialized signal set.
  - Commonly used to block signals or specify signals in functions like sigaction.

**Proper Usage Example**:
```c
#include <signal.h>
#include <stdio.h>

int main(void) {
    sigset_t set;

    if (sigemptyset(&set) == -1) {
        perror("sigemptyset failed");
        return 1;
    }

    if (sigaddset(&set, SIGTERM) == -1) {
        perror("sigaddset failed");
        return 1;
    }

    // SIGTERM now added to set; can be used with sigaction or sigprocmask
    return 0;
}
```

**Common Mistakes**:
Not checking the return value of sigaddset, which could indicate failures (e.g., invalid signal numbers).
Attempting to use a non-initialized set with sigaddset.

#### `kill`
- **Description**: Sends a signal to a process or group of processes specified by a process ID (PID).
- **Detailed**:
  - Sends a specified signal (e.g., SIGTERM, SIGKILL) to the process with the given PID.
  - If signal is 0, no signal is sent, but the function performs error checking. It can be used to verify if the process exists and whether the caller has permission to send signals to it.

**Proper Usage Example**:
```c
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
        return 1;
    }

    pid_t pid = atoi(argv[1]);

    if (kill(pid, SIGTERM) == -1) {
        perror("kill failed");
        return 1;
    }

    printf("SIGTERM successfully sent to PID %d\n", pid);
    return 0;
}
```
Checking if a Process Exists Using kill(pid, 0) Example:

```c
#include <signal.h>
#include <stdio.h>

int check_process_exists(pid_t pid) {
    if (kill(pid, 0) == 0) {
        printf("Process %d exists and is accessible.\n", pid);
        return 1;
    } else {
        perror("kill check failed");
        return 0;
    }
}

int main(void) {
    pid_t pid_to_check = 12345; // Replace with actual PID
    check_process_exists(pid_to_check);
    return 0;
}
```
**Common Mistakes**:
Sending inappropriate signals (e.g., using SIGKILL when a graceful termination with SIGTERM is preferred).
Not verifying the PID is valid before sending signals.
Misinterpreting kill(pid, 0) as sending a signal when it actually only checks process validity and permissions.

**Best Practices**:
Always properly initialize signal sets before manipulating them.
Handle return values and errors meticulously to ensure robust code.
Use kill(pid, 0) cautiously for checking process existence or permissions.
---


# Process Management Functions Documentation

This document describes various process management functions, including their headers, detailed behavior, proper usage examples, common mistakes, and a table comparing their features.

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

## wait

**Header:** `#include <sys/wait.h>`

**Description:**  
Waits for any child process to terminate.

**Detailed:**
- Suspends the calling process until any child process exits.
- The child's termination status is stored in the provided status integer.
- Helps prevent zombie processes by collecting child termination statuses.

**Proper Usage Example:**

```c
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    pid_t pid = fork();

    if (pid == 0) {
        // Child process
        _exit(0);
    } else if (pid > 0) {
        int status;
        if (wait(&status) == -1)
            perror("wait failed");
        else if (WIFEXITED(status))
            printf("Child exited with status %d\n", WEXITSTATUS(status));
    } else {
        perror("fork failed");
    }

    return 0;
}
```

**Common Mistakes:**
- Ignoring return values.
- Missing potential error checks.
- Misinterpreting the status without using proper macros (e.g., `WIFEXITED`, `WEXITSTATUS`).

---

## waitpid

**Header:** `#include <sys/wait.h>`

**Description:**  
Waits specifically for the termination of a designated child process.

**Detailed:**
- Waits for the child identified by its PID.
- Offers additional control through flags like `WNOHANG` (non-blocking) or `WUNTRACED`.
- Used to collect exit statuses and avoid zombies.

**Proper Usage Example:**

```c
#include <sys/wait.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    pid_t pid = fork();

    if (pid == 0) {
        // Child process
        sleep(2);
        _exit(0);
    } else if (pid > 0) {
        int status;
        if (waitpid(pid, &status, 0) == -1)
            perror("waitpid failed");
        else if (WIFEXITED(status))
            printf("Child %d exited with status %d\n", pid, WEXITSTATUS(status));
    } else {
        perror("fork failed");
    }

    return 0;
}
```

**Common Mistakes:**
- Incorrect usage of flags (e.g., `WNOHANG`) causing unintended behavior.
- Failing to handle return values and errors.

---

## wait3

**Header:** `#include <sys/wait.h>`

**Description:**  
Similar to `wait`, waits for any child process and also provides resource usage statistics.

**Detailed:**
- Collects CPU usage, memory consumption, context switches, and more in a `struct rusage`.
- Allows additional flags for customized waiting behavior.

**Proper Usage Example:**

```c
#include <sys/wait.h>
#include <sys/resource.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    pid_t pid = fork();

    if (pid == 0) {
        // Child process
        sleep(1);
        _exit(0);
    } else if (pid > 0) {
        int status;
        struct rusage usage;
        if (wait3(&status, 0, &usage) == -1) {
            perror("wait3 failed");
        } else if (WIFEXITED(status)) {
            printf("Child exited with status %d\n", WEXITSTATUS(status));
            printf("User CPU time: %ld.%06ld sec\n", usage.ru_utime.tv_sec, usage.ru_utime.tv_usec);
            printf("System CPU time: %ld.%06ld sec\n", usage.ru_stime.tv_sec, usage.ru_stime.tv_usec);
        }
    } else {
        perror("fork failed");
    }

    return 0;
}
```

**Common Mistakes:**
- Misinterpreting resource usage data in `struct rusage`.
- Neglecting proper status and error checks.

---

## wait4

**Header:** `#include <sys/wait.h>`

**Description:**  
Combines the functionalities of `waitpid` and `wait3` by waiting for a specific child process and providing detailed resource tracking.

**Detailed:**
- Waits for the designated child process based on its PID.
- Combines specific waiting with collection of resource usage statistics via `struct rusage`.

**Proper Usage Example:**

```c
#include <sys/wait.h>
#include <sys/resource.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    pid_t pid = fork();

    if (pid == 0) {
        // Child process
        sleep(1);
        _exit(0);
    } else if (pid > 0) {
        int status;
        struct rusage usage;
        if (wait4(pid, &status, 0, &usage) == -1) {
            perror("wait4 failed");
        } else if (WIFEXITED(status)) {
            printf("Child %d exited with status %d\n", pid, WEXITSTATUS(status));
            printf("User CPU time: %ld.%06ld sec\n", usage.ru_utime.tv_sec, usage.ru_utime.tv_usec);
            printf("System CPU time: %ld.%06ld sec\n", usage.ru_stime.tv_sec, usage.ru_stime.tv_usec);
        }
    } else {
        perror("fork failed");
    }

    return 0;
}
```

**Common Mistakes:**
- Incorrect handling or overlooking of resource usage statistics.
- Confusing its usage with `waitpid` and forgetting the additional `struct rusage` parameter.

---

## Comparison Table

| Function | Waits for         | Specific PID? | Provides Resource Usage? | Flags Supported? |
|----------|-------------------|---------------|--------------------------|------------------|
| wait     | Any child process | No            | No                       | No               |
| waitpid  | Specific child    | Yes           | No                       | Yes              |
| wait3    | Any child process | No            | Yes                      | Yes              |
| wait4    | Specific child    | Yes           | Yes                      | Yes              |

**Usage Recommendations:**
- **Use `wait`** if simply waiting for any child without requiring additional data.
- **Use `waitpid`** for waiting on a specific child or when non-blocking flags are needed.
- **Use `wait3`** when retrieving resource usage statistics for any terminating child.
- **Use `wait4`** for precise monitoring involving both process termination and resource statistics.

---

## execve

**Header:** `#include <unistd.h>`

**Description:**  
Replaces the current running process with a new program.

**Detailed:**
- Requires an explicit file path to the new executable, along with pointers to arguments (`argv`) and an array of environment variables (`envp`).
- On success, `execve` never returns; on failure, it returns `-1` and sets `errno`.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    char *argv[] = {"/bin/ls", "-l", NULL};
    char *envp[] = {NULL};

    execve("/bin/ls", argv, envp);

    // Only reached if execve fails
    perror("execve failed");
    return 1;
}
```

**Common Mistakes:**
- Assuming the process continues running after a successful call to `execve` (it does not).
- Incorrectly initializing the `argv` or `envp` arrays (both must be terminated by `NULL`).

**Note:**  
Always use the appropriate macros (such as `WIFEXITED`, `WEXITSTATUS`, `WIFSIGNALED`, etc.) to interpret the exit status for all wait-family functions to ensure robust error handling and maintain program stability.


## File I/O Functions (unistd.h, fcntl.h)

### `write, access, open, read, close`

## Overview

These system-level functions facilitate low-level input/output operations, allowing direct interaction with file descriptors, checking permissions, and managing file states efficiently.

---

## write

**Header:** `#include <unistd.h>`

**Description:**  
Writes up to `count` bytes from the provided buffer (`buf`) to the given file descriptor (`fd`).

**Detailed:**
- Returns the number of bytes actually written.
- Can return fewer bytes than requested (partial writes).
- Returns -1 on error (sets `errno`).

**Proper Usage Example:**

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(void) {
    int fd = open("output.txt", O_WRONLY | O_CREAT, 0644);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }

    const char *text = "Hello, minishell!\n";
    ssize_t written = write(fd, text, 18);
    if (written == -1)
        perror("write failed");

    close(fd);
    return 0;
}
```

**Common Mistakes:**
- Not handling partial writes (fewer bytes than requested).
- Ignoring return values and errors.


## access

**Header:** `#include <unistd.h>`

**Description:**  
Checks whether the calling process has permissions (read/write/execute) for a file.

**Detailed:**
- Useful for preliminary checks but can introduce TOCTOU (time-of-check, time-of-use) race conditions if improperly used.
- Returns 0 on success (permissions exist), -1 otherwise.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    if (access("file.txt", R_OK | W_OK) == 0)
        printf("Read/write permissions available.\n");
    else
        perror("access check failed");

    return 0;
}
```

**Common Mistakes:**
- Relying solely on `access` for security-critical operations (potential race conditions).
- Not handling the return value and error properly.


## open

**Header:** `#include <fcntl.h>`

**Description:**  
Opens or creates a file, returning a file descriptor.

**Detailed:**
- The file descriptor allows subsequent operations (read, write, close).
- Flags like `O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_CREAT`, `O_APPEND` control file access modes and behaviors.
- The mode parameter sets permissions for newly created files.

**Proper Usage Example:**

```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    int fd = open("example.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }
    // Use fd...
    close(fd);
    return 0;
}
```

**Common Mistakes:**
- Incorrect or missing mode parameter when using `O_CREAT`.
- Forgetting to check for errors.


## read

**Header:** `#include <unistd.h>`

**Description:**  
Reads up to `count` bytes from a file descriptor (`fd`) into a buffer (`buf`).

**Detailed:**
- Returns the number of bytes actually read.
- Returns 0 on EOF (end-of-file).
- Returns -1 on error.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(void) {
    int fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }

    char buffer[100];
    ssize_t bytes_read = read(fd, buffer, sizeof(buffer) - 1);
    if (bytes_read == -1) {
        perror("read failed");
    } else if (bytes_read == 0) {
        printf("Reached EOF.\n");
    } else {
        buffer[bytes_read] = '\0';
        printf("Read %ld bytes: %s\n", bytes_read, buffer);
    }

    close(fd);
    return 0;
}
```

**Common Mistakes:**
- Not handling EOF (0 bytes read) separately.
- Assuming the buffer is null-terminated automatically.


## close

**Header:** `#include <unistd.h>`

**Description:**  
Closes an open file descriptor.

**Detailed:**
- Frees resources associated with the file descriptor.
- Important to prevent resource leaks (running out of descriptors).

**Proper Usage Example:**

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(void) {
    int fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }
    
    if (close(fd) == -1)
        perror("close failed");

    return 0;
}
```

**Common Mistakes:**
- Not checking the return value, potentially missing errors.
- Forgetting to close descriptors leading to leaks.

---

# Directory and File System Functions (unistd.h, sys/stat.h)

## Overview

### `getcwd, chdir, stat, lstat, fstat, unlink`

These functions manage directories, handle file paths, and retrieve information about files and directories. They also manage filesystem links and the current working directory.


## getcwd

**Header:** `#include <unistd.h>`

**Description:**  
Retrieves the absolute pathname of the current working directory.

**Detailed:**
- Stores the absolute path into the provided buffer `buf` up to `size` bytes.
- If the buffer is too small or an error occurs, returns `NULL` and sets `errno`.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>
#include <limits.h>

int main(void) {
    char cwd[PATH_MAX];

    if (getcwd(cwd, sizeof(cwd)) == NULL) {
        perror("getcwd failed");
        return 1;
    }

    printf("Current working directory: %s\n", cwd);
    return 0;
}
```

**Common Mistakes:**
- Providing an insufficient buffer size causing unexpected failures.
- Ignoring the return value and error handling.


## chdir

**Header:** `#include <unistd.h>`

**Description:**  
Changes the calling process’s current working directory to the specified path.

**Detailed:**
- Alters the directory context for relative path operations.
- Returns `0` on success; `-1` and sets `errno` on failure.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    if (chdir("/tmp") == -1) {
        perror("chdir failed");
        return 1;
    }

    printf("Directory successfully changed to /tmp\n");
    return 0;
}
```

**Common Mistakes:**
- Not handling the return value, causing subsequent operations to fail silently if the directory change fails.


## stat

**Header:** `#include <sys/stat.h>`

**Description:**  
Retrieves detailed information about a file or symbolic link.

**Detailed:**
- Follows symbolic links and provides information about the file or its target.
- Information includes file type, permissions, size, timestamps, and inode number.

**Proper Usage Example:**

```c
#include <sys/stat.h>
#include <stdio.h>

int main(void) {
    struct stat sb;

    if (stat("file.txt", &sb) == -1) {
        perror("stat failed");
        return 1;
    }

    printf("File size: %lld bytes\n", (long long)sb.st_size);
    printf("File permissions: %o\n", sb.st_mode & 0777);
    return 0;
}
```

**Common Mistakes:**
- Misinterpreting file types by not using macros such as `S_ISREG`, `S_ISDIR`, or `S_ISLNK`.
- Ignoring return values, which can lead to reading invalid data.


## lstat

**Header:** `#include <sys/stat.h>`

**Description:**  
Retrieves information about a file or symbolic link without following symbolic links.

**Detailed:**
- Useful for obtaining details specifically about the symlink itself rather than its target.

**Proper Usage Example:**

```c
#include <sys/stat.h>
#include <stdio.h>

int main(void) {
    struct stat sb;

    if (lstat("link_to_file.txt", &sb) == -1) {
        perror("lstat failed");
        return 1;
    }

    if (S_ISLNK(sb.st_mode))
        printf("It is a symbolic link.\n");
    else
        printf("Not a symbolic link.\n");

    return 0;
}
```

**Common Mistakes:**
- Confusing `lstat` with `stat` when the intention is to get information about the symlink itself.
- Ignoring the return value, potentially leading to mishandled error states.


## fstat

**Header:** `#include <sys/stat.h>`

**Description:**  
Retrieves information about a file based on an already opened file descriptor.

**Detailed:**
- Useful when a file descriptor is already open, thus avoiding additional lookups by file path.

**Proper Usage Example:**

```c
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    int fd = open("file.txt", O_RDONLY);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }

    struct stat sb;
    if (fstat(fd, &sb) == -1) {
        perror("fstat failed");
        close(fd);
        return 1;
    }

    printf("File size: %lld bytes\n", (long long)sb.st_size);
    close(fd);
    return 0;
}
```

**Common Mistakes:**
- Not checking the return value, which can result in invalid data reads.
- Misinterpreting file type information by not using the proper macros.


## unlink

**Header:** `#include <unistd.h>`

**Description:**  
Removes a file's directory entry (link).

**Detailed:**
- Deletes the specified link from the filesystem.
- If it is the last link and no processes have the file open, the file's data is deleted immediately. Otherwise, deletion is deferred until no processes are using it.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    if (unlink("old_file.txt") == -1) {
        perror("unlink failed");
        return 1;
    }

    printf("File successfully unlinked.\n");
    return 0;
}
```

**Common Mistakes:**
- Expecting immediate file data removal even if other processes have the file open.
- Ignoring errors such as permissions issues or missing file paths.


## Symbolic Links (Symlinks)

### What are symbolic links (symlinks)?

A symbolic link (symlink) is a special file type that contains a reference to another file or directory. Symlinks provide an indirect way of accessing file resources, allowing flexibility in file system structure and organization.

### Creating a Symlink

You can create a symlink using the following bash command:

```bash
ln -s original_file.txt symlink_to_original.txt
```

### Checking if a File is a Symlink

**Proper Usage Example (using lstat):**

```c
#include <sys/stat.h>
#include <stdio.h>

int main(void) {
    struct stat sb;
    if (lstat("symlink_to_original.txt", &sb) == -1) {
        perror("lstat failed");
        return 1;
    }

    if (S_ISLNK(sb.st_mode))
        printf("This is a symbolic link.\n");
    else
        printf("This is not a symbolic link.\n");

    return 0;
}
```

## Best Practices

- Always check return values and handle errors explicitly.
- Use macros (e.g., `S_ISDIR`, `S_ISREG`, `S_ISLNK`) to accurately interpret file type information.
- Clearly differentiate when to use `stat`, `lstat`, or `fstat` based on whether you want to follow symbolic links.
- Manage symlinks cautiously, keeping in mind their impact on file operations and security.

---

# Descriptor Duplication and Pipes (unistd.h)

## Overview

### `dup, dup2, pipe`

These functions allow processes to duplicate file descriptors and create inter-process communication channels (pipes). They are crucial in shells and programs that redirect input/output streams or facilitate communication between processes.


## dup

**Header:** `#include <unistd.h>`

**Description:**  
Creates a copy of an existing file descriptor.

**Detailed:**
- The new file descriptor references the same open file/resource as the original.
- Returns the lowest-numbered available file descriptor.
- Both descriptors (the original and the new one) share file offsets and status flags.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(void) {
    int fd = open("output.txt", O_WRONLY | O_CREAT, 0644);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }

    int new_fd = dup(fd);
    if (new_fd == -1) {
        perror("dup failed");
        close(fd);
        return 1;
    }

    write(fd, "Hello", 5);      // Writes "Hello"
    write(new_fd, " World", 6); // Continues writing " World"

    close(fd);
    close(new_fd);
    return 0;
}
```

**Common Mistakes:**
- Not checking return values, which can cause unexpected errors.
- Forgetting to close duplicated descriptors, leading to resource leaks.


## dup2

**Header:** `#include <unistd.h>`

**Description:**  
Duplicates a file descriptor (`old_fd`) to a specified descriptor (`new_fd`).

**Detailed:**
- If `new_fd` is already open, it is automatically closed before duplication.
- Particularly useful for redirecting standard streams (stdin, stdout, stderr) or ensuring specific descriptor assignments.

**Proper Usage Example (redirecting stdout to a file):**

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>

int main(void) {
    int fd = open("log.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("open failed");
        return 1;
    }

    if (dup2(fd, STDOUT_FILENO) == -1) {
        perror("dup2 failed");
        close(fd);
        return 1;
    }

    // Now printf output goes to "log.txt"
    printf("This will be written to log.txt\n");

    close(fd); // STDOUT_FILENO remains open independently
    return 0;
}
```

**Common Mistakes:**
- Not handling errors properly, especially since `dup2` implicitly closes `new_fd`.
- Forgetting that `dup2` closes the target descriptor if it was previously open, which may lead to unintended closures.


## Differences Between dup and dup2

| Behavior / Feature       | dup                                     | dup2                                                 |
|--------------------------|-----------------------------------------|------------------------------------------------------|
| **Descriptor Chosen**    | Lowest-numbered available descriptor    | User-specified descriptor (`new_fd`)                 |
| **If Descriptor Open**   | Not applicable (always picks unused one) | Closes `new_fd` if already open before duplication   |
| **Usage Scenario**       | General duplication, descriptor backup  | Specific descriptor assignment, redirection          |

- **Summary:**
  - Use `dup` for simple descriptor duplication without specific number requirements.
  - Use `dup2` to precisely redirect or assign descriptors, as needed in shells or for I/O redirection.


## pipe

**Header:** `#include <unistd.h>`

**Description:**  
Creates a unidirectional communication channel between processes (a pipe).

**Detailed:**
- Provides two file descriptors: one for reading (`pipefd[0]`) and one for writing (`pipefd[1]`).
- Commonly used in IPC (inter-process communication) and shell pipelines.

**Proper Usage Example (Parent-child communication):**

```c
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(void) {
    int pipefd[2];
    char buffer[20];

    if (pipe(pipefd) == -1) {
        perror("pipe failed");
        return 1;
    }

    pid_t pid = fork();
    if (pid == -1) {
        perror("fork failed");
        return 1;
    }

    if (pid == 0) {
        // Child process: writes to pipe
        close(pipefd[0]); // Close unused read end
        const char *msg = "Hello from child";
        write(pipefd[1], msg, strlen(msg));
        close(pipefd[1]);
        _exit(0);
    } else {
        // Parent process: reads from pipe
        close(pipefd[1]); // Close unused write end
        ssize_t bytes_read = read(pipefd[0], buffer, sizeof(buffer) - 1);
        if (bytes_read > 0) {
            buffer[bytes_read] = '\0';
            printf("Parent received: %s\n", buffer);
        }
        close(pipefd[0]);
    }

    return 0;
}
```

**Common Mistakes:**
- Forgetting to close unused pipe ends, which can cause deadlocks or unintended behavior.
- Not properly handling return values, risking data integrity and resource misuse.


## Best Practices for dup, dup2, and pipe

- Always explicitly handle errors returned by these functions.
- Close unused file descriptors immediately to avoid leaks or deadlocks.
- Clearly differentiate between situations requiring `dup` (simple duplication) and `dup2` (explicit descriptor management).
- Carefully manage descriptor lifetimes when using pipes to prevent resource exhaustion and ensure proper inter-process communication.

---

# Directory Functions (`dirent.h`)

### Overview

**Directory streams** allow reading the contents of a directory entry by entry.

### `opendir, readdir, closedir`

The functions from `<dirent.h>` allow programs to read and manipulate directory entries. These functions provide an interface to explore directories entry-by-entry, typically used to list directory contents or perform operations on files/subdirectories.

## opendir

**Header:** `#include <dirent.h>`

**Description:**  
Opens a directory stream corresponding to the directory path provided.

**Detailed:**
- Returns a pointer to a `DIR` structure upon successful opening.
- Returns `NULL` on error (e.g., if the directory does not exist or there are permission issues) and sets `errno`.

**Proper Usage Example:**

```c
#include <dirent.h>
#include <stdio.h>

int main(void) {
    DIR *dir = opendir(".");
    if (dir == NULL) {
        perror("opendir failed");
        return 1;
    }

    printf("Directory opened successfully.\n");
    closedir(dir);
    return 0;
}
```

**Common Mistakes:**
- Not checking the return value, which can potentially cause segmentation faults when using a `NULL` pointer.
- Forgetting to close the directory stream, causing resource leaks.


## readdir

**Header:** `#include <dirent.h>`

**Description:**  
Reads the next entry from an opened directory stream (`DIR*`).

**Detailed:**
- Returns a pointer to a `struct dirent`, containing information like the file name (`d_name`) and inode number (`d_ino`).
- Returns `NULL` either at the end of the directory stream or on error. Use `errno` to distinguish end-of-directory from errors.

**Proper Usage Example (listing all entries in a directory):**

```c
#include <dirent.h>
#include <stdio.h>

int main(void) {
    DIR *dir = opendir(".");
    if (dir == NULL) {
        perror("opendir failed");
        return 1;
    }

    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL) {
        printf("Found entry: %s\n", entry->d_name);
    }

    closedir(dir);
    return 0;
}
```

**Common Mistakes:**
- Not differentiating between end-of-directory and an error condition (properly check `errno` if `readdir` returns `NULL` unexpectedly).
- Assuming entries are sorted (they typically are not).


## closedir

**Header:** `#include <dirent.h>`

**Description:**  
Closes a directory stream previously opened by `opendir`.

**Detailed:**
- Frees the resources allocated to the `DIR*` structure.
- Returns `0` on success, or `-1` on error, and sets `errno`.

**Proper Usage Example:**

```c
#include <dirent.h>
#include <stdio.h>

int main(void) {
    DIR *dir = opendir(".");
    if (dir == NULL) {
        perror("opendir failed");
        return 1;
    }

    if (closedir(dir) == -1) {
        perror("closedir failed");
        return 1;
    }

    printf("Directory stream closed successfully.\n");
    return 0;
}
```

**Common Mistakes:**
- Not handling potential errors from `closedir`.
- Forgetting to call `closedir`, which leads to resource leaks.


## What is the "next entry"?

- The "next entry" is the next available `struct dirent` returned by the function `readdir` each time it is called.
- Each call to `readdir` advances the read position in the directory stream, allowing iteration through the directory contents.

### Structure: `struct dirent`

```c
struct dirent {
    ino_t          d_ino;       // inode number
    off_t          d_off;       // offset to the next dirent
    unsigned short d_reclen;    // length of this record
    unsigned char  d_type;      // type of file (e.g., regular file, directory)
    char           d_name[256]; // filename (null-terminated)
};
```

## Best Practices

- Always handle and verify return values explicitly.
- Clearly differentiate between errors and end-of-directory conditions by inspecting `errno` after a `NULL` return from `readdir`.
- Close directory streams promptly after finishing operations to prevent resource leaks.

---


# Terminal & Device Functions (unistd.h, sys/ioctl.h)

## Overview

### `isatty, ttyname, ttyslot, ioctl`

Terminal and device functions facilitate interactions with terminal devices, enabling programs to detect, query, and manage terminal-specific characteristics. These functions are essential in programs like shells or command-line tools.


## isatty

**Header:** `#include <unistd.h>`

**Description:**  
Determines if a file descriptor refers to a terminal device.

**Detailed:**
- Returns `1` (true) if the file descriptor is connected to a terminal; otherwise returns `0`.
- Often used to check if standard input/output/error is from a terminal or redirected.

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    if (isatty(STDOUT_FILENO))
        printf("STDOUT is connected to a terminal.\n");
    else
        printf("STDOUT is not connected to a terminal.\n");

    return 0;
}
```

**Common Mistakes:**
- Misinterpreting the return value; always explicitly check for `1` (terminal) or `0` (non-terminal).


## ttyname

**Header:** `#include <unistd.h>`

**Description:**  
Returns the pathname of the terminal associated with a file descriptor (e.g., `/dev/pts/0`).

**Detailed:**
- Useful for identifying the terminal device being used.
- Returns `NULL` if the descriptor is not associated with a terminal or an error occurs (check `errno`).

**Proper Usage Example:**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    char *name = ttyname(STDIN_FILENO);
    if (name)
        printf("Terminal name: %s\n", name);
    else
        perror("ttyname failed");

    return 0;
}
```

**Common Mistakes:**
- Not checking for `NULL`, which can lead to segmentation faults if the descriptor isn't associated with a terminal.


## ttyslot

**Header:** `#include <unistd.h>`

**Description:**  
Retrieves the slot number of the terminal as listed in the system's terminal database (historically `/etc/ttys` or `/etc/utmp`).

**Detailed:**
- Considered a legacy feature; rarely used in modern software.
- Returns a positive slot number or `0` if not available.

**Proper Usage Example (Legacy):**

```c
#include <unistd.h>
#include <stdio.h>

int main(void) {
    int slot = ttyslot();
    if (slot > 0)
        printf("Terminal slot number: %d\n", slot);
    else
        printf("No terminal slot assigned.\n");

    return 0;
}
```

**Common Mistakes:**
- Relying on `ttyslot` in modern systems—this function is mostly obsolete.


## ioctl

**Header:** `#include <sys/ioctl.h>`

**Description:**  
Performs device-specific input/output operations.

**Detailed:**
- Allows interaction with device drivers to retrieve or set parameters directly.
- Frequently used with terminals for tasks like fetching terminal size (`TIOCGWINSZ`) or configuring terminal modes.

**Proper Usage Example (Getting Terminal Size):**

```c
#include <sys/ioctl.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    struct winsize ws;

    if (ioctl(STDOUT_FILENO, TIOCGWINSZ, &ws) == -1) {
        perror("ioctl failed");
        return 1;
    }

    printf("Terminal size: %d rows, %d columns\n", ws.ws_row, ws.ws_col);
    return 0;
}
```

**Common Mistakes:**
- Forgetting to include `<sys/ioctl.h>` for necessary definitions.
- Not handling the return value and errors explicitly, which could lead to undefined behavior.


## What is a Terminal Device?

Originally, terminal devices referred to physical devices (e.g., hardware terminals directly connected to a computer system). Today, most terminal interactions occur through pseudo-terminal devices (PTY), such as those in `/dev/pts/*`. These pseudo-terminals emulate physical terminals, allowing virtual terminal sessions (like SSH or terminal emulators) to communicate with processes.

**Example Paths:**
- **Physical terminal example (rare today):** `/dev/tty1`
- **Pseudo-terminal example (common):** `/dev/pts/0`, `/dev/pts/1`, etc.


## Best Practices

- Always explicitly handle function return values and errors.
- Be cautious when relying on legacy functions (`ttyslot`) as they might behave inconsistently across different systems.
- Clearly document terminal-specific logic in your code to maintain clarity and portability.


# Terminal Attribute Functions (termios.h)

## Overview

The functions `tcgetattr` and `tcsetattr` manage terminal attributes, allowing configuration of behaviors such as canonical mode (line-by-line input), echoing input characters, and other terminal characteristics. They are essential for customizing terminal interactions, particularly in shells and interactive programs.

## tcgetattr

**Header:** `#include <termios.h>`

**Description:**  
Retrieves the current attributes of the terminal associated with the provided file descriptor (`fd`).

**Detailed:**
- Populates a `struct termios` with the current terminal settings.
- Typically used to capture current settings before making modifications so that they can later be restored.

**Proper Usage Example:**

```c
#include <termios.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    struct termios old_settings;

    if (tcgetattr(STDIN_FILENO, &old_settings) == -1) {
        perror("tcgetattr failed");
        return 1;
    }

    printf("Successfully retrieved terminal attributes.\n");
    return 0;
}
```

**Common Mistakes:**
- Not checking the return value, which may lead to manipulating uninitialized terminal settings.

## tcsetattr

**Header:** `#include <termios.h>`

**Description:**  
Sets terminal attributes for the terminal associated with the provided file descriptor (`fd`).

**Detailed:**
- Accepts flags to specify when the attributes take effect:
  - **TCSANOW:** Changes occur immediately.
  - **TCSADRAIN:** Changes occur after all output written to `fd` has been transmitted.
  - **TCSAFLUSH:** Changes occur after flushing input and output queues.
- Frequently used to modify settings such as disabling canonical mode (`ICANON`) or turning off character echo (`ECHO`).

**Proper Usage Example (Disabling Canonical Mode and Echo):**

```c
#include <termios.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    struct termios oldt, newt;

    // Get current terminal attributes
    if (tcgetattr(STDIN_FILENO, &oldt) == -1) {
        perror("tcgetattr failed");
        return 1;
    }

    newt = oldt;

    // Disable canonical mode (line buffering) and echo
    newt.c_lflag &= ~(ICANON | ECHO);

    // Set modified attributes immediately
    if (tcsetattr(STDIN_FILENO, TCSANOW, &newt) == -1) {
        perror("tcsetattr failed");
        return 1;
    }

    printf("Canonical mode and echo are now OFF.\n");

    // Restore original attributes before exiting
    if (tcsetattr(STDIN_FILENO, TCSANOW, &oldt) == -1) {
        perror("tcsetattr restoration failed");
        return 1;
    }

    return 0;
}
```

**Common Mistakes:**
- Forgetting to restore original settings, which leaves the terminal in an unusable or unexpected state after program termination.
- Not handling return values and errors explicitly, resulting in subtle bugs or unstable terminal behavior.

## Understanding Canonical Mode and Echo

### Canonical Mode (`ICANON`):
- **Enabled (default):**  
  Input is processed line-by-line. The terminal buffers input until a newline (`\n`) or EOF character is received.
- **Disabled:**  
  Input is processed immediately without waiting for a newline, allowing programs to read input character-by-character. This mode is useful in interactive applications or text editors.

### Echo (`ECHO`):
- **Enabled (default):**  
  Characters typed by the user are displayed (echoed) back to the terminal.
- **Disabled:**  
  Characters typed are not displayed, which is useful for sensitive inputs such as passwords.

## Best Practices
- **Save Current Settings:**  
  Always save the current terminal attributes (using `tcgetattr`) before making changes so they can be restored later with `tcsetattr`.
- **Error Handling:**  
  Explicitly handle errors and return values to maintain terminal stability and ensure a consistent user experience.
- **Document Changes:**  
  Clearly document any changes to terminal behavior within your code, especially when disabling canonical mode or echo, to avoid confusing users.


## Terminal Capability Functions (termcap.h)

### Overview

The termcap library provides a way to handle different terminal capabilities consistently across various terminal types. It allows programs to query and utilize specific capabilities such as cursor movement, screen clearing, or terminal dimensions. This approach is essential for writing portable terminal applications.

## tgetent

**Header:** `#include <termcap.h>`

**Description:**  
Loads terminal capability entries for a given terminal type (usually defined by the TERM environment variable).

**Detailed:**
- Reads data from the termcap (or terminfo) database.
- **Returns:**
  - `1` if successful,
  - `0` if the terminal type is not found,
  - `-1` if the termcap database cannot be accessed.

**Proper Usage Example:**

```c
#include <termcap.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char *term_type = getenv("TERM");
    char term_buffer[2048];

    if (!term_type) {
        fprintf(stderr, "TERM environment variable not set.\n");
        return 1;
    }

    int success = tgetent(term_buffer, term_type);
    if (success < 0) {
        fprintf(stderr, "Could not access termcap database.\n");
        return 1;
    } else if (success == 0) {
        fprintf(stderr, "Terminal type '%s' not found.\n", term_type);
        return 1;
    }

    printf("Terminal type '%s' capabilities loaded successfully.\n", term_type);
    return 0;
}
```

## tgetflag

**Header:** `#include <termcap.h>`

**Description:**  
Queries boolean terminal capabilities (e.g., auto-margin capability).

**Detailed:**
- **Returns:**
  - `1` if the capability is available,
  - `0` otherwise.

**Proper Usage Example:**

```c
#include <termcap.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char term_buffer[2048];
    char *term_type = getenv("TERM");
    if (!term_type) {
        fprintf(stderr, "TERM environment variable not set.\n");
        return 1;
    }
    if (tgetent(term_buffer, term_type) != 1) {
        fprintf(stderr, "tgetent failed.\n");
        return 1;
    }

    int auto_margin = tgetflag("am");
    if (auto_margin)
        printf("Terminal supports automatic margins.\n");
    else
        printf("Terminal does NOT support automatic margins.\n");

    return 0;
}
```

## tgetnum

**Header:** `#include <termcap.h>`

**Description:**  
Retrieves numeric terminal capabilities (e.g., number of columns or lines).

**Detailed:**
- **Returns:**
  - The numeric value if the capability is found.
  - `-1` if the capability is unavailable.

**Proper Usage Example:**

```c
#include <termcap.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char term_buffer[2048];
    char *term_type = getenv("TERM");
    if (!term_type) {
        fprintf(stderr, "TERM environment variable not set.\n");
        return 1;
    }
    if (tgetent(term_buffer, term_type) != 1) {
        fprintf(stderr, "tgetent failed.\n");
        return 1;
    }

    int cols = tgetnum("co");
    int rows = tgetnum("li");

    if (cols != -1 && rows != -1)
        printf("Terminal size: %d columns, %d rows\n", cols, rows);
    else
        printf("Terminal dimensions not available.\n");

    return 0;
}
```

## tgetstr

**Header:** `#include <termcap.h>`

**Description:**  
Retrieves string-type terminal capabilities (e.g., commands to clear the screen or move the cursor).

**Detailed:**
- **Returns:**  
  A pointer to the capability string, or `NULL` if the capability is unavailable.  
  Capability strings often include control sequences for terminal operations.

**Proper Usage Example (Clear screen):**

```c
#include <termcap.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char term_buffer[2048], area[2048], *ap = area;
    char *term_type = getenv("TERM");
    if (!term_type) {
        fprintf(stderr, "TERM environment variable not set.\n");
        return 1;
    }
    if (tgetent(term_buffer, term_type) != 1) {
        fprintf(stderr, "tgetent failed.\n");
        return 1;
    }

    char *clear = tgetstr("cl", &ap);
    if (clear)
        tputs(clear, 1, putchar);
    else
        printf("Clear capability not available.\n");

    return 0;
}
```

## tgoto

**Header:** `#include <termcap.h>`

**Description:**  
Constructs a cursor-positioning command from a given capability string.

**Detailed:**
- Used for cursor movement operations (e.g., moving the cursor to a specific row/column).
- Typically paired with the "cm" (cursor motion) capability.

**Proper Usage Example (Move cursor):**

```c
#include <termcap.h>
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char term_buffer[2048], area[2048], *ap = area;
    char *term_type = getenv("TERM");
    if (!term_type) {
        fprintf(stderr, "TERM environment variable not set.\n");
        return 1;
    }
    if (tgetent(term_buffer, term_type) != 1) {
        fprintf(stderr, "tgetent failed.\n");
        return 1;
    }

    char *cursor_move = tgetstr("cm", &ap);
    if (!cursor_move) {
        printf("Cursor movement capability unavailable.\n");
        return 1;
    }

    // Move cursor to row 10, column 20
    char *cmd = tgoto(cursor_move, 20, 10);
    tputs(cmd, 1, putchar);

    printf("Cursor moved to row 10, column 20.\n");
    return 0;
}
```

## tputs

**Header:** `#include <termcap.h>`

**Description:**  
Outputs terminal control strings with appropriate padding.

**Detailed:**
- Handles delays and padding as required by certain terminals.
- Commonly used to execute control sequences retrieved via `tgetstr`.

**Proper Usage Example (Clear screen example from above):**

```c
#include <termcap.h>
#include <stdio.h>
#include <stdlib.h>

int putchar_wrapper(int c) {
    return putchar(c);
}

int main(void) {
    char term_buffer[2048], area[2048], *ap = area;
    char *term_type = getenv("TERM");
    if (!term_type) {
        fprintf(stderr, "TERM environment variable not set.\n");
        return 1;
    }
    if (tgetent(term_buffer, term_type) != 1) {
        fprintf(stderr, "tgetent failed.\n");
        return 1;
    }

    char *clear = tgetstr("cl", &ap);
    if (clear)
        tputs(clear, 1, putchar_wrapper);
    else
        printf("Clear capability not available.\n");

    return 0;
}
```

## What is Termcap?

Termcap stands for "terminal capability." It is a database (traditionally located at `/etc/termcap`, or using terminfo in modern systems) that describes various terminal behaviors and capabilities, including:

- **Cursor Movements:**  
  Commands to move the cursor around the screen.
- **Screen Manipulation:**  
  Functions to clear the screen, scroll, etc.
- **Keyboard Inputs and Control Sequences:**  
  Definitions for how the terminal handles specific key presses.
  
Libraries such as ncurses or the native termcap library utilize this database to create portable, terminal-independent applications.

## Best Practices
- **Verify Capabilities:**  
  Always verify that a terminal capability is present before using it.
- **Error Handling:**  
  Explicitly handle all error conditions and return values from termcap library functions.
- **Modern Alternatives:**  
  For new development, consider using modern alternatives like ncurses, as direct termcap usage can be complex and error-prone.

---

## Error Handling Functions (`errno.h`, `stdio.h`, `string.h`)

### Overview

Error handling in C commonly relies on the global integer `errno`, set by library functions and system calls to indicate specific errors. Functions like `strerror` and `perror` provide convenient ways to obtain human-readable error messages related to these error codes.

### Functions Explained

#### `strerror`

- **Header**: `#include <string.h>`
- **Description**: Converts an error number (typically from `errno`) into a descriptive, human-readable error message.
- **Detailed**:
  - Returns a pointer to a statically allocated string describing the error.
  - The returned string should **not** be modified or freed by the caller.
  - Useful when error messages need to be integrated within custom-formatted error reporting.

**Proper Usage Example**:
```c
#include <string.h>
#include <errno.h>
#include <stdio.h>
#include <fcntl.h>

int main(void) {
    int fd = open("nonexistent_file.txt", O_RDONLY);
    if (fd == -1) {
        printf("Failed to open file: %s\n", strerror(errno));
    }
    return 0;
}
```

**Common Mistakes**:
- Attempting to free or modify the returned pointer (can lead to undefined behavior).
- Forgetting to include `<errno.h>` to access the global `errno` variable.

#### `perror`

- **Header**: `#include <stdio.h>`
- **Description**: Prints a user-provided message followed by the error message associated with the current value of `errno`.
- **Detailed**:
  - Combines a descriptive message (`msg`) with `strerror(errno)` to provide clear error context.
  - Typically called immediately after a function or system call failure.
  - Writes output to standard error (`stderr`).

**Proper Usage Example**:
```c
#include <stdio.h>

int main(void) {
    FILE *fp = fopen("nonexistent.txt", "r");
    if (fp == NULL) {
        perror("fopen failed");
        return 1;
    }

    fclose(fp);
    return 0;
}
```

**Example Output**:
```
fopen failed: No such file or directory
```

**Common Mistakes**:
- Calling `perror` without immediately checking the failed function, risking `errno` changing due to subsequent calls.
- Not providing a meaningful message, thus reducing the clarity of error diagnostics.

#### Understanding `errno`

- **Header**: `#include <errno.h>`
- `errno` is a thread-local global integer automatically set by many system calls and library functions when an error occurs.
- Never manually set `errno` to `0` before a call; however, setting it to `0` explicitly before some library calls is a valid way to differentiate certain errors.

**Example usage (checking errno explicitly)**:
```c
#include <errno.h>
#include <stdio.h>
#include <fcntl.h>

int main(void) {
    errno = 0; // explicitly clear errno
    int fd = open("missing.txt", O_RDONLY);

    if (fd == -1) {
        if (errno == ENOENT)
            printf("File does not exist.\n");
        else
            perror("open failed");
    }

    return 0;
}
```

### **Best Practices**:
- Always check the return values of system calls or library functions immediately.
- Use `perror` for quick, straightforward error reporting directly to `stderr`.
- Use `strerror` when custom formatting or logging of error messages is required.
- Remember that error strings returned by `strerror` must not be freed or altered directly.
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
