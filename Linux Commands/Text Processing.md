📝 Text Processing Tools in Linux: Detailed Explanation

* * *

## 1\. **cut**

### What is `cut`?

- `cut` is a simple command-line tool to **extract sections from each line of a file or input**, based on:
    
    - **Delimited fields** (e.g., fields separated by commas, colons, spaces)
        
    - **Character positions**
        

### When to use it?

- You want to quickly get certain **columns or fields** from structured text.
    
- When the file format is consistent and simple.
    

### How it works

- You specify:
    
    - The **delimiter** (`-d`), like a comma `,` or colon `:`
        
    - The **field number(s)** to extract (`-f`), e.g., `-f1,3,5`
        
    - Or character ranges (`-c`), like `-c1-5` for first 5 characters
        

* * *

### Basic Syntax

```bash
cut -d '<delimiter>' -f <field_numbers> filename
```

OR

```bash
cut -c <character_ranges> filename
```

* * *

### Examples

1.  Extract the first field from `/etc/passwd` (colon-separated):

```bash
cut -d ':' -f 1 /etc/passwd
```

2.  Extract the first 10 characters of each line:

```bash
cut -c 1-10 filename.txt
```

3.  Extract multiple fields (1 and 3):

```bash
cut -d ',' -f 1,3 file.csv
```

* * *

### Limitations of `cut`

- Only works with **single-character delimiters**.
    
- Cannot handle **complex patterns** or variable whitespace.
    
- No support for conditions or calculations.
    
- Fields must be consistent across all lines.
    

* * *

## 2\. **awk**

### What is `awk`?

- `awk` is a **powerful pattern scanning and processing language**.
    
- It treats input as records (lines) and fields (columns).
    
- Supports **conditions**, **loops**, **calculations**, and **string processing**.
    
- Basically, it’s a tiny programming language specialized for text.
    

### When to use it?

- When you need to:
    
    - Extract and filter data conditionally
        
    - Perform calculations (sum, average)
        
    - Format output nicely
        
    - Parse files with complex delimiters or irregular spacing
        

* * *

### Basic Syntax

```bash
awk -F '<delimiter>' 'pattern { action }' filename
```

- `-F`: Sets the field delimiter (default is whitespace)
    
- `pattern`: Optional condition to match lines
    
- `{ action }`: What to do on matching lines, e.g., `print $1`
    

* * *

### Common Variables in `awk`

| Variable | Description |
| --- | --- |
| `$0` | Entire current line |
| `$1`, `$2`, ... | Individual fields (columns) |
| `NR` | Current record (line) number |
| `NF` | Number of fields in current line |

* * *

### Examples

1.  Print the first column of a file (default delimiter is whitespace):

```bash
awk '{ print $1 }' filename.txt
```

2.  Print users with UID greater than 1000 from `/etc/passwd`:

```bash
awk -F ':' '$3 > 1000 { print $1, $3 }' /etc/passwd
```

3.  Sum the values in the second column:

```bash
awk '{ sum += $2 } END { print sum }' file.txt
```

4.  Print lines where the third field equals "ERROR":

```bash
awk '$3 == "ERROR" { print $0 }' logfile.log
```

* * *

### Advanced Features

- String functions: `length()`, `substr()`, `index()`
    
- Arithmetic operations: `+`, `-`, `*`, `/`
    
- Control structures: `if`, `while`, `for`
    
- Arrays and variables
    
- Formatting output with `printf`
    

* * *

## 3\. Other Related Text Processing Tools (Overview)

| Command | Purpose | Notes |
| --- | --- | --- |
| `sed` | Stream editor, great for text substitution | Regex-based editing on streams |
| `grep` | Search text using patterns | Finds matching lines |
| `tr` | Translate or delete characters | Character-level transformations |
| `sort` | Sort lines of text | Alphabetical or numerical |
| `uniq` | Filter out duplicate lines | Usually used after `sort` |
| `head` | Show first lines | Useful for previews |
| `tail` | Show last lines | Follow logs, etc. |

* * *

## Summary Comparison: `cut` vs `awk`

| Feature | `cut` | `awk` |
| --- | --- | --- |
| Complexity | Simple | Powerful, programmable |
| Delimiters | Single-char only | Any regex, flexible delimiter |
| Conditions | No  | Yes |
| Calculations | No  | Yes |
| Output formatting | No  | Yes |
| Best for | Quick field extraction | Complex parsing & reporting |

* * *

## When to use what?

| Task | Tool |
| --- | --- |
| Extract fixed fields fast | `cut` |
| Extract fields with complex conditions | `awk` |
| Perform calculations or formatting | `awk` |
| Edit lines with substitution | `sed` |
| Search for patterns | `grep` |

* * *

&nbsp;

Absolutely! Here’s a detailed explanation of **`sed`** and **`grep`**, two other essential text processing tools in Linux. They often go hand-in-hand with `awk` and `cut` but serve different purposes.

* * *

# Detailed Explanation: `sed` and `grep`

* * *

## 1\. **grep**

* * *

### What is `grep`?

- `grep` stands for **Global Regular Expression Print**.
    
- It is used to **search text** for lines matching a specified **pattern** (usually a regular expression).
    
- It prints the matching lines by default.
    
- It’s one of the most widely used commands for text searching.
    

* * *

### When to use `grep`?

- To find lines containing a specific word, phrase, or pattern.
    
- To filter logs, config files, or command outputs.
    
- To count occurrences or find patterns inside files.
    

* * *

### Basic Syntax

```bash
grep [options] 'pattern' filename
```

OR to search command output:

```bash
command | grep 'pattern'
```

* * *

### Commonly used options

