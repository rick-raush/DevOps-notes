t’s a lot to take in because we just went from **log files** to **kernel internals**. Let’s thread the needle and connect all these concepts into one clear picture.

**The Big Picture: How it all connects**

Imagine your Linux server is a **High-Security Bank**:

1.  **`/var/log/secure`**: This is the **Security Guard’s Logbook** at the front door. It notes who walked in (Login), who left, and who tried to use a key they weren't supposed to have (`sudo`).
2.  **`/var/log/audit/audit.log`**: This is the **CCTV System** inside the vault. It records every tiny movement—who touched which box, who opened a drawer, and what tools they used.
3.  **System Calls (Syscalls)**: These are the **Physical Actions** the people inside the bank take (e.g., "Open Drawer," "Pick up Pen"). In Linux, a program can't do anything without asking the Kernel via a syscall.
4.  **`strace`**: This is your **X-Ray Specs**. It allows you to watch those "Physical Actions" (syscalls) happening in real-time.
5.  **IPC (Inter-Process Communication)**: This is **Radio Chatter** between two different employees. They use syscalls (the actions) to send messages to each other.

* * *

**1\. The Two Log Files (Where data is stored)**

- **Use `/var/log/secure`** when you want to know: *"Did 'John' successfully log in via SSH at 2 PM?"* or *"Who tried to use `sudo` and failed?"*
- **Use `/var/log/audit/audit.log`** when you want to know: *"Which process modified the `/etc/shadow` file?"* It records the **syscalls** that modified the file.

**2\. System Calls (What the computer is actually doing)**

Every time you saw a line like `openat(...)` or `mmap(...)` in your prompt, you were looking at a **Syscall**.

- Programs live in **User Mode** (low privilege).
- The Hardware/Files live in **Kernel Mode** (high privilege).
- The **Syscall** is the bridge. The program says "I want to read this file," the Kernel checks if it's allowed, and then does it.

**3\. IPC vs. Syscalls (How programs talk)**

- **Syscall** is the **Mechanism**: `write()`, `read()`, `mmap()`.
- **IPC** is the **Goal**: "Process A needs to send a secret code to Process B."
- To achieve **IPC**, the processes use **Syscalls** to create a "Shared Memory" space or a "Message Queue."

**4\. Understanding your `strace` output**

The text you pasted (`openat`, `mmap`, `mprotect`) is the "startup routine" of almost every Linux program:

1.  **`openat`**: "Find the library file I need to run."
2.  **`read`**: "Check if the file is valid."
3.  **`mmap`**: "Map that file into my memory so I can use its code."
4.  **`mprotect`**: "Lock the memory so nobody can hack/change my code while I'm running."
5.  **`close`**: "I'm done with the file on disk; I have it in my memory now."

* * *

**Summary Table**

| Concept | What is it? | Why do you care? |
| --- | --- | --- |
| **`/var/log/secure`** | Auth Log | Tracking **who** accessed the system. |
| **`audit.log`** | Forensic Log | Tracking **what** the system did (detailed). |
| **Syscall** | Kernel Request | The **only way** a program interacts with hardware. |
| **IPC** | Process Talk | How apps **share data** with each other. |
| **`strace`** | Debug Tool | To see **syscalls** as they happen. |
| **`ipcs`** | IPC Tool | To see **shared memory** currently in use. |