---
title: Policies
sidebar:
    order: 2
---

This section explains the Tx3 syntax for _policies_.

Policies represent specific logic involved with a protocol. These are usually scripts on-chain that validate actions.

# Definition

To define a named policy use the following syntax:

```tx3
party {Name};
```

-  `Name`: an alphanumeric string to identify a policy within the protocol.

If you have a static on-chain identifier for this policy use the following syntax:

```tx3
policy {Name} = {Value};
```

-  `Name`: an alphanumeric string to identify a policy within the protocol.
-  `Value`: the on-chain identifier of the policy usually represented as the hash of the script.

## Examples

A basic policy definition with name `Validator`:

```
party Validator;
```

A policy definition with a static on-chain identifier:

```
policy Vesting = 0xABCDEF;
```

# Usage

Policies can be used to specify the owner for an input UTxO:

```tx3
tx transfer(amount: Int) {
    input source {
        from: Validator,
    }
}
```

Policies can be used to specify the target for an output UTxO:

```tx3
tx transfer(amount: Int) {
    output {
        to: Vesting,
    }
}
```
