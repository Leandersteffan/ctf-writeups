---
title: "Puzzled"
platform: "picoCTF"
category: "Forensics"
difficulty: "Easy"
date: 2025-12-26
tags: ["forensics", "metadata", "exiftool", "base64", "pdf"]
time_spent: "10m"
license: "CC BY 4.0"
---

# TL;DR

The challenge involved analyzing a PDF file named `confidential.pdf`. After finding no clues in the visible text, I inspected the file's metadata using `exiftool`. I discovered a Base64-encoded string within the **Author** field. Decoding this string revealed the flag.

**picoCTF{puzzl3d_m3tadata_f0und!_ee454950}**

# Challenge

[picoCTF - **Puzzled**](https://play.picoctf.org/practice/challenge/530) - Sometimes, the most important information isn't written on the page. We found this "confidential" document, but the text seems like a dead end. Can you figure out the secret way in?

# What I needed

* **Terminal:** A Linux or macOS terminal (or WSL on Windows).
* **Tools:** 
    * `exiftool` (for reading file metadata).
    * [CyberChef](https://gchq.github.io/CyberChef/) (for Base64 decoding).

# Summary (what you will do)

1. Download the challenge file `confidential.pdf`.
2. Attempt to read the PDF and confirm the visible text is irrelevant.
3. Use `exiftool` to extract the file's metadata.
4. Identify a suspicious Base64 string in the **Author** metadata field.
5. Decode the string using CyberChef to retrieve the flag.


# Step-by-step walkthrough

## 1) Information Gathering
I downloaded the file `confidential.pdf`. Upon opening it, the document contained generic text that did not lead to a flag or any obvious hints. In CTF forensics challenges involving documents, the next logical step is to check for hidden information "behind" the file content.

## 2) Extracting Metadata
I opened my terminal and navigated to the directory containing the file. I used `exiftool`, a powerful tool for reading and editing meta information in a wide variety of files.

**Command executed:**
```bash
exiftool confidential.pdf
```

**Output Analysis:**
The tool returned several lines of data. However, one specific entry stood out:

`Author : cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV9lZTQ1NDk1MH0=`

## 3) Decoding the Hint (Base64)

The string in the Author field ended with an `=` sign and consisted of an alphanumeric mix. This is a classic indicator of **Base64** encoding.

1. Copy the string: `cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV9lZTQ1NDk1MH0=`
2. Paste it into **CyberChef**.
3. Apply the **From Base64** operation.

**Decoded Result:**
`picoCTF{puzzl3d_m3tadata_f0und!_ee454950}`

# Troubleshooting tips & explanations

## Why did this work?

This challenge demonstrates the importance of **Metadata Forensics**.

* **The Metadata**: Files like PDFs contain metadata—information about the file itself (creator, software used, timestamps). Developers or attackers can hide data here that isn't visible when the file is opened normally.
* **The Encoding**: While metadata is "hidden" from the average user, it is easily accessible via tools. To make it slightly harder to find via simple text searches (like `grep`), the flag was encoded in Base64.

## Failed to get the flag?

* **Exiftool not installed**: If `exiftool` is not available, you can try the `strings` command: `strings confidential.pdf | grep Author`.
* **Copy-Paste Errors**: Ensure you copy the entire string including the trailing `=` sign, as it is necessary for correct Base64 padding.

© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)
