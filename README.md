# MAC-Attack

A length-extension attack against a SHA-1 based message authentication code, forging a valid MAC for an extended message without knowing the secret key.

## What this demonstrates

A naive MAC of the form `SHA1(secret || message)` is forgeable. SHA-1 is an iterative Merkle-Damgard hash: its output is its full internal state after the final block. If you know a valid MAC, you know the hash state at that point, so you can seed a SHA-1 instance with that state and keep hashing your own appended data. This forges the MAC for `secret || message || padding || addition` without ever knowing the secret, as long as you reconstruct the padding the original message would have had.

## Concepts demonstrated

- Length-extension weakness of `SHA1(secret || message)` MAC construction
- SHA-1 as a Merkle-Damgard iterated hash where the digest is the internal state
- Seeding the SHA-1 state (`state[0..4]`) from an intercepted MAC to resume hashing
- Reconstructing the message's original glue padding (the `0x80` byte, zero fill, and 64-bit length) so the receiver's recomputation lines up
- Faking the block count via `count[0] = 1024` so the hash behaves as though two 512-bit blocks were already processed
- Why the forged MAC computed on the appended bytes matches the receiver's `SHA1(secret || forged message)`

## What's implemented

- `main.c` is a modified SHA-1 (based on the public clibs/sha1 implementation). `SHA1Init` seeds the state from the intercepted MAC and sets the block count; `main` hashes the appended bytes and prints the forged MAC as hex. The original message, its glue padding, and the test vectors are kept inline as commented reference.

## Stack

- C (SHA-1 core adapted from https://github.com/clibs/sha1)

## Usage

```
gcc -o main main.c
./main
```

The attack values are compiled in, so no input is required. It prints the forged 20-byte MAC as hex.

## Credit

The SHA-1 core is adapted from the public domain implementation at https://github.com/clibs/sha1. The length-extension modifications are the contribution here.
