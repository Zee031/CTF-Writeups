---
description: 'Challenge author: etern1ty'
---

# pad

The challenge runs on an instance and its script is given

<pre class="language-python"><code class="lang-python">from Crypto.Util.Padding import unpad
from Crypto.Cipher import AES
from os import urandom
from random import SystemRandom

with open("flag.txt", "r") as f:
    FLAG = f.read().strip()

rng = SystemRandom()

class Challenge:
    def __init__(self):
        self.msg = urandom(16).hex()
        self.key = urandom(16)
        self.q = 0
        self.L = 15000

    def inc(self):
        self.q += 1
        return self.q >= self.L

<strong>    def encrypt(self):
</strong>        iv = urandom(16)
        c  = AES.new(self.key, AES.MODE_CBC, iv=iv).encrypt(self.msg.encode("ascii"))
        return (iv + c).hex()

    def unpad_oracle(self, xhex: str):
        try:
            if len(xhex) % 2 != 0:
                return 0
            b = bytes.fromhex(xhex)
            if len(b) &#x3C; 32 or len(b) % 16 != 0:
                return 0
            iv, ct = b[:16], b[16:]
            pt = AES.new(self.key, AES.MODE_CBC, iv=iv).decrypt(ct)
            try:
                unpad(pt, 16)
                good = True
            except ValueError:
                good = False
            noisy = good ^ (rng.random() > 0.2)
            return 1 if noisy else 0
        except Exception:
            return 0

    def check(self, s: str):
        return s == self.msg

def main():
    ch = Challenge()
    while True:
        if ch.q >= ch.L:
            print("bye :(")
            break
        print("\n:3\n")
        print("1 - Encrypt fresh IV-CT for the secret message")
        print("2 - Padding check for supplied (IV-CT) hex")
        print("3 - Submit recovered message")
        print("4 - Quit")
        choice = input("Choice: ").strip()

        if choice == "1":
            ct = ch.encrypt()
            quit_now = ch.inc()
            print(f"ct = {ct}")
            if quit_now:
                print("bye :(")
                break

        elif choice == "2":
            x = input("ct = ").strip()
            r = ch.unpad_oracle(x)
            quit_now = ch.inc()
            print(f"pad = {r}")
            if quit_now:
                print("bye :(")
                break

        elif choice == "3":
            m = input("msg = ").strip()
            quit_now = ch.inc()
            if ch.check(m):
                print(f"flag = {FLAG}")
            else:
                print("whuh")
            if quit_now:
                print("bye :(")
                break

        elif choice == "4":
            print("bye :(")
            break

        else:
            print("what")

if __name__ == "__main__":
    main()

</code></pre>

It randomly generates a 16 bytes secret message in ASCII, which is encoded into hex, resulting in a 32 character ascii hex string that is stored.\
The AES key (which is 16 bytes) is also randomly generated once for a session.\
The oracle query is also limited to 15000.

For the encryption, it uses the CBC mode to encrypt the secret message which is two AES blocks \
When the instance is run, the secret message and key are generated.\
The query count is incremented every time option 1, 2, or 3 is sent and will terminate the program when it reaches higher than 15000.

If option 1 is sent, the secret message is encrypted with the key and the randomly generated IV (generated every time option 1 is sent). Afterwards, it prints out IV||C1||C2 (C1 and C2 being the 1st and 2nd block of the encrypted message as the whole message is 2 AES blocks (16 bytes) long)

If option 2 is sent, the instance takes an input of IV || CT which will return 0 if the length is not even or if the bytes.fromhex() decoding length is less than 32 bytes or not divisible by 16. The input is then split into the IV and the CT which will then be used alongside the generated key to decrypt the CT. Afterwards, the decrypted CT is then  checked for padding which will return 0 if no valid padding is detected and will return 1 if otherwise. The value of the padding checked is then XOR'd with rng.random() > 0.2 which gives an 80% chance of the padding validity be inverted.

If option 3 is sent, the instance will take the input and match it with the secret message and if it returns true, it will print the flag, otherwise, it will print "whuh?"

