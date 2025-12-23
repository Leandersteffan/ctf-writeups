---
title: "DISKO 1"
platform: "picoCTF"
category: "Forensics"
difficulty: "Easy"
date: 2025-12-23
tags: ["forensics", "disk-image", "strings", "grep"]
time_spent: "5m"
license: "CC BY 4.0"
---

# TL;DR

Found the flag by performing a simple string search on the provided raw disk image. After extracting the compressed file, running the `strings` command filtered through `grep` allowed me to retrieve the flag instantly from the disk's binary data.

**picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}**

# Challenge

[picoCTF - **DISKO 1**](https://play.picoctf.org/practice/challenge/505) - Can you find the flag in this disk image?

# What I needed

* **OS:** Kali Linux.
* **Tools:** `gunzip` (to extract the image), `strings`, and `grep`.

# Summary (what you will do)

1. Download the compressed disk image file `disko-1.dd.gz`.
2. Decompress the file to get the raw `.dd` image.
3. Use the `strings` utility to extract all printable characters from the binary.
4. Filter the output using `grep` to find the specific flag format.


# Step-by-step walkthrough

## 1) Information Gathering

After downloading the file from the challenge link, I identified it as a compressed Gzip archive containing a disk image named `disko-1.dd`.

1. Open the terminal in the directory containing the file.
2. Extract the archive:
   `gunzip disko-1.dd.gz`

## 2) Analyzing the Hint

The challenge description provided a specific hint: *"Maybe Strings could help? If only there was a way to do that?"*. In forensics, the `strings` command is the standard way to find human-readable text hidden inside binary files like disk images.

## 3) Executing the Search

Since I knew the flag format for picoCTF, I piped the output of `strings` into `grep` to find the flag immediately.

**Command executed:**
`strings disko-1.dd | grep "picoCTF"`

## 4) Capturing the Flag

The command returned the flag directly from the raw data sectors of the disk image.

**Result:**
`picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}`

# Troubleshooting tips & explanations

## Why did this work?

This challenge demonstrates the simplicity of **Unallocated Space/Plaintext storage**.

* **The Hint**: The challenge author points towards `strings` because many beginners overlook that data on a disk is often stored without encryption or obfuscation.
* **Raw Images**: A `.dd` file is a bit-for-bit copy. If a flag is written to a text file on that disk, it exists as literal ASCII characters in the binary.

## Failed to find the flag?

* **Case Sensitivity**: Ensure you use `grep "picoCTF"` or `grep -i "pico"` to account for case sensitivity.
* **Extraction**: Make sure the `.gz` file was fully extracted; running `strings` on the compressed archive will not yield the flag.

© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)