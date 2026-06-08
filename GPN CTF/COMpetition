# COMpetition

**Event:** GPN CTF 2026  
**Category:** Crypto  
**Author:** uxxct  

> This week's special: A chance to compete against our esteemed guest, the rock-paper-scissors grand master. If you manage to beat them 100 out of 100 times we will reward you with a flag specially made for you.

## Challenge Overview

The challenge presents a game of Rock-Paper-Scissors. To win the flag, we must beat the server **100 consecutive times**.

Before each round, we submit a commitment. The server then reveals its move, after which we reveal our choice and provide proof that it matches the commitment.

At first glance, this appears to be a standard commitment scheme designed to prevent us from changing our move after seeing the server's choice.

---

## Source Analysis

The verification logic is:

```python
def verify(commitment: bytes, message: bytes, unveil_info: tuple[bytes, bytes]) -> bool:
    r1, r2 = unveil_info
    return commitment == sha256(r1 + message + r2).digest()
```

The server stores a commitment and later verifies it by hashing:

```python
sha256(r1 + message + r2)
```

The issue is that the verifier only checks whether the reconstructed string hashes to the commitment. It does **not** ensure that the commitment can only be opened to a single message.

---

## The Vulnerability

A secure commitment scheme should be:

* **Hiding** — the committed value cannot be determined before it is revealed.
* **Binding** — the committer cannot reveal the same commitment as multiple different messages.

This implementation is not binding.

Consider the string:

```text
_rock_paper_scissors_0
```

and the commitment:

```python
sha256(b"_rock_paper_scissors_0")
```

The same commitment can be opened as all three possible moves.

### Opening as `rock`

```text
r1 = "_"
message = "rock"
r2 = "_paper_scissors_0"
```

Result:

```text
_ + rock + _paper_scissors_0
=
_rock_paper_scissors_0
```

### Opening as `paper`

```text
r1 = "_rock_"
message = "paper"
r2 = "_scissors_0"
```

Result:

```text
_rock_ + paper + _scissors_0
=
_rock_paper_scissors_0
```

### Opening as `scissors`

```text
r1 = "_rock_paper_"
message = "scissors"
r2 = "_0"
```

Result:

```text
_rock_paper_ + scissors + _0
=
_rock_paper_scissors_0
```

All three openings reconstruct the exact same committed string, meaning they all produce the same SHA-256 hash.

This allows us to decide **after seeing the server's move** which message we want to reveal.

---

## Exploitation Strategy

For every round:

1. Commit to:

```text
_rock_paper_scissors_i
```

where `i` is the round number.

2. Send:

```python
sha256(commitment_string)
```

3. Wait for the server to reveal its move.

4. Open the commitment as the move that beats the server.

| Server Move | Our Move |
| ----------- | -------- |
| Rock        | Paper    |
| Paper       | Scissors |
| Scissors    | Rock     |

The challenge includes a protection against reusing the same commitment for different moves:

```python
elif com in already_seen and already_seen[com] != your_choice:
```

Using the round number as a suffix ensures every commitment is unique.

---

## Solver

```python
from pwn import remote
from hashlib import sha256

HOST = "steamed-souffle-over-minced-green-onions-ehhw.gpn24.ctf.kitctf.de"
PORT = 443

io = remote(HOST, PORT, ssl=True)

for i in range(100):
    io.recvuntil(b"Commitment (hex): ")

    committed = f"_rock_paper_scissors_{i}".encode()
    commitment = sha256(committed).hexdigest()

    io.sendline(commitment.encode())

    server_move = io.recvline().decode()

    if "rock" in server_move:
        move = "paper"
        r1 = b"_rock_"
        r2 = f"_scissors_{i}".encode()

    elif "paper" in server_move:
        move = "scissors"
        r1 = b"_rock_paper_"
        r2 = f"_{i}".encode()

    else:
        move = "rock"
        r1 = b"_"
        r2 = f"_paper_scissors_{i}".encode()

    io.recvuntil(b"What did you choose? ")
    io.sendline(move.encode())

    io.recvuntil(b"Proof (hex): ")
    io.sendline(f"{r1.hex()} {r2.hex()}".encode())

print(io.recvall(timeout=2).decode())
```

---

## Flag

```text
GPNCTF{Wait, iT's NOt ju5t 1uck? N3vEr hAs b3en.}
```
