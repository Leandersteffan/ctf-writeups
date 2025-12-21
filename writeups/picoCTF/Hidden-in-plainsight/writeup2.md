---
title: "Hidden in plainsight"
platform: "picoCTF"
category: "Forensics"
difficulty: "Easy"
date: 2025-11-23
tags: ["forensics","steganography","exiftool","steghide","base64"]
time_spent: "15m"
license: "CC BY 4.0"
-------------------------------------------------------------

# TL;DR

Found a flag hidden inside an image using steganography. I used `exiftool` to find a hidden Base64 string in the metadata, decoded it twice to reveal a `steghide` password, and used that password to extract the flag file from the image.

**picoCTF{h1dd3n_1n_1m4g3_54e31417}**

# Challenge

[picoCTF - **Hidden in plainsight**](https://play.picoctf.org/practice/challenge/524) - Computers are smart, but they only do what we tell them to. Can you find the flag hidden in the image?

# What I needed

* **OS:** Kali Linux (or any Linux with the tools installed)
* **Tools:** `exiftool`, `base64`, `steghide` (Standard in Kali)

# Summary (what you will do)

1. Analyze the image metadata with `exiftool`.
2. Find a suspicious Base64 string in the comments.
3. Decode the string to reveal a hint (tool and second encoded string).
4. Decode the second string to get the password.
5. Use `steghide` with the password to extract the flag.

# Step-by-step walkthrough

## 1) Find the Hidden Metadata

First, we look for data hidden in the file headers that isn't part of the visible image.

Run `exiftool` on the image:

```bash
exiftool img.jpg
```

**What to look for:**
Scan the output for unusual strings. You will see a `Comment` field containing a long, random-looking string.

**Found String:**
`c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9`

## 2) The First Decode (Layer 1)

The string is Base64 encoding. We decode it:

```bash
echo "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9" | base64 -d
```

**Result:**
`steghide:cEF6endvcmQ=`

## 3) Analyze and Decode Again (Layer 2)

The output provides a clear instruction in the format `Tool:Password`.
*   **Left side (`steghide`)**: Tells us which tool to use.
*   **Right side (`cEF6endvcmQ=`)**: Another Base64 string (note the `=` padding).

We must decode that second part to get the actual password.

```bash
echo "cEF6endvcmQ=" | base64 -d
```

**Result:**
`pAzzword`

Now we have the password!

## 4) Extract the Payload

Now we have all the components:
*   **Tool:** `steghide`
*   **File:** `img.jpg`
*   **Password:** `pAzzword`

Run the extraction command (`-sf` stands for "source file" and `-p` for "passphrase"):

```bash
steghide extract -sf img.jpg -p pAzzword
```

**Output:**
`wrote extracted data to "flag.txt".`

## 5) Capture the Flag

Finally, read the content of the extracted file:

```bash
cat flag.txt
```

**Flag:**
`picoCTF{h1dd3n_1n_1m4g3_54e31417}`

# Troubleshooting tips & explanations

## Why did I do that?
In Cybersecurity and CTFs, this is known as **Obfuscation**.

*   **Metadata Hiding:** They hid data where normal users don't look (Exif tags).
*   **Base64:** They made the data unreadable to the human eye.
*   **Nested Encoding:** They encoded the password *inside* the encoded hint to ensure you were paying attention to the output structure.

## `steghide` fails?
*   Ensure you are typing the password exactly as decoded (`pAzzword`), it is case-sensitive.
*   Make sure `steghide` is installed (`sudo apt install steghide`).

© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)