```python
import socket, time, random
from binascii import hexlify, unhexlify

HOST = "intersec.hcs-team.com"
PORT = 10092

ASCII_HEX = b"0123456789abcdef"
BASE_INTERVAL = 0.012        # pacing + jitter
GOODBYE = b"bye :("
QUIET = False
HEARTBEAT = False

REPEATS_PER_CANDIDATE = 14   # ~7k queries per run

# ---------- I/O helpers ----------
def sendline(sock, s: str):
    sock.sendall(s.encode() + b"\n")

def recv_until(sock, marker: bytes) -> bytes:
    data = b""
    while marker not in data and GOODBYE not in data:
        chunk = sock.recv(1024)
        if not chunk:
            raise ConnectionError("Server closed unexpectedly")
        data += chunk
    if GOODBYE in data:
        raise ConnectionError("Goodbye from server (cap/forced drop)")
    return data

def read_block(sock) -> str:
    return recv_until(sock, b"Choice:").decode(errors="ignore")

# ---------- Oracle ----------
_oracle_queries = 0

def oracle(sock, forged: bytes, repeats=REPEATS_PER_CANDIDATE) -> int:
    global _oracle_queries
    xhex = hexlify(forged).decode()
    ok_count = 0
    for _ in range(repeats):
        sendline(sock, "2")
        _ = recv_until(sock, b"ct = ")
        sendline(sock, xhex)
        block = read_block(sock)
        if "pad = 0" in block:   # polarity: 0 = valid
            ok_count += 1
        _oracle_queries += 1
        if HEARTBEAT:
            print(".", end="", flush=True)
        time.sleep(BASE_INTERVAL + random.uniform(-0.003, 0.003))
    return ok_count

# ---------- Scoring ----------
def score_heavy7000(sock, prev0: bytes, prev: bytearray, padlen: int, target_block: bytes, i: int):
    scores = {}
    for g in ASCII_HEX:
        prev[i] = prev0[i] ^ g ^ padlen
        scores[g] = oracle(sock, bytes(prev) + target_block)
    return max(scores.items(), key=lambda kv: kv[1])[0]

# ---------- Block recovery ----------
def recover_block(sock, prev_block: bytes, target_block: bytes, label: str) -> bytes:
    prev0 = bytes(prev_block)
    prev  = bytearray(prev_block)
    known = bytearray(16)

    if not QUIET:
        print(f"[*] Recovering {label} ...")
    for i in range(15, -1, -1):
        padlen = 16 - i
        for j in range(15, i, -1):
            prev[j] = prev0[j] ^ known[j] ^ padlen

        best_g = score_heavy7000(sock, prev0, prev, padlen, target_block, i)
        known[i] = best_g
        prev[i] = prev0[i] ^ best_g ^ padlen
        if not QUIET:
            print(f"[+] byte {i:02d}: {chr(best_g)}")
    return bytes(known)

# ---------- One full attempt ----------
def one_run() -> bool:
    global _oracle_queries
    _oracle_queries = 0
    t0 = time.time()

    sock = socket.create_connection((HOST, PORT), timeout=8.0)
    sock.settimeout(8.0)
    _ = read_block(sock)

    sendline(sock, "1")
    block = read_block(sock)
    ct_line = next(ln for ln in block.splitlines() if ln.startswith("ct ="))
    ct_hex = ct_line.split("=", 1)[1].strip()
    full = unhexlify(ct_hex)
    IV, C1, C2 = full[:16], full[16:32], full[32:48]
    print(f"[i] Got ciphertext: {ct_hex}")

    P2 = recover_block(sock, C1, C2, "P2")
    P1 = recover_block(sock, IV, C1, "P1")

    msg = (P1 + P2).decode("ascii")
    print("\n[***] Recovered msg:", msg)

    if not (len(msg) == 32 and all(c in "0123456789abcdef" for c in msg)):
        print("[!] Sanity check failed: not 32 lower-hex chars.")
        return False

    sendline(sock, "3")
    _ = recv_until(sock, b"msg = ")
    sendline(sock, msg)
    block = read_block(sock)

    elapsed = time.time() - t0
    print(f"[i] Attempt stats: {elapsed:.1f}s, {_oracle_queries} oracle queries")

    for ln in block.splitlines():
        if ln.startswith("flag ="):
            print(ln)
            return True
        if ln.strip() == "whuh":
            print("[!] Submission rejected (noise).")
            return False
    return False

# ---------- Retry loop ----------
def main():
    attempt = 1
    while True:
        try:
            print(f"\n=== Attempt {attempt} ===")
            if one_run():
                break
            attempt += 1
            time.sleep(0.3)
        except ConnectionError as e:
            print(f"[!] ConnectionError: {e}")
            attempt += 1
            time.sleep(0.3)
        except Exception as e:
            print(f"[!] Unexpected error: {e}")
            break

if __name__ == "__main__":
    main()

```

The solver first acquires the IV and encrypted blocks C1 and C2. First the solver recovers P2 by using a padding oracle attack which is done by forging the previous ciphertext block and testing the byte values one at a time through option 2 of the instance. The process is then repeated on the IV and P1 to acquire the P1. P1 and P2 are then concatenated to be submitted through option 3 of the instance to acquire the flag.
