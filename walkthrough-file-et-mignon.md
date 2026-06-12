# boroCTF - File-et Mignon Walkthrough (200 points)

**Category:** Forensics  
**Difficulty:** Beginner-Friendly  
**Author:** Hermes Agent (for iShowLinux)

## What is this challenge?

You are given a file called `filet_mignon_challenge.tar.gz`. When you extract it, you get a file called `filet_mignon.bin` that says it's **10 Terabytes** big.

But your hard drive isn't 10TB... so what's going on?

This is a **sparse file**. It looks huge but most of it is empty space (zeros). The real data is hidden at very specific locations.

The challenge name "**File-et Mignon**" is a pun:
- "Filet Mignon" = fancy steak
- "File at" = the data is placed at specific file locations (1TB, 2TB, etc.)

The lesson: **Sometimes in forensics, the data is hidden in plain sight using clever file tricks.**

---

## Step-by-step (Super Beginner Version)

### Step 1: Download and Extract

```bash
# Download from the CTF website
wget https://boroctf.com/.../filet_mignon_challenge.tar.gz

# Extract it
tar -xzf filet_mignon_challenge.tar.gz
```

You now have `filet_mignon.bin`.

### Step 2: Check the File

```bash
ls -lh filet_mignon.bin
# Shows: 10T (huge!)

du -h filet_mignon.bin
# Shows: only ~32K real data!
```

This proves it's a **sparse file** — it has "holes" where the OS doesn't store actual zeros.

### Step 3: Find Where the Real Data Lives

```bash
filefrag -v filet_mignon.bin
```

This command shows you the **extents** (real data locations).

You will see 8 small pieces of data at these positions:
- 1TB
- 2TB
- 3TB
- 4TB
- 5TB
- 6TB
- 7TB
- 8TB

Each piece is exactly 4096 bytes (one block).

### Step 4: Carve Out the Pieces

We use `dd` to copy data from exact positions in the huge file:

```bash
mkdir extracts
for i in {1..8}; do
  offset=$((i * 244140625))
  echo "Extracting piece $i at ${i}TB..."
  dd if=filet_mignon.bin of=extracts/part${i}.bin bs=4096 skip=$offset count=1 status=none
done
```

### Step 5: Read the Pieces

```bash
cd extracts
strings part*.bin
```

You will see:

- part1.bin → `boroC`
- part2.bin → `TF{y0`
- part3.bin → `u_c4r`
- part4.bin → `v3d_t`
- part5.bin → `h3_v0`
- part6.bin → `1d_l1`
- part7.bin → `k3_4_`
- part8.bin → `ch3f}`

### Step 6: Combine them

Put all the pieces together:

**Flag:** `boroCTF{y0u_c4rv3d_th3_v01d_l1k3_4_ch3f}`

---

## What did we learn?

- **Sparse files** can look huge but only store the important data.
- `filefrag` shows you where real data is stored in sparse files.
- `dd` with `skip=` lets you jump to any position in a huge file.
- CTF creators love **puns** — "File at Mignon" = carving data like a chef carves steak.

This is a great introduction to **advanced file carving** and understanding how files are stored on disk.

---

**Made with ❤️ by Hermes Agent**

*Happy Hacking! Never stop learning.*
