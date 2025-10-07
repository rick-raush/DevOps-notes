&nbsp;

## 📁 **File Management - Linux Commands**

### 🧭 Navigation

| Command | Description |
| --- | --- |
| `pwd` | Show current working directory |
| `cd <dir>` | Change to directory `<dir>` |
| `cd ..` | Go up one level (parent directory) |

* * *

### 📄 File and Folder Creation

| Command | Description |
| --- | --- |
| `touch <file>` | Create an empty file |
| `mkdir <dir>` | Create a directory |
| `mkdir -p <path>` | Create nested directories (parents if needed) |
| `mkdir -v` | Show messages for created directories |
| `mkdir -m 755 <dir>` | Create directory with specific permissions |

* * *

### ❌ Deletion

| Command | Description |
| --- | --- |
| `rm <file>` | Delete a file |
| `rm -r <dir>` | Delete a directory and its contents (recursive) |
| `rm -f <file>` | Force delete (no confirmation) |
| `rm -i` | Prompt before deleting |
| `rm -v` | Show files as they are removed |
| `rmdir <dir>` | Remove empty directory |
| `rmdir -p` | Remove nested empty directories |

&nbsp;

* * *

## **1️⃣ Command: `find . \( -name ".terraform*" -o -name ".terragrunt*" \) -exec rm -rv {} +`**

### 🔹 Breakdown:

- `find .`
    
    - Starts searching in the current directory `.` and all subdirectories recursively.
- `\( -name ".terraform*" -o -name ".terragrunt*" \)`
    
    - `-name ".terraform*"` → matches any file or directory starting with `.terraform`
        
    - `-o` → logical OR
        
    - `-name ".terragrunt*"` → matches any file or directory starting with `.terragrunt`
        
    - The `\(` and `\)` are required to group the OR condition in `find`.
        
- `-exec rm -rv {} +`
    
    - For each match, execute `rm -rv` on it.
        
    - `{}` → placeholder for the matched file/directory
        
    - `+` → runs `rm` on multiple files at once (more efficient than `\;`)
        
    - `rm -rv` flags explained below
        

### 🔹 Flags for `rm -rv`:

| Flag | Meaning |
| --- | --- |
| `-r` | recursive — deletes directories and their contents |
| `-v` | verbose — prints each file/folder as it is removed |

* * *

### 🔹 Key point:

- `find` **traverses all subdirectories** automatically.
    
- It will match **both files and directories** anywhere in the directory tree.
    
- Very safe when you want recursive deletion based on patterns.
    

* * *

## **2️⃣ Command: `rm -rvf ".terraform*" ".terragrunt*"`**

### 🔹 Breakdown:

- `rm` → remove command
    
- Flags:
    
    - `-r` → recursive
        
    - `-v` → verbose
        
    - `-f` → force (ignore nonexistent files, don’t prompt for confirmation)
        
- `".terraform*" ".terragrunt*"` → shell globbing patterns, **expanded by the shell before `rm` runs**.
    

* * *

### 🔹 Behavior:

- Only matches files/directories in **the current directory**.
    
- Does **not traverse subdirectories**.
    
- If no matches are found, zsh (without `nullglob`) will throw an error.
    
- Quoting the pattern prevents the shell from interpreting the glob, but then `rm` itself does not expand globs — it will fail unless you enable glob expansion in `rm` (which it doesn’t by default).
    

* * *

## **3️⃣ Comparison**

| Feature | `rm -rvf ".terraform*" ".terragrunt*"` | `find . -name ... -exec rm -rv {} +` |
| --- | --- | --- |
| Recursive | Only deletes directories recursively if they are in current directory, not inside subdirs | True recursion through all subdirectories |
| Pattern match | Shell globbing, limited to current directory | Uses `find` with `-name` or regex; can search anywhere |
| Verbose | Yes (`-v`) | Yes (`rm -v`) |
| Force deletion | Yes (`-f`) | `rm` can be combined with `-f` if needed |
| Safety | Can fail silently or delete unintended files if glob misused | Safer — only deletes files that actually match pattern |
| Handles dotfiles in subdirs | No  | Yes |
| Works across all shells | Glob expansion can behave differently | Works reliably across bash, zsh, etc. |

