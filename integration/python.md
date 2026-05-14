---
title: Python SDK
sidebar:
  label: Python
  order: 4
---

The official Python SDK for Tx3 is published on PyPI as [`tx3-sdk`](https://pypi.org/project/tx3-sdk/). It loads a compiled `.tii` protocol, binds parties and signers, and drives the full transaction lifecycle (`resolve -> sign -> submit -> wait`) through TRP.

The source lives at [tx3-lang/python-sdk](https://github.com/tx3-lang/python-sdk).

## Installation

```bash
pip install tx3-sdk
```

## Quick start

```python
import asyncio

from tx3_sdk import CardanoSigner, Party, PollConfig, Protocol, TrpClient, Tx3Client


async def main() -> None:
    # 1) Load a compiled .tii protocol
    protocol = Protocol.from_file("examples/transfer.tii")

    # 2) Create a low-level TRP client
    trp = TrpClient(endpoint="https://preprod.trp.tx3.dev")

    # 3) Configure signer and parties
    sender_signer = CardanoSigner.from_mnemonic(
        address="addr_test1qz...",
        phrase="word1 word2 ... word24",
    )

    client = (
        Tx3Client(protocol, trp)
        .with_profile("preprod")
        .with_party("sender", Party.signer(sender_signer))
        .with_party("receiver", Party.address("addr_test1qz..."))
    )

    # 4) Build, resolve, sign, submit
    submitted = await (
        await (
            await client.tx("transfer").arg("quantity", 10_000_000).resolve()
        ).sign()
    ).submit()

    # 5) Wait for confirmation
    status = await submitted.wait_for_confirmed(PollConfig.default())
    print(f"Confirmed at stage: {status.stage}")


asyncio.run(main())
```

## Concepts

| SDK Type | Glossary Term | Description |
|---|---|---|
| `Protocol` | TII / Protocol | Loaded `.tii` with transactions, parties, and profiles |
| `Tx3Client` | Facade | High-level client holding protocol + TRP + party bindings |
| `TxBuilder` | Invocation builder | Collects args and resolves transactions |
| `Party` | Party | `Party.address(...)` or `Party.signer(...)` |
| `Signer` | Signer | Protocol producing a `TxWitness` for a `SignRequest` |
| `SignRequest` | SignRequest | Input passed to `Signer.sign`: `tx_hash_hex` + `tx_cbor_hex` |
| `CardanoSigner` | Cardano Signer | BIP32-Ed25519 signer at `m/1852'/1815'/0'/0/0` |
| `Ed25519Signer` | Ed25519 Signer | Generic raw-key Ed25519 signer |
| `ResolvedTx` | Resolved transaction | Output of `resolve()`, ready for signing |
| `SignedTx` | Signed transaction | Output of `sign()`, ready for submission |
| `SubmittedTx` | Submitted transaction | Output of `submit()`, pollable for status |
| `PollConfig` | Poll configuration | Poll attempts and delay for wait modes |

## Low-level TRP client

```python
from tx3_sdk import TrpClient
from tx3_sdk.trp import ResolveParams

trp = TrpClient(endpoint="http://localhost:8000", headers={"Authorization": "Bearer token"})
envelope = await trp.resolve(ResolveParams(tir=..., args={"quantity": 100}))
```

## Custom Signer

Implement the `Signer` protocol. `sign` receives a `SignRequest` carrying both
the tx hash and the full tx CBOR; hash-based signers read `tx_hash_hex`,
tx-based signers (e.g. wallet bridges) read `tx_cbor_hex`.

```python
from tx3_sdk import SignRequest, Signer
from tx3_sdk.signer import TxWitness
from tx3_sdk.signer.witness import vkey_witness


class MySigner(Signer):
    def address(self) -> str:
        return "addr_test1..."

    def sign(self, request: SignRequest) -> TxWitness:
        # sign request.tx_hash_hex with your key
        return vkey_witness(public_key_hex="aabb", signature_hex="ccdd")
```

## Manual witness attachment

When a witness is produced outside any registered signer — for example by an
external wallet app or a remote signing service — resolve the transaction
first, hand the resolved hash (or full tx CBOR) to the wallet, then attach
the returned witness before `sign()`:

```python
from tx3_sdk.signer.witness import vkey_witness

resolved = await client.tx("transfer").arg("quantity", 10_000_000).resolve()

# Hand resolved.hash (or resolved.tx_hex) to the external wallet and get
# back a witness. The wallet needs the resolved tx to sign.
witness = vkey_witness(public_key_hex="aabb", signature_hex="ccdd")  # sign resolved.hash with external wallet

signed = await resolved.add_witness(witness).sign()
submitted = await signed.submit()
```

`add_witness` may be called any number of times; manual witnesses are appended after registered-signer witnesses in attach order. Note: `ResolvedTx` is a frozen dataclass, so `add_witness` returns a new instance.

## Tx3 protocol compatibility

- TRP protocol version: `v1beta0`
- TII schema version: `v1beta0`
