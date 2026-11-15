---

# Sygil.fun - CTF Write-up

## Challenge Overview

* **Platform:** Sygil.fun


* **Category:** Reverse Engineering
* **Objective:** Authenticate successfully by bypassing the custom cryptographic/hashing check and generating a valid validation token.

---

## Analysis & Methodology

### 1. Disassembly & Logic Review

By examining the binary using a disassembler/decompiler, the validation mechanism was identified:

* **Input Validation:** The program checks the length of the provided alias (`true name`), requiring it to be greater than 3 characters (`3 < sVar6`).
* **Hashing Algorithm:** The core logic implements a custom variant of the **FNV-1a hash algorithm**, starting with the initial constant `0x811c9dc5` and iteratively applying XOR operations with each character's ASCII value followed by multiplication with the magic constant `0x1000193`.


* **Bit Folding:** To compress the 32-bit hash value into a 16-bit segment, a right shift (`>> 0x10`) is performed to align the upper 16 bits with the lower 16 bits, combined via an `XOR` operation with `uVar9 & 0xffff`.


* **Token Formatting:** The final token structure follows the format `syg-XXXXXXXX-XXXX-XXXXXXXX` utilizing specific XOR masks (`0x5947494c` and `0x535947`).



### 2. Solution Script

To automate the token generation for any given alias, the following Python script was developed based on the reverse-engineered logic:

```python
def generate_token(alias):
    # 1. Initial FNV-1a Hash value
    uVar9 = 0x811c9dc5
    
    # 2. Iterate through alias characters (matching the C while loop)
    for char in alias:
        uVar7 = ord(char)
        if uVar7 == 0xd:  # Carriage return
            break
        uVar9 = (uVar9 ^ uVar7) * 0x1000193
        uVar9 &= 0xFFFFFFFF

    # 3. Folding operation for upper and lower 16 bits
    high_16 = uVar9 >> 0x10
    low_16 = uVar9 & 0xffff
    uVar7_folded = high_16 ^ low_16

    # 4. Final formatting via sprintf equivalents
    part1 = uVar9 ^ 0x5947494c
    part2 = uVar7_folded
    part3 = uVar9 ^ 0x5947494c ^ uVar7_folded ^ 0x535947

    part1 &= 0xFFFFFFFF
    part2 &= 0xFFFF
    part3 &= 0xFFFFFFFF

    token = f"syg-{part1:08x}-{part2:04x}-{part3:08x}"
    return token

# Generate token for the target alias
my_alias = "chabouh"
print(f"The Token for '{my_alias}' is: {generate_token(my_alias)}")

```

---

## Challenges & Obstacles Faced

* **Navigating Complex Assembly Logic:** Understanding how the binary performs low-level bit manipulations—specifically the right-shift (`>> 0x10`) combined with bit-folding (`XOR` against the lower 16 bits)—required deep focus and breaking down the mathematical purpose of the operations.


* **Mental Fatigue & Persistence:** Sticking with the challenge over a multi-day period (3 days of continuous analysis) tested problem-solving endurance, especially when dealing with unfamiliar custom validation structures instead of standard static passwords.


---

## Result

After properly mapping the input and utilizing the automated script, the system authenticated successfully, displaying the **"CURSE SEALED"** status and the pentagram seal.

---
