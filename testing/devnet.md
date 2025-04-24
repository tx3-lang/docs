---
title: Devnet
sidebar:
    order: 1
---

This guide covers how to execute the devnet network for tests.

## Overview

Devnet is a custom Cardano network for tests where it is possible to define the initial funds for wallets. It's required to execute `tx3up` to prepare the environment. When the command is executed, dolos will be executed with a custom genesis file for the devnet network.

Create a config `trix.toml` file in the directory where trix is being executed.

```sh
trix devnet
```
It's possible to run the devnet as a background process using the parameter `-b`

## Configuration

```toml
[profiles.devnet]
chain = "CardanoDevnet"

[[profiles.devnet.wallets]]
name = "alice"
random_key = true
initial_balance = 10000000

[[profiles.devnet.wallets]]
name = "bob"
random_key = true
initial_balance = 10000000
```
