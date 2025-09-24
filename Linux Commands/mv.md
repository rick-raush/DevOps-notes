&nbsp;

## 🔄 What Happens When You Run `mv`?

```bash
mv oldname.yaml newname.yaml
```

You might think this:

> "It copies the file to a new name and then deletes the old one."

But actually, **that’s not how it works — especially on the same filesystem**.

* * *

## ⚙️ Here's What Really Happens (on same filesystem):

### ✅ `mv` is a **rename operation**, not copy + delete.

- The file stays **in the same place** on disk.
    
- Only the **filename (metadata in the directory)** is updated.
    
- The file’s **contents don’t move**.
    
- This makes the operation **very fast and atomic**.
    

* * *

## 🧠 What Does “Atomic” Mean?

> An **atomic** operation is one that either **completes fully** or **doesn’t happen at all** — there’s no in-between.

For `mv`:

- If the rename succeeds, it’s done.
    
- If it fails, the original file is untouched.
    
- There’s no partial or corrupt file.
    

* * *

## 📦 Why Is It So Fast?

Because the OS doesn't touch the file’s data, only its **entry in the directory structure (inode in Linux)**.

Think of it like renaming a contact in your phone:

- You’re not copying all the contact’s data.
    
- You’re just editing the name field.
    

* * *

## ❗ Exception: Moving Across Filesystems

If you do something like:

```bash
mv /mnt/drive1/file.yaml /mnt/drive2/file.yaml
```

This **is** a copy + delete because you're moving between two different filesystems.

You can check this with:

```bash
df -T
```

If the source and destination have different filesystems (e.g., `ext4`, `xfs`), it won't be atomic or fast.

* * *

## ✅ Summary

| Scenario | What `mv` does | Speed | Atomic |
| --- | --- | --- | --- |
| Same filesystem | Just updates the directory entry | Fast | Yes |
| Different filesystems | Copies file, then deletes original | Slower | No  |

* * *

[Introduction to Linux.pdf](../_resources/Introduction%20to%20Linux.pdf)

&nbsp;