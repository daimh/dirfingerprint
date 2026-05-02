# Dirfingerprint

<details>
<summary>Table of contents</summary>

<ol>
  <li><a href="#about">About</a></li>
  <li><a href="#install">Install</a></li>
  <li><a href="#example">Example</a></li>
  <li><a href="#license">License</a></li>
  <li><a href="#acknowledgment">Acknowledgment</a></li>
</ol>

</details>

---

## About

Comparing large directories is often painful, especially across different filesystems.

The standard `du` command is not always helpful because it includes metadata size. For example, a directory containing only empty files can still report a non-zero size, and that value varies depending on the filesystem. Most users only care about the size of actual file contents, not metadata.

Another common problem: detecting directories that have been renamed or moved.

On top of that, using `rsync` for large transfers is inefficient since it is not parallel. In practice, running multiple `rsync` processes significantly improves performance, especially for directories that have a lot of small files.

**Dirfingerprint** addresses these issues by generating a set of properties for every subdirectory (recursively). These properties can then be used to:

- Compare directory trees
- Detect moved or renamed directories
- Generate parallel `rsync` scripts

### Reported Properties

Each subdirectory is described by:

- **FingerPrint**  
  An MD5 hash based on three components:
  1. Counts of `SubdirCount` and `SpecialCount`
  2. List of `(name, size, mtime)` for regular files (one level deep)
  3. List of `(name, FingerPrint)` for subdirectories (one level deep)

- **FileBytes**  
  Total size of regular files only (always ≤ `du -b`)

- **FileCount**  
  Number of regular files

- **FileMinMtime**  
  Oldest modification time among files

- **FileMaxMtime**  
  Newest modification time among files

- **SubdirCount**  
  Number of subdirectories

- **SpecialCount**  
  Number of non-regular, non-directory files

- **Depth**  
  Depth of the subdirectory (root = 0)

- **Dir**  
  Subdirectory name

---

## Install

```bash
wget https://raw.githubusercontent.com/daimh/dirfingerprint/master/dirfingerprint
chmod +x dirfingerprint
./dirfingerprint -h
```

---

## Example

### Generate fingerprints for all subdirectories under `/tmp`

```bash
(cd /tmp; dirfingerprint hash) > 0.dfp
```

### Detect new or moved directories

```bash
mkdir /tmp/1 /tmp/2

(cd /tmp; dirfingerprint hash) > 1.dfp
dirfingerprint diff 0.dfp 1.dfp

mv /tmp/1 /tmp/2

(cd /tmp; dirfingerprint hash) > 2.dfp
dirfingerprint diff 1.dfp 2.dfp
dirfingerprint diff 0.dfp 2.dfp
```

### Generate parallel `rsync` commands

```bash
dirfingerprint rsync 0.dfp 2.dfp user@host:src/
dirfingerprint rsync 0.dfp 2.dfp user@host:src/ | parallel -j 8
```

### Filter directories by depth (≤ 2)

```bash
awk '$8 < 2' 0.dfp
```

### Sort level-3 subdirectories by size

```bash
awk '$8 == 2' 2.dfp | sort -k 2n
```

---

## License

Developed by [Manhong Dai](mailto:daimh@umich.edu)

Copyright © 2022 University of Michigan

Licensed under [GPLv3+](https://gnu.org/licenses/gpl.html)  
(GNU GPL version 3 or later)

This is free software: you are free to modify and redistribute it.

**No warranty** is provided, to the extent permitted by law.

---

## Acknowledgment

- Ruth Freedman, MPH — Former Administrator, MNI, UMICH  
- Fan Meng, Ph.D. — Research Associate Professor, Psychiatry, UMICH  
- Huda Akil, Ph.D. — Director, MNI, UMICH  
- Stanley J. Watson, M.D., Ph.D. — Director, MNI, UMICH  
