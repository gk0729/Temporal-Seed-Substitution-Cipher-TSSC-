# Temporal-Seed-Substitution-Cipher-TSSC-

一個設計簡單、實用的文檔內容加密 / 解密方式！

**Temporal Seed Substitution Cipher (TSSC)**

---

## Overview

Temporal Seed Substitution Cipher (TSSC) is a lightweight, practical encryption and obfuscation scheme.  
It uses the **file timestamp** (to the minute or second) as a random seed for generating a **unique per-file codebook (substitution table)** that reorders the mapping between characters and cipher codes.

Every document can have a distinct, unpredictable code mapping — even with the same content.

TSSC is especially well-suited for:

- Knowledge compression, local obfuscation, or lossless encryption scenarios  
- Chinese character code-tables, custom input methods, or AI-friendly encodings  
- Scenarios where traditional cryptography (AES, RSA) is overkill but practical security and combinatorial resistance are needed

---

## How TSSC Works

### 1. Seed Generation

- Upon saving a file, extract a timestamp string (e.g. `"2025-11-26 21:17"`), accurate to the minute or second.  
- This timestamp is converted (or hashed) into an integer as the **seed for a PRNG** (pseudo-random number generator).

### 2. Dynamic Codebook Shuffling

1. Define a base codemap  
   - Example: a mapping of all 4808 common traditional Chinese characters to code strings, or any Unicode / binary set.
2. Shuffle the **values** in the codebook using the PRNG seeded by the timestamp.
3. This yields a unique mapping for each timestamp, impossible to reconstruct without knowing the seed.

### 3. Encryption and Decryption

- **Encryption**  
  For each character in the plaintext, substitute with its shuffled code in the per-file codebook.

- **Decryption**  
  Regenerate the exact codebook using the stored / extracted timestamp, then reverse the mapping.

---

## Minimal Python Example

```python
import random

def create_tssc_codemap(base_codemap: dict, file_timestamp: str) -> dict:
    """
    Use a timestamp in 'YYYY-MM-DD HH:MM' (or 'YYYY-MM-DD HH:MM:SS') format
    to generate a deterministic shuffled codemap.
    """
    # Remove '-', ' ', ':' and convert to integer as seed
    seed = int(
        file_timestamp
        .replace("-", "")
        .replace(" ", "")
        .replace(":", "")
    )
    rng = random.Random(seed)

    keys = list(base_codemap.keys())
    values = list(base_codemap.values())

    rng.shuffle(values)
    return dict(zip(keys, values))


# Usage
base_codemap = {
    "你": "n",
    "好": "h",
    "世": "s",
    "界": "j",
}  # Example — in practice, for 4808+ chars

timestamp = "2025-11-26 21:17"
tssc_codemap = create_tssc_codemap(base_codemap, timestamp)

# To decrypt, recreate the same codemap with the *same* timestamp!
```

**Important:**

- The timestamp must be **reliably stored or extractable**.  
- The base codemap (source codes) should remain **private** for better security.

---

## Security Model

If the timestamp is not known or predictable, brute-forcing all possible timestamps (even just **up to the minute**) faces:

- 500,000+ combinations per year  
- multiplied by the codebook complexity

This quickly makes frequency analysis and many statistical attacks infeasible.

You can further strengthen / obfuscate the scheme by:

- Leveraging **nanosecond** precision  
- Storing **fake multiple file timestamps** or redundant metadata  
- Mixing in additional entropy (e.g. file name hash, machine ID, etc.)

The codemap can also be expanded to **1024 or more possible symbols**, removing traditional keyboard restrictions and enabling richer encodings.

---

## Advantages

- Resistant to **frequency analysis** (codebook mapping adapts per file instance)  
- Open-source implementation, but the **private base codemap / table is never published**  
- Lightweight, no external libraries required beyond the Python standard library  
- Usable as a core engine for:
  - Multi-layer semantic encoding  
  - Knowledge compression  
  - Lightweight file obfuscation

---

## Suggested File Structure

```text
TSSC/
├─ README.md
├─ tssc_cipher.py      # Main implementation (example as above)
├─ test_example.py     # Demo with sample codemap and timestamp
├─ LICENSE
└─ .gitignore
```

---

## Philosophy

> "When I was young, skipping through a dense cryptography textbook in the library,  
> I realized: effective security doesn't need complex math—it needs thoughtful design  
> that makes attackers' effort >> reward."
>
> This project embodies that principle: simple enough for anyone to understand,  
> hard enough for no one to break.

「安全性不一定要依賴複雜數學；  
只要把『時間』與『結構設計』用得巧妙，就能讓攻擊者的成本遠大於他們能拿到的利益。」

本專案是為了那些重視實用性與穩健設計的：

- 架構師（Architects）  
- 工程師（Engineers）  
- 研究員與創客（Tinkerers）

歡迎：

- 提出 Issue / PR 討論理論與實作細節  
- 分享改良版的 codemap 設計、組合方案或實務應用  
- 將 TSSC 組合到更大型的安全 / 壓縮 / 知識編碼系統中

For full theory, see the philosophy above or submit issues / PRs for deeper discussion.  
Contributions, improvements, and research on further composability are welcome.
「Licensed under the MIT License」。
