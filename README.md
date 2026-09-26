# Coattail

**Track records nobody can fake. Then ride the good ones.**

Copy trading has one structural problem: the record is self-reported. Anyone can claim a hit
rate, quietly delete the losses, or show you a screenshot. Coattail removes the claim entirely.
A trader's position is readable straight off their signature, so the record is something you
compute rather than something you are told.

Live: https://coattail.vercel.app

## The mechanism

A Discreet Log Contract binds a payout to an oracle's future signature. Before the oracle has
said anything, both sides can compute the exact curve point that signature will land on, one
point per outcome:

```
S_outcome = R + e(R, P, outcome)·P
```

A trader takes a position by building a Schnorr adaptor signature against one of those points.
The signature is published; the chosen outcome is not.

The property Coattail is built on is that the binding is **recoverable**. An observer holding
only the published signature `(R_a, s_a)`, the trader's key `Y`, and the two candidate points
can test each one:

```
s_a·G  ==  R_a − S_candidate + e·Y
```

Exactly one candidate satisfies it. That is the position, read without the trader saying a word.
Run the page and you can watch three traders take positions privately while the table recovers
all three.

Once the oracle publishes its scalar `s`, `s_a + s` completes the winning side's signature and
the losing side stays locked, because completing it needs a number the oracle never published.
Wins and losses therefore accumulate on their own. Nobody submits a result.

## Running it

Static. No build, no server, no node, no wallet.

```
python3 -m http.server 8000
```

Then open `http://localhost:8000`. Append `?demo` to run three markets and a copied position
automatically.

## What is implemented

BIP-340 oracle attestation with x-only keys and even-y lifting, anticipation point derivation,
Schnorr adaptor signature creation, recovery of the bound outcome by an observer, settlement
against the published scalar, record accumulation, and copying a leader's position onto a new
market.

Not implemented: funding transactions, CET construction, broadcast, and multi-outcome markets.
This proves the mechanism; it does not move coins.

## Verification

The page script itself is driven in Node through a DOM shim, so the tests exercise the shipped
code rather than a reimplementation of it. Twenty-two assertions cover the full lifecycle plus a
negative control over 21 freshly keyed positions:

```
PASS  exactly one point verifies per trader
PASS  oracle scalar matches precomputed point
PASS  every position settled exactly once
PASS  marked leader has max hit rate
PASS  leader's position recovered from signature
PASS  own signature reads back as the same side
PASS  no signature ever verifies against both outcomes   both=0
PASS  no signature ever fails both outcomes              neither=0
```

The last two are the ones that matter. If a signature could verify against both points the read
would be meaningless, and if it could verify against neither the record would have holes.

## Dependencies

`@noble/secp256k1` v2, vendored into the repo so the page has no runtime third-party dependency.
WebCrypto `SHA-256` for the tagged hashes. No framework, no build step.

## Licence

MIT.
