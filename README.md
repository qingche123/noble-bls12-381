# noble-bls12-381

[bls12-381](https://electriccoin.co/blog/new-snark-curve/), a pairing-friendly elliptic curve construction.

This is a [Barreto-Lynn-Scott curve](https://eprint.iacr.org/2002/088) with an embedding degree of 12 & 128-bit security level.

It allows simple construction of threshold signatures, which allows a user to
sign 100 messages with one signature and verify them swiftly in batch.

### This library belongs to *noble* crypto

> **noble-crypto** — high-security, easily auditable set of contained cryptographic libraries and tools.

- No dependencies
- Easily auditable TypeScript/JS code
- Uses es2019 bigint. Supported in Chrome, Firefox, node 10+
- All releases are signed and trusted
- Check out all libraries:
  [secp256k1](https://github.com/paulmillr/noble-secp256k1),
  [ed25519](https://github.com/paulmillr/noble-ed25519),
  [bls12-381](https://github.com/paulmillr/noble-bls12-381),
  [ripemd160](https://github.com/paulmillr/noble-ripemd160),
  [secretbox-aes-gcm](https://github.com/paulmillr/noble-secretbox-aes-gcm)

## Usage

> npm install noble-bls12-381

### Sign a message

```js
import * as bls from "bls12-381";

const DOMAIN = 2;
const PRIVATE_KEY = 0xa665a45920422f9d417e4867ef;
const HASH_MESSAGE = new Uint8Array([99, 100, 101, 102, 103]);

(async () => {
  const publicKey = await bls.getPublicKey(PRIVATE_KEY);
  const signature = await bls.sign(HASH_MESSAGE, PRIVATE_KEY, DOMAIN);
  const isMessageSigned = await bls.verify(HASH_MESSAGE, publicKey, signature, DOMAIN);
})();
```

### Sign 1 message 3 times

```js
import * as bls from "bls12-381";

const DOMAIN = 2;
const PRIVATE_KEYS = [81, 455, 19];
const HASH_MESSAGE = new Uint8Array([99, 100, 101, 102, 103]);

(async () => {
  const publicKeys = await Promise.all(PRIVATE_KEYS.map(bls.getPublicKey));
  const signatures = await Promise.all(PRIVATE_KEYS.map(p => bls.sign(HASH_MESSAGE, p, DOMAIN)));
  const publicKey = await bls.aggregatePublicKeys(publicKeys);
  const signature = await bls.aggregateSignatures(signatures);
  const isMessageSigned = await bls.verify(HASH_MESSAGE, publicKey, signature, DOMAIN);
})();
```

### Sign 3 messages with 3 keys

```js
import * as bls from "bls12-381";

const DOMAIN = 2;
const PRIVATE_KEYS = [81, 455, 19];
const HASH_MESSAGES = ["deadbeaf", "111111", "aaaaaabbbbbb"];

(async () => {
  const publicKeys = await Promise.all(PRIVATE_KEYS.map(bls.getPublicKey));
  const signatures = await Promise.all(PRIVATE_KEYS.map((p, i) => bls.sign(HASH_MESSAGES[i], p, DOMAIN)));
  const signature = await bls.aggregateSignatures(signatures);
  const isMessageSigned = await bls.verifyMultiple(HASH_MESSAGES, publicKeys, signature, DOMAIN);
})();
```

## API

```typescript
function getPublicKey(privateKey: Uint8Array | string | bigint): Uint8Array;
```
- `privateKey: Uint8Array | string | bigint` will be used to generate public key.
  Public key is generated by executing scalar multiplication of a base Point(x, y) by a fixed
  integer. The result is another `Point(x, y)` which we will by default encode to hex Uint8Array.
- Returns `Uint8Array`: endcoded publicKey for signature verification

```typescript
function sign(
  hash: Uint8Array | string,
  privateKey: Uint8Array | string | bigint,
  domain: Uint8Array | string | bigint
): Promise<Uint8Array>;
```
- `hash: Uint8Array | string` - message hash which would be signed
- `privateKey: Uint8Array | string | bigint` - private key which will sign the hash
- `domain: Uint8Array | string | bigint` - version of signature. Different domains will give different signatures. Setting a new domain in an upgraded system prevents it from being affected by the old messages and signatures.
- Returns encoded signature.

```typescript
function verify(
  hash: Uint8Array | string,
  publicKey: Uint8Array | string,
  signature: Uint8Array | string,
  domain: Uint8Array | string | bigint
): Promise<boolean>
```
- `hash: Uint8Array | string` - message hash that needs to be verified
- `publicKey: Uint8Array | string` - e.g. that was generated from `privateKey` by `getPublicKey`
- `signature: Uint8Array | string` - object returned by the `sign` or `aggregateSignatures` function
- Returns `Promise<boolean>`: `Promise<true>` if `signature == hash`; otherwise `Promise<false>`

```typescript
function aggregatePublicKeys(publicKeys: Uint8Array[] | string[]): Uint8Array;
```
- `publicKeys: Uint8Array[] | string[]` - e.g. that have been generated from `privateKey` by `getPublicKey`
- Returns `Uint8Array`: one aggregated public key which calculated from putted public keys

```typescript
function aggregateSignatures(signatures: Uint8Array[] | string[]): Uint8Array;
```
- `signatures: Uint8Array[] | string[]` - e.g. that have been generated by `sign`
- Returns `Uint8Array`: one aggregated signature which calculated from putted signatures

```typescript
function verifyMultiple(
  hashes: Uint8Array[] | string[],
  publicKeys: Uint8Array[] | string[],
  signature: Uint8Array | string,
  domain: Uint8Array | string | bigint
): Promise<boolean>
```
- `hashes: Uint8Array[] | string[]` - messages hashes that needs to be verified
- `publicKeys: Uint8Array[] | string[]` - e.g. that were generated from `privateKeys` by `getPublicKey`
- `signature: Uint8Array | string` - object returned by the `aggregateSignatures` function
- Returns `Promise<boolean>`: `Promise<true>` if `signature == hashes`; otherwise `Promise<false>`

The library also exports helpers:

```typescript
// 𝔽p
bls.P // 0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaabn

// Prime order
bls.PRIME_ORDER // 0x73eda753299d7d483339d80809a1d80553bda402fffe5bfeffffffff00000001n
```

## Curve Description

BLS12-381 is a pairing-friendly elliptic curve construction from the [BLS family](https://eprint.iacr.org/2002/088), with embedding degree 12. It is built over a 381-bit prime field `GF(p)` with...

* z = `-0xd201000000010000`
* p = (z - 1)<sup>2</sup> ((z<sup>4</sup> - z<sup>2</sup> + 1) / 3) + z
	* = `0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaab`
* q = z<sup>4</sup> - z<sup>2</sup> + 1
	* = `0x73eda753299d7d483339d80809a1d80553bda402fffe5bfeffffffff00000001`

... yielding two **source groups** G<sub>1</sub> and G<sub>2</sub>, each of 255-bit prime order `q`, such that an efficiently computable non-degenerate bilinear pairing function `e` exists into a third **target group** G<sub>T</sub>. Specifically, G<sub>1</sub> is the `q`-order subgroup of E(F<sub>p</sub>) : y^2 = x^3 + 4 and G<sub>2</sub> is the `q`-order subgroup of E'(F<sub>p<sup>2</sup></sub>) : y<sup>2</sup> = x<sup>3</sup> + 4(u + 1) where the extention field F<sub>p<sup>2</sup></sub> is defined as F<sub>p</sub>(u) / (u<sup>2</sup> + 1).

BLS12-381 is chosen so that `z` has small Hamming weight (to improve pairing performance) and also so that `GF(q)` has a large 2<sup>32</sup> primitive root of unity for performing radix-2 fast Fourier transforms for efficient multi-point evaluation and interpolation. It is also chosen so that it exists in a particularly efficient and rigid subfamily of BLS12 curves.

## Security

Noble is production-ready & secure. Our goal is to have it audited by a good security expert.

We're using built-in JS `BigInt`, which is "unsuitable for use in cryptography" as [per official spec](https://github.com/tc39/proposal-bigint#cryptography). This means that the lib is vulnerable to [timing attacks](https://en.wikipedia.org/wiki/Timing_attack). But:

1. JIT-compiler and Garbage Collector make "constant time" extremely hard to achieve in a scripting language.
2. Which means *any other JS library doesn't use constant-time bigints*. Including bn.js or anything else. Even statically typed Rust, a language without GC, [makes it harder to achieve constant-time](https://www.chosenplaintext.ca/open-source/rust-timing-shield/security) for some cases.
3. Overall they are quite rare; for our particular usage they're unimportant. If your goal is absolute security, don't use any JS lib — including bindings to native ones. Try LibreSSL & similar low-level libraries & languages.
4. We however consider infrastructure attacks like rogue NPM modules very important; that's why it's crucial to minimize the amount of 3rd-party dependencies & native bindings. If your app uses 500 dependencies, any dep could get hacked and you'll be downloading rootkits with every `npm install`. Our goal is to minimize this attack vector.

## License

MIT (c) Paul Miller (https://paulmillr.com), see LICENSE file.
