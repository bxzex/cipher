# Cipher

AES, GCM, SHA-256 and HMAC written from scratch in JavaScript. You can step
through every AES round, and every output is checked against the published
test vectors and against the browser's own WebCrypto.

Live: https://bxzex.github.io/cipher/

## What is implemented

- **AES-128, AES-192 and AES-256** (FIPS 197): key expansion, SubBytes,
  ShiftRows, MixColumns and AddRoundKey, plus the full inverse cipher. The
  S-box is not pasted in. It is generated at load from the field arithmetic:
  a walk over GF(2^8) by powers of 3 gives each byte's multiplicative inverse,
  followed by the affine map.
- **Modes**: ECB and CBC with PKCS#7 padding, CTR with a full 128-bit counter,
  and **GCM** (SP 800-38D) with a 96-bit nonce, additional authenticated data
  and a 128-bit tag. GHASH multiplies in GF(2^128) using the spec's
  bit-reflected algorithm on BigInts. Decryption checks the tag before it
  returns any plaintext.
- **SHA-256** (FIPS 180-4), including the 64-bit length field for long inputs
- **HMAC-SHA256** (RFC 2104), hashing keys longer than the block size first

## What the page shows

- **Inside block 0.** Pick any round and see the 4×4 state after each step,
  coloured by byte value, with a dot on every byte that step changed. The key
  schedule sits alongside with the current round key lit.
- **Diffusion.** The block is encrypted again with one input bit flipped, and
  the page counts the state bytes that differ after each round: 1 after the
  first SubBytes, 4 after round 1 and all 16 by round 2.
- **Avalanche.** Flip any single bit of a SHA-256 input and watch roughly half
  of the 256 output bits change.
- **Message schedule and compression.** W0 to W63 and the working variables
  after all 64 rounds.

## Verification

The Verify tab runs two kinds of check on load.

**Published vectors, 13 of 13.** The FIPS 197 appendix C examples for all
three key sizes, forwards and inverse. GCM test cases 1 and 2 from the original
GCM specification. The FIPS 180-2 SHA-256 examples, including the two-block
448-bit message and one million repetitions of `a`. RFC 4231 HMAC cases 1
and 2.

**Differential testing, 1,050 of 1,050.** Random keys, IVs, AAD and message
lengths from 0 to 400 bytes, 150 cases per primitive, each computed here and by
`crypto.subtle` and compared byte for byte:

| Primitive | Check |
|---|---|
| AES-128-CBC, AES-256-CBC | ciphertext matches WebCrypto, decryption round trips |
| AES-128-CTR | matches WebCrypto with `length: 128`. The counters start near a 64-bit boundary, so the carry into the upper half is exercised on every case. |
| AES-256-GCM | ciphertext and tag match WebCrypto, decryption round trips |
| AES-128-GCM tamper | one random bit flipped in the ciphertext or tag must make decryption throw |
| SHA-256, HMAC-SHA256 | match WebCrypto |

The whole run takes about 160 ms.

One mistake was caught this way. While building the vector table I copied the
RFC 4231 case 1 tag with a character missing. The table showed the mismatch
immediately, and Python's `hmac` confirmed that the implementation was right
and the transcription was wrong.

## Notes

This is for learning. The code is not constant time. The S-box lookups and
BigInt field multiplication leak timing, so use WebCrypto for anything real.
Chrome's WebCrypto has no AES-192 and no ECB, so those two are checked against
the FIPS vectors only.

One HTML file. No libraries, no build step.

Built by [bxzex](https://bxzex.com).
