---
title: Go SDK
sidebar:
  label: Go
  order: 2
---

The official Go SDK for Tx3 is published as the [`github.com/tx3-lang/go-sdk/sdk`](https://pkg.go.dev/github.com/tx3-lang/go-sdk/sdk) module. It loads a compiled `.tii` protocol, binds parties and signers, and drives the full transaction lifecycle (resolve, sign, submit, confirm) via the Transaction Resolve Protocol (TRP).

The source lives at [tx3-lang/go-sdk](https://github.com/tx3-lang/go-sdk).

## Installation

```bash
go get github.com/tx3-lang/go-sdk/sdk
```

## Quick start

```go
package main

import (
    "context"
    "fmt"
    "log"

    tx3 "github.com/tx3-lang/go-sdk/sdk"
    "github.com/tx3-lang/go-sdk/sdk/signer"
    "github.com/tx3-lang/go-sdk/sdk/trp"
)

func main() {
    // Load a compiled .tii protocol
    protocol, err := tx3.ProtocolFromFile("transfer.tii")
    if err != nil {
        log.Fatal(err)
    }

    // Connect to a TRP server
    trpClient := tx3.NewTRPClient(trp.ClientOptions{
        Endpoint: "http://localhost:3000",
    })

    // Create a Cardano signer from a mnemonic
    mySigner, err := signer.CardanoSignerFromMnemonic(
        "addr_test1qz...",
        "word1 word2 ... word24",
    )
    if err != nil {
        log.Fatal(err)
    }

    // Configure client with profile and parties
    client := tx3.NewClient(protocol, trpClient).
        WithProfile("preprod").
        WithParty("sender", tx3.SignerParty(mySigner)).
        WithParty("receiver", tx3.AddressParty("addr_test1qz..."))

    ctx := context.Background()

    // Build, resolve, sign, submit, and wait for confirmation
    status, err := client.Tx("transfer").
        Arg("quantity", 10_000_000).
        Resolve(ctx)         // -> ResolvedTx
        .Sign()              // -> SignedTx
        .Submit(ctx)         // -> SubmittedTx
        .WaitForConfirmed(ctx, tx3.DefaultPollConfig())
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Confirmed at stage %s\n", status.Stage)
}
```

> **Note:** The quick-start example above shows the conceptual flow. In real code, check errors at each step since Go doesn't chain errors across method calls.

## Concepts

| SDK Type | Glossary Term | Description |
|---|---|---|
| `Protocol` | TII / Protocol | Loaded `.tii` file exposing transactions, parties, and profiles |
| `Tx3Client` | Facade | Entry point holding protocol, TRP client, and party bindings |
| `TxBuilder` | Invocation builder | Collects args, resolves via TRP |
| `Party` | Party | Named participant — `AddressParty` (read-only) or `SignerParty` (signing) |
| `Signer` | Signer | Interface producing a `TxWitness` for a `SignRequest` |
| `SignRequest` | SignRequest | Input passed to `Signer.Sign`: `TxHashHex` + `TxCborHex` |
| `CardanoSigner` | Cardano Signer | BIP32-Ed25519 signer at `m/1852'/1815'/0'/0/0` |
| `Ed25519Signer` | Ed25519 Signer | Generic raw-key Ed25519 signer |
| `ResolvedTx` | Resolved transaction | Output of `Resolve()`, ready for signing |
| `SignedTx` | Signed transaction | Output of `Sign()`, ready for submission |
| `SubmittedTx` | Submitted transaction | Output of `Submit()`, pollable for status |
| `PollConfig` | Poll configuration | Controls `WaitForConfirmed` / `WaitForFinalized` polling |

## Low-level TRP client

```go
import "github.com/tx3-lang/go-sdk/sdk/trp"

client := trp.NewClient(trp.ClientOptions{
    Endpoint: "http://localhost:3000",
    Headers:  map[string]string{"Authorization": "Bearer token"},
})

envelope, err := client.Resolve(ctx, trp.ResolveParams{...})
resp, err := client.Submit(ctx, trp.SubmitParams{...})
status, err := client.CheckStatus(ctx, []string{txHash})
```

## Custom Signer

Implement the `Signer` interface. `Sign` receives a `SignRequest` carrying both
the tx hash and the full tx CBOR; hash-based signers read `TxHashHex`, tx-based
signers (e.g. wallet bridges) read `TxCborHex`.

```go
import "github.com/tx3-lang/go-sdk/sdk/signer"

type MySigner struct { /* ... */ }

func (s *MySigner) Address() string { return "addr_test1..." }

func (s *MySigner) Sign(request signer.SignRequest) (*signer.TxWitness, error) {
    // sign request.TxHashHex with your key
    return signer.NewVKeyWitness(pubKeyHex, signatureHex), nil
}

client.WithParty("sender", tx3.SignerParty(&MySigner{}))
```

## Manual witness attachment

When a witness is produced outside any registered signer — for example by an
external wallet app or a remote signing service — resolve the transaction
first, hand the resolved hash (or full tx CBOR) to the wallet, then attach
the returned witness before `Sign()`:

```go
import "github.com/tx3-lang/go-sdk/sdk/trp"

resolved, err := client.Tx("transfer").Arg("quantity", 10_000_000).Resolve(ctx)
if err != nil { /* ... */ }

// Hand resolved.Hash (or resolved.TxHex) to the external wallet and get
// back a witness. The wallet needs the resolved tx to sign.
var witness trp.TxWitness // sign resolved.Hash with external wallet

signed, err := resolved.AddWitness(witness).Sign()
if err != nil { /* ... */ }

submitted, err := signed.Submit(ctx)
```

`AddWitness` may be called any number of times; manual witnesses are appended after registered-signer witnesses in attach order.

## Error handling

All errors are discriminable via `errors.As()` — no string matching needed:

```go
import "github.com/tx3-lang/go-sdk/sdk/facade"

_, err := client.Tx("transfer").Resolve(ctx)
var unknownParty *facade.UnknownPartyError
if errors.As(err, &unknownParty) {
    fmt.Printf("Party %q not found in protocol\n", unknownParty.Name)
}
```

## Tx3 protocol compatibility

- **TRP protocol version:** v1beta0
- **TII schema version:** v1beta0
