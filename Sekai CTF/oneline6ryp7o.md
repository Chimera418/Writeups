# oneline6ryp7o

**Category:** Cryptography 
**Author:** Neobeo

## Challenge

```python
assert __import__('re').match('SEKAI{[67]{67}}$', flag := input()) and \
       not int.from_bytes(flag.encode()) % ~(6 + ~7) ** 67
```

---

## Analysis

Despite being only a single line of Python, the challenge hides a neat mathematical trick behind a few pieces of Python syntax.

The assertion has two independent conditions:

1. The input must match the required flag format.
2. The resulting integer must satisfy a modular arithmetic constraint.

### Step 1 — Flag format

The regular expression

```python
re.match('SEKAI{[67]{67}}$', flag)
```

requires the flag to be of the form

```text
SEKAI{xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx}
```

where every `x` is either `'6'` or `'7'`.

This leaves exactly **67 unknown characters**.

---

### Step 2 — Simplifying the modulus

The second half of the assertion is

```python
not int.from_bytes(flag.encode()) % ~(6 + ~7) ** 67
```

The confusing part is the expression

```python
~(6 + ~7) ** 67
```

Recall that Python's bitwise NOT satisfies

```python
~x = -x - 1
```

Therefore,

```python
~7
= -8
```

which gives

```python
6 + ~7
= 6 - 8
= -2
```

Now,

```python
(-2) ** 67
```

is negative because the exponent is odd,

```python
(-2) ** 67 = -(2 ** 67)
```

Applying the bitwise NOT once more,

```python
~(-(2 ** 67))
= 2 ** 67 - 1
```

So the assertion becomes

```python
int.from_bytes(flag.encode()) % (2 ** 67 - 1) == 0
```

---

### Step 3 — Understanding the integer

`int.from_bytes(flag.encode())` interprets the entire flag as one large big-endian integer.

Every unknown character contributes either

* `'6'` → ASCII 54
* `'7'` → ASCII 55

Changing a character from `'6'` to `'7'` increases the integer by exactly one copy of that byte's positional weight.

---

### Step 4 — The mathematical trick

The modulus is the Mersenne number

[
2^{67}-1.
]

Since every byte contributes a power of

[
256 = 2^8,
]

its positional weight modulo (2^{67}-1) is

[
2^{8k}.
]

Because

[
\gcd(8,67)=1,
]

the sequence

[
2^0,2^8,2^{16},\ldots
]

cycles through **every power of two modulo (2^{67}-1)** exactly once.

This means each of the 67 unknown positions corresponds to one unique bit of a 67-bit number.

If we begin with all characters set to `'6'`, the remaining value needed to make the integer divisible by (2^{67}-1) is simply a 67-bit mask. Every set bit indicates that the corresponding character should be changed from `'6'` to `'7'`.

---

## Solver

```python
MOD = (1 << 67) - 1

prefix = b"SEKAI{"
suffix = b"}"

length = len(prefix) + 67 + len(suffix)

weights = [
    pow(256, length - 1 - (len(prefix) + i), MOD)
    for i in range(67)
]

# Contribution of the fixed characters and assuming every unknown is '6'
const = 0
for b in prefix + b"6" * 67 + suffix:
    const = (const * 256 + b) % MOD

target = (-const) % MOD

flag = ["6"] * 67

for i, w in enumerate(weights):
    if target & w:
        flag[i] = "7"

print("SEKAI{" + "".join(flag) + "}")
```

---

## Flag

```text
SEKAI{6777676667666666677676776776777766777777777776777767777776677666666}
```

---

## Takeaways

* `~x` in Python is equivalent to `-x - 1`.
* `int.from_bytes()` treats the entire byte string as one large integer.
* Choosing the modulus (2^{67}-1) is the key idea: since (256 = 2^8) and (\gcd(8,67)=1), the byte weights become a permutation of the 67 individual bits, turning what initially appears to be a difficult modular arithmetic problem into reconstructing a single 67-bit mask.
