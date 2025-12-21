---
title: "Cookie Monster Secret Recipe"
platform: "picoCTF"
category: "Web"
difficulty: "Easy"
date: 2025-12-21
tags: ["cookies", "base64", "decoding"]
time_spent: "15m"
license: "CC BY 4.0"
---

# TL;DR

Found a Base64 encoded flag hidden within a session cookie named `secret_recipe` after interacting with the site; decoded via CyberChef.

**picoCTF{c00k1e_m0nster_l0ves_c00kies_2C8040EF}**

# Challenge

[picoCTF - **Cookie Monster Secret Recipe**](https://play.picoctf.org/practice/challenge/469) - Find the secret recipe hidden on the provided web instance.

# What I needed

* Browser: Firefox or Chrome (for Developer Tools)
* Tools: [CyberChef](https://gchq.github.io/CyberChef/)
* Knowledge: Basic understanding of HTTP cookies and Base64 encoding.

# Summary (what you will do)

1. Launch the challenge instance and navigate to the web page.
2. Interact with the site (attempt login) to force the server to set a session cookie.
3. Use Browser Developer Tools to inspect the stored cookies.
4. Identify the `secret_recipe` cookie and extract its Base64-encoded value.
5. Decode the value using CyberChef to reveal the flag.

# Step-by-step walkthrough

## Step 1: Launch and Access the Instance

1.  On the picoCTF challenge page, click **"Launch Instance"**.
2.  Once the status changes to "Running," click the link provided in the description text ("You can access the Cookie Monster **here**").

## Step 2: Triggering the Cookie

Once on the website, you are greeted with an "Access Denied" message or a login screen.

  * **Action:** Attempt to interact with the site (e.g., try logging in with credentials like `admin` / `admin`).
  * **Note:** While the login might fail or show "Access Denied," this interaction forces the server to set a session cookie in your browser.
  * *Pro Tip:* If you don't see the specific cookie mentioned in the next step immediately, try clearing your browser cookies for this site and refreshing or logging in again. This ensures a fresh session is created.

## Step 3: Inspecting Developer Tools

Now we need to see what the "Cookie Monster" left behind in our browser.

1.  Open your browser's **Developer Tools** (Press `F12` or `Ctrl + Shift + I`).
2.  Navigate to the **Storage** tab (Firefox) or **Application** tab (Chrome).
3.  On the left sidebar, expand the **Cookies** section and click on the URL for the challenge (e.g., `verbal-sleep.picoctf.net`).
4.  Look through the list of cookies. You will spot a suspicious one named:
      * **Name:** `secret_recipe`
      * **Value:** `cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzJDODA0MEVGfQ%3D%3D`

This "Value" looks like a scrambled string, but the character set (A-Z, a-z, 0-9) and the padding at the end (`%3D%3D` is URL-encoded `==`) strongly suggest it is **Base64 encoded**.

## Step 4: Decoding the "Recipe" with CyberChef

To read the secret, we need to decode this string.

1.  Copy the entire **Value** string from the cookie.
2.  Go to **[CyberChef](https://gchq.github.io/CyberChef/)** (a popular tool for data decoding).
3.  Paste the cookie value into the **Input** box.
4.  In the "Operations" search bar, type `Base64` and drag the **From Base64** block into the "Recipe" area.
5.  Look at the **Output** box. The random characters will be converted into readable text.

**Input:**

```text
cGljb0NURntjMDBrMWVfbTBuc3Rlcl9sMHZlc19jMDBraWVzXzJDODA0MEVGfQ%3D%3D
```

**Output:**

```text
picoCTF{c00k1e_m0nster_l0ves_c00kies_2C8040EF}
```

-----

## 🚩 Final Flag

**`picoCTF{c00k1e_m0nster_l0ves_c00kies_2C8040EF}`**

-----

### Key Takeaway

Web developers often use cookies to store session data. If sensitive data (like a flag or a "secret recipe") is stored in a cookie without proper encryption (Base64 is **encoding**, not encryption), anyone with access to the browser can easily read and manipulate it\!



# Troubleshooting tips & explanations

* **Cookie not appearing?** Try clearing your browser cache or cookies for that specific site and refreshing. Some instances require a fresh "handshake" to set the cookie.


© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)