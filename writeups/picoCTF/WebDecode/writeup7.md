---
title: "WebDecode"
platform: "picoCTF"
category: "Web Exploitation"
difficulty: "Easy"
date: 2025-12-23
tags: ["web", "base64", "inspector", "browser-tools"]
time_spent: "15m"
license: "CC BY 4.0"
---

# TL;DR

Navigated to the "About" page of the challenge website and used the Browser Inspector to examine the HTML source code. Found a suspicious Base64 encoded string assigned to a `notify_true` attribute within a section tag. Decoding this string revealed the flag.

**picoCTF{web_succ3ssfully_d3c0ded_02cdcb59}**

# Challenge

[picoCTF - **WebDecode**](https://play.picoctf.org/practice/challenge/427) - Do you know how to use the web inspector? Start searching here to find the flag.

# What I needed

* **Browser:** Firefox (Kali Linux) Developer Tools.
* **Tools:** Terminal (for `base64` command) or CyberChef.

# Summary (what you will do)

1. Launch the challenge instance and navigate to the provided URL.
2. Explore the site's subpages, specifically the **About** page.
3. Open the **Inspector** (F12) to view the raw HTML.
4. Locate the `notify_true` attribute containing an encoded string.
5. Decode the Base64 string to obtain the flag.

# Step-by-step walkthrough

## 1) Information Gathering

The challenge description asks if I know how to use the web inspector. After launching the instance, I navigated to the site and saw a simple layout with navigation links for **HOME**, **ABOUT**, and **CONTACT**.

## 2) Inspecting the "About" Page

Based on the hint to "use the web inspector on other files," I moved away from the landing page.

1. Clicked on the **ABOUT** link in the navigation bar.
2. Right-clicked the page and selected **Inspect** (or pressed `F12`) to open the Developer Tools.

## 3) Finding the Encoded String

While reviewing the HTML structure in the **Inspector** tab, I noticed an unusual attribute in one of the section tags.

**Location Found:**
```html
<section class="about" notify_true="cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfMDJjZGNiNTl9">
  ...
</section>
```

The string `cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfMDJjZGNiNTl9` appeared to be **Base64** encoded due to its alphanumeric character set.

## 4) Decoding the Flag

I copied the string and used the Kali Linux terminal to decode it. (you can also use CyberChef)

**Command:**

```bash
echo "cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfMDJjZGNiNTl9" | base64 -d

```

**Output:**
`picoCTF{web_succ3ssfully_d3c0ded_02cdcb59}`

# Troubleshooting tips & explanations

## Why was it hidden there?

This challenge demonstrates how developers might hide data in **Custom Data Attributes**. While the `class` attribute is standard, `notify_true` is a custom attribute. These are often used by JavaScript to store state or metadata that isn't immediately visible to the user but is easily accessible via the DOM (Document Object Model).

## Identifying Base64

You can identify Base64 by looking for:

* A string consisting of uppercase letters (A-Z), lowercase letters (a-z), numbers (0-9), and sometimes `+` and `/`.
* Padding characters (`=`) at the end of the string (though not present in every single instance).

## Other places to check

If the flag wasn't in the HTML tags, standard web forensics would suggest checking:

* **CSS files:** Check `style.css` for hidden comments or font-face strings.
* **JavaScript files:** Look for hardcoded variables in the **Debugger** tab.
* **Network Headers:** Look for custom HTTP headers in the **Network** tab.

© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)