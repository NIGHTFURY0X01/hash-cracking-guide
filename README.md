# Hash Cracking Guide — Reference Table, Installation, rockyou.txt, and Advanced Usage (Deep Dive)

**Author:** NIGHTFURY0X01 (Arash)

---

## A comprehensive, copy-ready Markdown guide explaining:
 - a quick hash length → type → `hashcat` mode reference table  
 - how to install `hashcat` on Linux (multiple distros)  
 - obtaining and preparing `rockyou.txt` (and other wordlists)  
 - practical `hashcat` commands and workflows for MD5 / SHA1 / SHA256 examples  
 - advanced cracking strategies, performance tips, and operational notes

---

## Hash Length Reference Table

| Length    | Hash Type | Hashcat Mode |
|-----------|-----------|--------------|
| 32 chars  | MD5       | `-m 0`       |
| 40 chars  | SHA1      | `-m 100`     |
| 64 chars  | SHA256    | `-m 1400`    |
| 128 chars | SHA512    | `-m 1700`    |

### Common Examples
- **MD5 (32)**: `482c811da5d5b4bc6d497ffa98491e38`  
- **SHA1 (40)**: `b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3`  
- **SHA256 (64)**: `916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745`

---

## Installing hashcat on Linux

> Prefer using your distribution package manager for convenience — but if you need the latest version or GPU drivers, consider downloading official releases and installing GPU drivers (NVIDIA/AMD) separately.

### Debian / Ubuntu (Apt)
```bash
sudo apt update
sudo apt install -y hashcat
# verify
hashcat --version
```
## Fedora 
```bash 
sudo dnf install -y hashcat
hashcat --version
```

## Arch Linux (Pacman)
```bash
sudo pacman -Syu hashcat
hashcat --version
```

---

## Installing official hashcat release (binary)

+ 1)  Download release from the official site (hashcat.net) or GitHub releases.

+ 2)  Extract and run the binary `(./hashcat.bin` or symlink to `hashcat`).

## GPU drivers
+ NVIDIA: install the latest proprietary driver & CUDA toolkit as needed.

+ AMD: install AMDGPU-PRO or ROCm if supported.

+ Make sure `nvidia-smi` (NVIDIA) or appropriate tools show your GPU and that `hashcat` can see devices:

```bash
hashcat -I        # show OpenCL/ROC availability
hashcat -b        # benchmark (quick/long) - will test kernels
```

---

### rockyou.txt — obtaining & preparing wordlists

+ `rockyou.txt` is a widely used password wordlist (commonly from leaked password dumps) used for dictionary attacks. Use it responsibly and only on targets you are authorized to test (CTF, your systems, or with permission).

## Download (example repository)
```bash
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
# or from other curated sources if you have a local copy:
# cp /usr/share/wordlists/rockyou.txt /path/to/your/dir
```

## If compressed (e.g., rockyou.txt.gz):
```bash
gunzip rockyou.txt.gz
# or
zcat rockyou.txt.gz > rockyou.txt
```
## Best practices 
+ Keep wordlists organized: `wordlists/rockyou.txt`, `wordlists/combos.txt`, `wordlists/custom.txt`.

+ Combine or filter lists with `cat`, `sort -u`, `grep -v` to tailor sizes.

+ Use `split` or `head` when testing to avoid long runs during tuning:

```bash
# test with first 100k candidates
head -n 100000 rockyou.txt > rockyou-small.txt
```
--- 

### Preparing hashes for hashcat 

+ Single-hash (quick): you can give a single hash on the command line (hashcat accepts it as the "hashfile" argument), e.g. `hashcat -m 0 <hash> rockyou.txt`

+ Multiple hashes: put each hash on its own line in a file, then run `hashcat -m <mode> <hashfile> <wordlist>`
```bash 
# Example: create a file for the three example hashes
printf "482c811da5d5b4bc6d497ffa98491e38\nb7a875fc1ea228b9061041b7cec4bd3c52ab3ce3\n916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745\n" > hashes.txt
```
+ If hashes have prefixes (like `sha1$...` or salts), ensure you use the correct hash mode and format as needed by `hashcat` (read hashcat docs).

## Basic `hashcat` commands — copy-ready examples
```bash
Modes used:

MD5 → -m 0

SHA1 → -m 100

SHA256 → -m 1400
```
+ Crack a single MD5 using rockyou : 
```bash 
hashcat -m 0 -a 0 482c811da5d5b4bc6d497ffa98491e38 rockyou.txt
>> 482c811da5d5b4bc6d497ffa98491e38:password123
```
+ Crack a single SHA1 using rockyou 
```bash
hashcat -m 100 -a 0 b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3 rockyou.txt
>> b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3:letmein
```
+ Crack a single SHA256 using rockyou (optimized kernels)
```bash
hashcat -m 1400 -a 0 -O 916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745 rockyou.txt
>> 916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745:qwerty098
```
+ Crack multiple hashes from a file
```bash 
hashcat -m 0 -a 0 hashes.txt rockyou.txt        # if hashes.txt contains only MD5s
# or specify correct -m for the file if all are same mode
```
+ Crack multiple hashes from a file
```bash
hashcat -m 0 -a 0 hashes.txt rockyou.txt        # if hashes.txt contains only MD5s
# or specify correct -m for the file if all are same mode
```
---
### Important common flags

+ `-a 0` → dictionary attack (wordlist)

