---
title: Parties
sidebar:
    order: 1
---

This section explains the Tx3 syntax for _parties_.

A party is a way of abstracting the role of someone or something interacting with your protocol. In technical terms, a party is just a placeholder for an on-chain address.

# Definition

To define a name party use the following syntax:

```tx3
party {Name};
```

-  `Name`: an alphanumeric string to identify the party within the protocol.


## Examples

A basic party definition with name `Sender`:

```
party Sender;
```

Multiple party definitions in the same protocol:

```
party Sender;
party Receiver;
party Escrow;
```

# Usage

Parties can be used to specify the owner for an input UTxO:

```tx3
tx transfer(amount: Int) {
    input source {
        from: Sender,
    }
}
```

Parties can be used to specify the target for an output UTxO:

```tx3
tx transfer(amount: Int) {
    output {
        to: Receiver,
    }
}
```
