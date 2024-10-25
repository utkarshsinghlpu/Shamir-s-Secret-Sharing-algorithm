# shamir-secret-sharing
Simple, independently audited, zero-dependency TypeScript implementation of Shamir's Secret Sharing algorithm.

Uses GF(2^8). Works on Uint8Array objects. Implementation inspired by hashicorp/vault.

Both Node and browser environments are supported.

Made with ❤️ by Privy.

Security considerations
This library has been independently audited by Cure53 (audit report) and Zellic (audit report).

There are a couple of considerations for proper use of this library.

Resistance to side channel attacks: JavaScript is a garbage-collected, just-in-time compiled language and it is thus unrealistic to achieve true constant-time guarantees. Where possible, we aim to achieve algorithmic constant-time.
This library is not responsible for verifying the result of share reconstruction. Incorrect or corrupted shares will produce an incorrect value. Thus, it is the responsibility of users of this library to verify the integrity of the reconstructed secret.
Secrets should ideally be uniformly distributed at random. If this is not the case, it is recommended to first encrypt the value and split the encryption key.
Usage
We can split a secret into shares and later combine the shares to reconstruct the secret.

import {split, combine} from 'shamir-secret-sharing';

const toUint8Array = (data: string) => new TextEncoder().encode(data);

// Example of splitting user input
const input = document.querySelector("input#secret").value.normalize('NFKC');
const secret = toUint8Array(input);
const [share1, share2, share3] = await split(secret, 3, 2);
const reconstructed = await combine([share1, share3]);
console.log(btoa(reconstructed) === btoa(secret)); // true

// Example of splitting random entropy
const randomEntropy = crypto.getRandomValues(new Uint8Array(16));
const [share1, share2, share3] = await split(randomEntropy, 3, 2);
const reconstructed = await combine([share2, share3]);
console.log(btoa(reconstructed) === btoa(randomEntropy)); // true

// Example of splitting symmetric key
const key = await crypto.subtle.generateKey(
  {
    name: "AES-GCM",
    length: 256
  },
  true,
  ["encrypt", "decrypt"]
);
const exportedKeyBuffer = await crypto.subtle.exportKey('raw', key);
const exportedKey = new Uint8Array(exportedKeyBuffer);
const [share1, share2, share3] = await split(exportedKey, 3, 2);
const reconstructed = await combine([share2, share1]);
console.log(btoa(reconstructed) === btoa(exportedKey)); // true
API
This package exposes two functions: split and combine.

split
/**
 * Splits a `secret` into `shares` number of shares, requiring `threshold` of them to reconstruct `secret`.
 *
 * @param secret The secret value to split into shares.
 * @param shares The total number of shares to split `secret` into. Must be at least 2 and at most 255.
 * @param threshold The minimum number of shares required to reconstruct `secret`. Must be at least 2 and at most 255.
 * @returns A list of `shares` shares.
 */
declare function split(secret: Uint8Array, shares: number, threshold: number): Promise<Uint8Array[]>;
combine
/**
 * Combines `shares` to reconstruct the secret.
 *
 * @param shares A list of shares to reconstruct the secret from. Must be at least 2 and at most 255.
 * @returns The reconstructed secret.
 */
declare function combine(shares: Uint8Array[]): Promise<Uint8Array>;



