---
## Challenge Writeup: x-tension

**Category:** Forensics


**Description Hint:**

> “Trying to get good at something while watching youtube isn’t the greatest idea...”

* The hint suggests that the user’s activity was being monitored, possibly via a **browser extension**, while they were distracted.
* The challenge provided a **pcap file**: `chal.pcapng`.

---

### Step 1: Inspect the pcap

* Open `chal.pcapng` in **Wireshark**.
* Filter for HTTP traffic:

  ```
  http.request
  ```
* We notice a **GET request for a Chrome extension**:

  ```
  GET /FunnyCatPicsExtension.crx HTTP/1.1
  Host: 192.168.20.1:1023
  ```
* The server responds with the `.crx` file containing `content.js` and `manifest.json`.

💡 This is likely **where the exfiltration mechanism is hidden**.

---

### Step 2: Analyze the extension

* `manifest.json` shows a content script runs on **all URLs** (`<all_urls>`).
* `content.js` contains obfuscated JavaScript. After deobfuscation, we see:

1. `getKey()` generates a **single-character key** based on the **current UTC minute + 0x20**.
2. `xorEncrypt()` XORs each character of the typed input with this key and encodes it as hex.
3. An event listener on `keydown` sends the **XOR-encrypted value** to a remote server:

   ```
   http://192.9.137.137:42552/?t=ENCRYPTED_HEX
   ```
4. Essentially, the extension **acts as a keylogger**, exfiltrating the flag one character at a time.

---

### Step 3: Extract encrypted characters from the pcap

* Use **PyShark** to parse HTTP requests in the pcap:

  * Filter for requests to `192.9.137.137:42552`.
  * Extract the `t` parameter (hex-encoded XORed character).
  * Compute the key using the **packet timestamp**: `UTC minute + 0x20`.

---

### Step 4: Decrypt the flag

* Each character is decrypted with:

```python
def xor_decrypt(enc_hex, key):
    enc_bytes = bytes.fromhex(enc_hex)
    return ''.join([chr(b ^ key) for b in enc_bytes])
```

* Concatenate all decrypted characters in the order of packet timestamps to reconstruct the **full flag**.

---

### Step 5: Python Script (Automation)

* The full Python script automates extraction and decryption from the pcap (see previous message).
* Running it yields the full flag.

---

### Step 6: Flag

```
ictf{funny_cat_extension_xored_my_password}
```

---

### Lessons Learned

1. **Browser extensions are a common attack vector** — even harmless-looking ones can exfiltrate sensitive info.
2. **Network forensics** is critical to detect such exfiltration:

   * Follow HTTP streams or TCP streams in Wireshark.
   * Look for unusual hosts or ports.
3. **XOR encryption with predictable keys** (like UTC minutes) is trivial to break.
4. Always monitor extensions, especially those with `<all_urls>` permissions.

---