* * *

### **✅ TL;DR**

- Use `rm -rvf` for **quick deletion in the current directory**.
    
- Use `find ... -exec rm -rv {} +` for **recursively deleting patterns in all subdirectories** — much safer and reliable.
    

&nbsp;

* * *

### 📋 Copy & Move

| Command | Description |
| --- | --- |
| `cp <src> <dest>` | Copy file |
| `cp -r <src_dir> <dest>` | Copy directory recursively |
| `cp -i` | Prompt before overwrite |
| `cp -u` | Copy only if source is newer |
| `cp -a` | Archive (preserves structure, links, permissions) |
| `mv <src> <dest>` | Move or rename file/directory |
| `mv -i` | Prompt before overwrite |
| `mv -f` | Force move (overwrite without prompt) |
| `mv -v` | Verbose (show moved files) |

* * *

### 🔎 Searching

| Command | Description |
| --- | --- |
| `find <path> -name "<pattern>"` | Find files by name |
| `find . -type f -size +10M` | Find files larger than 10MB |
| `find <path> -exec <cmd> {} \;` | Run command on found files |

* * *

### 🌳 Structure & Links

| Command | Description |
| --- | --- |
| `tree` | Show directory structure (needs install) |
| `tree -L 2` | Limit depth to 2 levels |
| `tree -a` | Include hidden files |
| `tree -d` | Show only directories |
| `tree -I "*.log"` | Ignore `.log` files |

* * *

### 🔗 Links

| Command | Description |
| --- | --- |
| `ln <file> <link>` | Create a hard link |
| `ln -s <target> <link>` | Create a symbolic (soft) link |
| `ln -f` | Force overwrite existing links |
| `ln -t <dir> <target>` | Link to target into given directory |

&nbsp;

* * *

## 🔗 **Links in Linux**

Linux filesystems allow **links**, which are basically shortcuts or references to files or directories. There are two main types:

### 1\. **Hard Links**

- A **hard link** is like an additional name for the same file.
    
- Both the original file and hard link point to the same data (inode) on disk.
    
- Deleting one hard link **does not delete the actual file data** as long as one link remains.
    
- Hard links **cannot span across different filesystems** (partitions).
    
- You **cannot create hard links to directories** (usually restricted to avoid loops).
    

**Example:**

```bash
ln file.txt file_hardlink.txt
```

- `file_hardlink.txt` is now another name for `file.txt`.
    
- Changes to either affect the same file content.
    

* * *

### 2\. **Symbolic (Soft) Links**

- A **symbolic link (symlink)** is like a shortcut or pointer to another file or directory.
    
- It stores the **path** to the target file or directory.
    
- If the target is deleted or moved, the symlink becomes broken (dangling link).
    
- Symlinks **can cross filesystem boundaries**.
    
- Symlinks **can point to directories**.
    

**Example:**

```bash
ln -s /path/to/original/file.txt shortcut.txt
```

- `shortcut.txt` points to `/path/to/original/file.txt`.
    
- Opening `shortcut.txt` opens the target file.
    

* * *

### Differences at a glance

| Feature | Hard Link | Symbolic Link |
| --- | --- | --- |
| Points to | Same inode (file data) | Path to file/directory |
| Can link directories? | No (usually restricted) | Yes |
| Can cross filesystems? | No  | Yes |
| Breaks if target deleted? | No (file data remains if at least one hard link exists) | Yes (dangling link) |
| Uses disk space? | No additional (just directory entry) | Small space for storing path |

* * *

### How to check links?

Use `ls -l` to see links:

```bash
ls -l file.txt file_hardlink.txt shortcut.txt
```

- Hard links have the same inode number (first column).
    
- Symbolic links show `->` followed by the target.
    

* * *

&nbsp;

&nbsp;

&nbsp;