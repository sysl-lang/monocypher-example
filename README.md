# monocypher-example

**A worked example, not a library.** One [sysl](https://github.com/sysl-lang/sysl) program that depends
on [`sysl-lang/monocypher`](https://github.com/sysl-lang/monocypher) and does the thing you would
actually use it for: two people agree a key over an insecure channel, then send a signed, sealed
message that a third party cannot read, forge or redirect.

It exists to be *run*, and to be the shortest complete answer to two questions — what a sysl project
with a dependency looks like, and what the monocypher binding feels like to use.

## Run it

```
sysl run .
```

That is the whole of it. The dependency is fetched, its vendored C is compiled, and the program is
linked and run. Needs sysl **0.0.7 or newer** — older versions dropped a dependency's `.c` files and
the link failed naming every `crypto_` symbol.

## What you get

```
alice sends her public key:  8520f0098930a754748b7ddcb43ef75a0dbf3a0d26381af4eba4a98eaa9b4e6a
bob sends his:                de9edb7d7b7dc1b4d35b61c2ece435373f8343c85b78674dadfc7e146f882b4f
both reach the same point:    4a5d9d5ba4ce2de1728e3bf480350f25e07e21c947d19e3376f09b3c1e161742
  and they agree:             true
hashed into a real key:       bb16f461d45d47c32a89c90de36d7c7902b314364611d6e4a58247162ec9d4d9

alice's verifying key:        d75a980182b10ab7d54bfed3c964073a0ee172f3daa62325af021a68f707511a
she signs the message:        f36c3384951ea8f7e03d1954fdd5ed61 ...

what goes on the wire:        7c55f817710260d9768f1a16b158e25a4c23e8f2bf921426d3
  with the tag:               8167b4885bc3da86e8a86f375c22e5c8
  and the envelope in clear:  to:bob

bob opens it:                 the tide comes in at four

one bit flipped in the ciphertext and bob rejects it
the envelope rewritten to eve and bob rejects that too

keys wiped:                   0000000000000000000000000000000000000000000000000000000000000000
```

**Every line above is checkable against a specification rather than against this program.** The
secret keys are the ones RFC 7748 §6.1 and RFC 8032 §7.1 print, so the two public keys and the shared
point are the values those documents publish. If your run prints something else, something is wrong
and it is not the test that is wrong.

## What the three files are

```
package.hocon    the dependency, and nothing else
main.sysl        the program
sysl.sum         the digest of what was fetched, written by the first run
```

**A one-file program with no dependencies needs no `package.hocon`.** This one has it solely for the
`dependencies` block — the manifest is optional, and the single-file case is the general case with
nothing filled in.

**`sysl.sum` is committed on purpose.** It pins the exact tree that coordinate resolved to, so a run
here fetches the same bytes this output came from.

## The parts worth reading

- **The shared point is hashed before it is used.** An X25519 output is a curve point and its low bits
  are not uniformly distributed, so `blake2b` over it is what produces a key. Skipping that step is
  the classic mistake and it does not announce itself.
- **The envelope is authenticated but not encrypted.** `to:bob` travels in the clear and is still
  covered by the tag, which is why rewriting it to `to:eve` is caught. Associated data is the part
  people forget is protected.
- **The nonce is fixed here so the output can be checked**, and that is the one thing in this program
  you must not copy. A nonce must never repeat under one key; a real program draws a fresh one per
  message.
- **There is no key generation**, because monocypher supplies no randomness and neither does sysl's
  library. Every secret is an input. On a hosted target read 32 bytes from `/dev/urandom`.
- **A byte slice does not print** — nothing in sysl's library renders `[]u8` — so `text()` at the top
  of `main.sysl` decodes the bytes that are meant to be read as text. Any program using this binding
  needs some version of that.

## Licence

The example is public domain (CC0). It vendors nothing; monocypher carries its own licence.
