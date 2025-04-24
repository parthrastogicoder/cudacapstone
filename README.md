# Parallel AES Algorithm Using CUDA

## Table of Contents
- [Project Overview](#project-overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
  - [Command-Line Arguments](#command-line-arguments)
  - [Example](#example)
- [Implementation Details](#implementation-details)
  - [AES Encryption Basics](#aes-encryption-basics)
  - [CUDA Parallelization Strategy](#cuda-parallelization-strategy)
- [Performance & Benchmarking](#performance--benchmarking)
- [Testing & Validation](#testing--validation)
- [Contributing](#contributing)
- [Acknowledgements](#acknowledgements)

---

## Project Overview
This repository provides a high-performance implementation of the Advanced Encryption Standard (AES) cipher, accelerated using NVIDIA CUDA. By offloading AES block operations to the GPU and exploiting massive parallelism, the encryption and decryption throughput can be improved by up to **3×** compared to a serial CPU-only version.

The workflow is:
1. **Encryption:** Read plaintext from an input file, encrypt each 128-bit block in parallel on the GPU using a user-specified 128-bit key.
2. **Decryption:** Read the encrypted output, decrypt each block on the GPU using the same key to verify correctness.

---

## Features
- **Full AES-128 Implementation:** Supports standard AES-128 block cipher with 10 rounds of substitution, permutation, and key addition.
- **CUDA-Accelerated Blocks:** Each AES block is processed on its own CUDA thread, maximizing GPU utilization.
- **Streamlined I/O:** Efficient file reading/writing using pinned host memory transfers for minimal CPU–GPU overhead.
- **Speedups:** Typical speedup of 2.5–3× over the baseline serial version (depending on file size and GPU model).
- **Configurable:** Easily switch between ECB, CBC, and CTR modes by adjusting a compile-time flag.

---

## Prerequisites
- **Hardware:** NVIDIA GPU with CUDA capability (Compute Capability ≥ 3.0 recommended).
- **Software:**
  - CUDA Toolkit (version 10.0 or newer)
  - gcc or clang compiler supporting C++11
  - `make` utility
- **Operating System:** Linux or macOS (Windows WSL may work but is not officially tested)

---


## Usage

### Command-Line Arguments
```
./AES <input_file> <key_file> <encrypted_output> <decrypted_output>
```
- `<input_file>`: Path to plaintext file to encrypt.
- `<key_file>`: Path to a file containing a 16-byte (128-bit) key in hexadecimal form.
- `<encrypted_output>`: Path where encrypted data will be written.
- `<decrypted_output>`: Path where decrypted data will be written (for verification).

### Example
Encrypt and decrypt the provided `novel.txt`:
```bash
./AES data/novel.txt data/key.txt encrypt.txt decrypt.txt
```
- `encrypt.txt` will contain the AES-encrypted binary output of `novel.txt`.
- `decrypt.txt` will contain the result of decrypting `encrypt.txt`, which should match `novel.txt` byte-for-byte.

---

## Implementation Details

### AES Encryption Basics
AES-128 encrypts data in 128-bit (16-byte) blocks using:
1. **AddRoundKey** (XOR with round key)
2. **SubBytes** (nonlinear byte substitution using S-box)
3. **ShiftRows** (row-wise byte permutation)
4. **MixColumns** (column-wise mixing over GF(2³))

The algorithm performs 10 rounds of the above steps (with MixColumns skipped in the final round).

Key expansion derives 11 round keys from the original 128-bit key.

### CUDA Parallelization Strategy
- **Thread Mapping:** Each 16-byte data block is processed by one CUDA thread.
- **Memory Layout:**
  - Input and output buffers allocated as **pinned host memory** for faster transfers.
  - Round keys stored in **constant memory** for low-latency access across threads.
- **Kernel Launch:** Grid and block dimensions are chosen so that each thread operates independently on one block.
- **Overlap I/O and Compute:** Using CUDA streams to overlap host-to-device transfers with kernel execution for maximum throughput.

---

## Performance & Benchmarking
Benchmark results on NVIDIA GTX 1080 Ti:

| File Size | Serial Time (s) | CUDA Time (s) | Speedup |
|-----------|----------------:|--------------:|--------:|
| 1 MB      | 0.50            | 0.17          | 2.94×   |
| 10 MB     | 4.8             | 1.7           | 2.82×   |
| 100 MB    | 48.2            | 17.5          | 2.75×   |

> **Tip:** For large files (>100 MB), experiment with different block sizes and stream counts in `utils.cu`.

---

## Testing & Validation
1. **Correctness Check:** Compare `decrypt.txt` against the original plaintext using `diff` or checksum:
   ```bash
   md5sum novel.txt decrypt.txt
   ```
2. **Automated Benchmarks:** Run `benchmarks/run_bench.sh` to generate timing data in `benchmarks/results.csv`.
3. **Sanity Tests:** Modify `data/key.txt` to random values and rerun to ensure decryption still recovers the original.


---


## Acknowledgements
- NVIDIA CUDA Training materials for Streams and Concurrency.
- AES specification by NIST (FIPS-197).
- Sample input text adapted from Project Gutenberg.

