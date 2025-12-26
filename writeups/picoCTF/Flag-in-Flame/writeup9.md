---
title: "Flag in Flame"
platform: "picoCTF"
category: "Forensics"
difficulty: "Easy"
date: 2025-12-26
tags: ["forensics", "base64", "steganography", "hex", "picoMini"]
time_spent: "20m"
license: "CC BY 4.0"
---

# TL;DR

Found an enormous block of Base64 encoded text in a suspicious log file. After decoding the text into a PNG image, I discovered a hidden line of Hexadecimal code at the bottom of the image. Converting this Hex string to ASCII revealed the flag.

**picoCTF{forensics_analysis_is_amazing_c75dd08e}**

# Challenge

[picoCTF - **Flag in Flame**](https://play.picoctf.org/practice/challenge/523) - The SOC team discovered a suspiciously large log file after a recent breach. Instead of typical logs, they found an enormous block of encoded text. Your mission is to inspect the resulting file and reveal the real purpose of it.

# What I needed

* **Terminal:** A Linux or macOS terminal (for Base64 decoding).
* **Tools:** [CyberChef](https://gchq.github.io/CyberChef/) (for Hex decoding) and an Image Viewer.

# Summary (what you will do)

1. Download the challenge file `logs.txt`.
2. Analyze the file and identify the Base64 encoding.
3. Decode the Base64 data to reconstruct a hidden image file.
4. Inspect the resulting image for visual clues.
5. Extract a Hexadecimal string from the image.
6. Convert the Hex string to plain text to find the flag.


# Step-by-step walkthrough

## 1) Analyzing the Logs
After downloading `logs.txt`, I opened the file and noticed it didn't contain standard server logs. Instead, it was filled with a massive, continuous block of characters, which is a common indicator of a file being embedded as text.

## 2) Decoding the Base64 Data
The hint suggested that the data was Base64 and should generate an image. I used the terminal to decode the text file and output it as a PNG.

**Command executed:**
```bash
base64 -d logs.txt > decoded_image.png
```

## 3) Visual Inspection

I opened `decoded_image.png`. While the image itself was the primary output, the real secret was hidden in plain sight. At the very bottom of the image, there was a string of characters that looked like Hexadecimal (consisting only of numbers 0-9 and letters A-F).

**Hex String Found:**
`7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F63373564643038657D`

## 4) Decoding the Flag (Hex to ASCII)

I needed to convert this Hex sequence into readable text to get the final flag.

1. Open **CyberChef**.
2. Paste the Hex string into the **Input** area.
3. Search for and apply the **"From Hex"** recipe.

**Decoded Result:**
`picoCTF{forensics_analysis_is_amazing_c75dd08e}`

# Troubleshooting tips & explanations

## Why did this work?

This challenge uses two layers of **data hiding**:

* **File Embedding**: Using Base64 to hide a binary file (like an image) inside a text-based log file is a common way to exfiltrate data past simple security filters.
* **Visual Steganography**: Placing the flag (or a encoded version of it) directly into the pixel data of an image is a classic forensics technique. Because it was in Hex, it wasn't immediately searchable as a "picoCTF" string until decoded.

## Common Issues

* **Corrupt Image**: If the image doesn't open, ensure you didn't have any extra characters (like "Author:" or "Logs:") at the start of your `logs.txt` before decoding. The Base64 string must be "clean."
* **OCR Errors**: If you type the Hex string manually from the image, double-check characters like `0` (zero) vs `O` (letter O) and `1` (one) vs `I` (letter I). In Hex, you will only see `0-9` and `A-F`.

© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)
