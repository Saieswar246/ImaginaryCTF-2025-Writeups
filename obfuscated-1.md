### Challenge Writeup: Obfuscated-1
**Category:** Forensics

### Challenge Description
I installed every old software known to man... The flag is the VNC password.

### The Goal
The objective of this forensics challenge was to find the VNC password, which is the flag, from a provided challenge files. The password was hidden in a Windows registry file.

### Step 1: Initial Forensics and Registry Analysis
The first step was to locate and analyze the user's Windows registry hive, NTUSER.DAT. This file contains user-specific registry data. To make the data readable and searchable, it was exported to a .reg file using a forensics tool.

The command used to export the registry hive was:

```Bash
    reged -x NTUSER.DAT HKEY_CURRENT_USER \\ ntuser.reg
```
This command exports the HKEY_CURRENT_USER hive from NTUSER.DAT and saves it to a new file named ntuser.reg.

With the registry data in a readable format, we then used the grep command to search for keywords related to VNC. But I didn't find any password values related to VNC.

```Bash
    grep -i "hex:" ntuser.reg
```
A separate search for a hexadecimal value in the registry uncovered a key named Password.

The full hexadecimal password was:

```7e,9b,31,12,48,b7,c8,a8```

This value, when combined into a single hex string, is 7e9b311248b7c8a8. It was clear that this was not a plaintext password but a form of encoded or encrypted data.

### Step 2: Formulating the Decryption Hypothesis
TightVNC is known to use the DES (Data Encryption Standard) algorithm with a hardcoded key to encrypt passwords. Initial attempts involved trying the most common DES keys for older versions of TightVNC and similar VNC software.

We used a Python script with the pycryptodome library to perform the decryption, but these attempts failed, resulting in a UnicodeDecodeError or unreadable output. This suggested that the key was incorrect.

### Step 3: Finding the Critical Clue
The key to solving the challenge lay in identifying the exact version of TightVNC that was installed on the system. The challenge description's hint, "I installed every old software known to man," suggested that a specific, possibly obscure version was used.

We used the file command on the provided MSI installer file, tightvnc-2.8.85-gpl-setup-64bit.msi, which revealed the exact version number: 2.8.85.

This piece of information was crucial. A quick search for this confirmed that this version uses a different, but still hardcoded, DES key. The correct key was found to be ```0xE8 0x4A 0xD6 0x60 0xC4 0x72 0x1A 0xE0```

### Step 4: The Final Decryption
With the correct DES key identified, we could now confidently decrypt the password using Python. The following script combines the hexadecimal password with the correct DES key to produce the final, plaintext password.

```Python

from Crypto.Cipher import DES
import binascii

# The hex password from the registry
password_hex = '7e9b311248b7c8a8'

# The hardcoded DES key for TightVNC 2.x
des_key_hex = 'e84ad660c4721ae0'

# Convert hex to binary data
ciphertext = binascii.unhexlify(password_hex)
key = binascii.unhexlify(des_key_hex)

# Create a DES cipher object with ECB mode
cipher = DES.new(key, DES.MODE_ECB)

# Decrypt the ciphertext
decrypted = cipher.decrypt(ciphertext)

# Decode the result, trimming null bytes
password = decrypted.decode('latin-1').rstrip('\x00')

print(password)
```
Running this script will give the final password.

### Conclusion
The flag for this challenge was the decrypted password and input in the ictf{<password>}. The key takeaway from this challenge is that precise information, such as software version numbers, is often critical in forensics.  