| Option | Meaning |
| --- | --- |
| `-i` | Ignore case (case-insensitive search) |
| `-v` | Invert match (show lines that **don’t** match) |
| `-r` | Recursive search in directories |
| `-n` | Show line numbers of matching lines |
| `-c` | Show count of matching lines only |
| `-w` | Match whole words only |
| `-E` | Use extended regex (same as `egrep`) |

* * *

### Examples

1.  Search for the word “error” in a log file (case insensitive):

```bash
grep -i 'error' /var/log/syslog
```

2.  Find lines that **do not** contain "root":

```bash
grep -v 'root' /etc/passwd
```

3.  Count the number of lines that contain "bash":

```bash
grep -c 'bash' /etc/passwd
```

4.  Search recursively for "TODO" in current directory:

```bash
grep -r 'TODO' .
```

5.  Show line numbers with matched lines:

```bash
grep -n 'failed' /var/log/auth.log
```

* * *

### How it works internally

- `grep` reads input line-by-line.
    
- For each line, it checks if the pattern matches.
    
- If yes, it prints the line (or other requested output).
    

* * *

## 2\. **sed**

* * *

### What is `sed`?

- `sed` stands for **Stream Editor**.
    
- It is used to **perform basic text transformations on an input stream (file or pipeline)**.
    
- Unlike editors like `vim` which are interactive, `sed` works **non-interactively**.
    
- Often used for **search and replace, insertion, deletion, and substitution** in text.
    

* * *

### When to use `sed`?

- Modify files or streams without opening a text editor.
    
- Batch substitution or deletion of lines.
    
- Scripting automated changes on text files.
    
- Extract or manipulate parts of lines using regex.
    

* * *

### Basic Syntax

```bash
sed [options] 'script' filename
```

Where `script` is a command or series of commands, often in single quotes.

* * *

### Common commands in `sed`

| Command | Description | Example |
| --- | --- | --- |
| `s/pattern/repl/` | Substitute pattern with replacement | `sed 's/foo/bar/' file.txt` |
| `d` | Delete matching lines | `sed '/^#/d' file.txt` (delete comment lines) |
| `p` | Print selected lines | `sed -n '/error/p' logfile` |
| `i\` | Insert line before current line | `sed '/pattern/i\New line' file.txt` |
| `a\` | Append line after current line | `sed '/pattern/a\New line' file.txt` |
| `q` | Quit after N lines | `sed '10q' file.txt` (stop after 10 lines) |

* * *

### Examples

1.  Replace first occurrence of “apple” with “orange” on each line:

```bash
sed 's/apple/orange/' file.txt
```

2.  Replace **all** occurrences on each line (`g` flag for global):

```bash
sed 's/apple/orange/g' file.txt
```

3.  Delete all blank lines:

```bash
sed '/^$/d' file.txt
```

4.  Delete lines containing the word “error”:

```bash
sed '/error/d' file.txt
```

5.  Print only lines containing “INFO”:

```bash
sed -n '/INFO/p' file.txt
```

6.  Insert a line before every line that contains “Start”:

```bash
sed '/Start/i\This is inserted line' file.txt
```

7.  Append a line after every line that contains “End”:

```bash
sed '/End/a\This is appended line' file.txt
```

* * *

### Editing files in-place

- `sed` can edit files directly with `-i` option (be careful!):

```bash
sed -i 's/foo/bar/g' filename
```

This replaces “foo” with “bar” **inside the file** without creating a backup (unless you specify one).

* * *

## Summary

| Command | Primary Use | Strengths | Typical Usage |
| --- | --- | --- | --- |
| `grep` | Search text lines by pattern | Fast, simple search, filtering | Find errors, logs, count matches |
| `sed` | Stream editing & substitution | Search/replace, delete, insert lines | Batch file edits, regex replacements |

* * *

## Quick comparison:

| Feature | `grep` | `sed` |
| --- | --- | --- |
| Function | Search and print matching lines | Edit/transform text in stream |
| Supports Regex | Yes | Yes |
| Can modify files? | No (outputs results only) | Yes (with `-i` option) |
| Typical use case | Filtering logs, searching files | Substitution, deleting lines, batch editing |

* * *

&nbsp;

* * *

# Vi Editor Quick Reference

### What is Vi?

- Powerful text editor used in Unix/Linux systems.
    
- Open a file: `vi filename`
    

* * *

### Modes

| Mode | Purpose | How to enter |
| --- | --- | --- |
| **Normal** | Navigation & commands (default mode) | Press `Esc` to switch here |
| **Insert** | Text editing | Press `i` (insert), `a` (append), or `o` (new line) |
| **Command** | Run commands like save/quit | Press `:` in Normal mode |

* * *

### Basic Navigation (Normal Mode)

| Key | Action |
| --- | --- |
| `h` | Move left |
| `l` | Move right |
| `j` | Move down |
| `k` | Move up |

* * *

### Editing (Insert Mode)

| Command | Action |
| --- | --- |
| `i` | Insert before cursor |
| `a` | Append after cursor |
| `o` | Open new line below |
| `Esc` | Return to Normal mode |

* * *

### Copy, Cut, Paste (Normal Mode)

| Command | Action |
| --- | --- |
| `yy` | Copy (yank) line |
| `dd` | Cut (delete) line |
| `p` | Paste after cursor |
| `P` | Paste before cursor |

* * *

### Searching

| Command | Action |
| --- | --- |
| `/word` | Search forward for "word" |
| `?word` | Search backward for "word" |
| `n` | Repeat last search forward |
| `N` | Repeat last search backward |

* * *

### Saving and Exiting (Command Mode, after pressing `:`)

| Command | Action |
| --- | --- |
| `:w` | Save file |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |

* * *

&nbsp;