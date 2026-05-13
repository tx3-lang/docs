---
title: "Transaction Resolver Protocol"
---

TRP is the wire protocol that backends speak to resolve and submit Tx3 transactions. The OpenRPC spec lives at [tx3-lang/trp](https://github.com/tx3-lang/trp) (`v1beta0/trp.json`).

Sequence diagram of how the Client interacts with the chain through a TRP endpoint.

```mermaid
sequenceDiagram
    Client->>+TRP: resolve(IR + args)
    TRP->>+Chain: query UTxOs
    TRP->>+TRP: Apply Inputs
    TRP->>+TRP: Compile Tx
    TRP->>+Client: Return CBOR
    Client->>+Client: Sign CBOR
    Client->>+TRP: submit(txhash+signature)
    TRP->>+Chain: submit Tx
```    