+ `-a 3` → mask attack (brute force masks)

+ `-O` → optimized kernels (faster but some length/charset limits)

+ `-w 3` → workload profile (1..4) for speed/IO tradeoffs

+ `--session name` → name session for restore

+ `--restore` → restore interrupted session

+ `--potfile-path /path/to/potfile` → change potfile location

+ `--show` → show cracked hashes from a potfile or hashfile

+ `--rules-file <file>` → use custom rules


---

### Advanced attack modes & strategies

1) Dictionary + Rules (best first step)
+ Use a baseline wordlist + rule set (e.g., `rockyou.txt` + `rules/best64.rule`) to mutate candidates:
```bash
hashcat -m 100 -a 0 target.hash rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```
+ Rules introduce common mutations (capitalization, leet, suffixes, etc.) without increasing wordlist size too much.

2) Mask attack (targeted brute-force)
+ If you know structure (e.g., `qwerty` + 3 digits), use masks:
```bash
# ?l = lower, ?u = upper, ?d = digit, ?s = special
hashcat -m 1400 -a 3 target.hash ?l?l?l?l?l?d?d?d
```
3) Combinator (combine two lists)
+ Combine two wordlists:
```bash
hashcat -m 0 -a 1 target.hash list1.txt list2.txt
```
4) Hybrid attacks (dictionary + mask)
+ Dictionary candidate followed by mask append/prepend:
```bash
# dictionary + 2-digit numeric suffix
hashcat -m 100 -a 6 target.hash rockyou.txt ?d?d
```
5) Rules + Masks
+ You can chain strategies: use a smaller candidate set with aggressive masks, and rules to mutate.

---

### Performance tuning & best practices
+ GPU vs CPU: GPU is orders of magnitude faster for many hash types. Use GPU-enabled systems where possible.

+ Use `hashcat -b` to benchmark kernels and see expected speeds per mode.

+ Chunk your runs: test with `--increment` or smaller wordlist first to validate configs.

+ Use `--session` and `--restore` to recover long jobs:

```bash
hashcat --session myrun -m 1400 -a 0 hashes.txt rockyou.txt
# Ctrl+C during run, later:
hashcat --session myrun --restore
```
+ Potfile: cracked passwords are stored in `~/.hashcat/hashcat.potfile` by default. Use `hashcat --show` to display cracked results.

+ Avoid `-O` if you need long candidates: `-O` trades correctness for speed by using optimized kernels with constraints.

+ Use multiple rules in staging: run fast rules first (best64), then heavier rule sets (rules/dive.rule) if needed.

--- 

### Common pitfalls & troubleshooting

+ Wrong `-m` mode → no results. Double-check hash length & format. Some algorithms include salts or extra metadata.

+ Salted hashes → must use the correct `-m` for salted formats (e.g., `-m 10` for MySQL MyISAM, etc.). Check hashcat's help for format list.

+ Encoding / case → some hashes might be uppercase hex; hashcat accepts both but be consistent.

+ Too slow on CPU → consider mask/rule, reduce search space, or use a faster GPU.

+ OpenCL errors → ensure GPU drivers and OpenCL are properly installed. Run `hashcat -I` to inspect devices.

--- 

### Example end-to-end workflow (practical)

1) Save hash
```bash
echo "482c811da5d5b4bc6d497ffa98491e38" > md5.txt
```

2) Test with small subset
```bash
head -n 50000 rockyou.txt > rock_small.txt
hashcat -m 0 -a 0 md5.txt rock_small.txt
```

3) Full run
```bash
 hashcat -m 0 -a 0 md5.txt rockyou.txt --session pico_md5 -w 3
```

4) Check potfile / show results
```bash
hashcat --show -m 0 md5.txt
```

5) Restore if interrupted
```bash
hashcat --session pico_md5 --restore
```

---

### Ethics, legality & operational security

+ Only test hashes that you own or have explicit permission to crack (CTFs, pen tests with authorization, lab environments).

+ Do not use leaked wordlists against third-party services or accounts you do not own.

+ Keep GPU drivers and system secure; long cracking runs can expose hardware to heavy load — monitor temps and power.

+ Log and document your activities when performing authorized penetration testing.

---

### Quick reference: Useful hashcat commands
```bash
# Benchmark
hashcat -b

# Show cracked (from potfile)
hashcat --show -m 0 md5.txt

# Restore interrupted session
hashcat --session mysession --restore

# Use rules
hashcat -m 100 -a 0 sha1.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Mask attack
hashcat -m 1400 -a 3 sha256.txt ?l?l?l?l?d?d

# Combine lists
hashcat -m 0 -a 1 targets.txt listA.txt listB.txt
```

---

### Final notes (for the three example hashes)

+ MD5 example: `hashcat -m 0 -a 0 482c811da5d5b4bc6d497ffa98491e38 rockyou.txt`

+ SHA1 example: `hashcat -m 100 -a 0 b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3 rockyou.txt`

+ SHA256 example: `hashcat -m 1400 -a 0 -O 916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745 rockyou.txt`

## If you want I can:

+ produce a single runnable script that runs these three tasks and prints results (requires hashcat installed), or

+ generate a compact cheat-sheet PDF from this Markdown, or

+ tailor the guide with recommended rulesets and concrete rule examples (e.g., `best64.rule`, `rockyou-3000.rule`) and show expected runtime estimates for different GPU classes.

