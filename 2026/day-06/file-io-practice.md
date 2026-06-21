## Step 1 — Create the file

```bash
$ touch notes.txt
```
**What it did:** Created an empty `notes.txt` file. `touch` doesn't write anything — it just creates the file if it doesn't exist (or updates its timestamp if it does).

---

## Step 2 — Write the first line (overwrite mode)

```bash
$ echo "Line 1" > notes.txt
```
**What it did:** `>` redirects output into the file and **overwrites** whatever was there before. Since the file was empty, this just wrote `Line 1`. ⚠️ Important: using `>` on a file with existing content wipes it.

---

## Step 3 — Append more lines (append mode)

```bash
$ echo "Line 2" >> notes.txt
$ echo "Line 4" >> notes.txt
$ echo "Line 5" >> notes.txt
```
**What it did:** `>>` appends to the end of the file instead of overwriting. This is the safe redirection operator for adding content without losing what's already there.

---

## Step 4 — Write and display at the same time with `tee`

```bash
$ echo "Line 3" | tee -a notes.txt
Line 3
```
**What it did:** `tee -a` writes the input to the file (append mode, because of `-a`) **and** prints it to the terminal at the same time. Without `-a`, `tee` would overwrite the file instead of appending. This is useful when running a command and you want to both save the output to a log and watch it live.

```bash
$ echo "Line 6" | tee -a notes.txt > /dev/null
```
**What it did:** Same as above, but I redirected the terminal output to `/dev/null` to suppress the screen print — useful when you only want the file write, not the display, e.g., inside a script.

---

## Step 5 — Read the full file with `cat`

```bash
$ cat notes.txt
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
```
**What it did:** `cat` dumps the entire file to the terminal in order. Good for small files; not ideal for huge logs (use `less` or `tail -f` instead for those).

---

## Step 6 — Read the top of the file with `head`

```bash
$ head -n 2 notes.txt
Line 1
Line 2
```
**What it did:** Shows only the first 2 lines. Useful for quickly checking a file's header or the start of a config without scrolling through the whole thing.

---

## Step 7 — Read the bottom of the file with `tail`

```bash
$ tail -n 2 notes.txt
Line 5
Line 6
```
**What it did:** Shows only the last 2 lines — exactly what I'd reach for to check the most recent entries in a log file.

---

## Bonus — Verify the file

```bash
$ wc -l notes.txt
6 notes.txt

$ ls -l notes.txt
-rw-r--r-- 1 root root 42 Jun 21 08:25 notes.txt
```
**What it did:** `wc -l` counts lines (confirms I have 6 lines, matching what I wrote). `ls -l` confirms the file exists, its size (42 bytes), and permissions (`rw-r--r--` — owner can read/write, everyone else read-only).

---

## Final File Contents

```
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
```

---

## Key Takeaways

| Command | When I'll Use It |
|---------|-------------------|
| `>` | Only when I deliberately want to overwrite/reset a file |
| `>>` | The safe default for adding to logs, configs, scripts |
| `tee -a` | When I need to log AND watch output live (e.g., during a deploy script) |
| `cat` | Quick full read of small files |
| `head -n` | Peek at the start — headers, top of configs |
| `tail -n` | Peek at the end — most recent log entries |

---
