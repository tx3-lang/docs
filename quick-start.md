---
title: Quick Start Guide
---

This guide will help you write your first Tx3 program. We'll create a simple transfer program that allows one party to send ADA to another.

We're assuming you have some experience building transactions for UTxO blockchains. If you need a refresher, here's a [good one]().

## Install the Toolchain

To start working with Tx3 you'll need to install the toolchain. The fastest way to setup your environment is to run `tx3up`, which bundles everything you need in a single install experience:

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/tx3-lang/up/releases/download/v0.4.1/tx3up-installer.sh | sh
```

Once you've installed `tx3up`, run the following command to execute the install of toolchain:

```bash
tx3up
```

You'll see how the process downloads and install many components. By the end, you should have everything you need.

Run the following command to check the installed versions of each component:

```bash
tx3up show
```

The installer has a lot of advanced features, check our [tx3up guide](/tools/tx3up) if you want to learn more.

## Initialize a new Project

The next thing you need to do is prepare a project workspace. This is a directory in your file system that holds everything you need for developing your protocol.

Lets create a `my-protocol` folder for our tutorial:

```bash
mkdir my-protocol && cd my-protocol
```

Once you're inside your protocol folder, you need to initialize the basic files using `trix`, the package manager for **tx3**.

```bash
trix init
```

The `init` command will command will ask you a few questions and then create a few files. Make sure to opt-in into the "Typescript" code generation option because we'll need it for the tutorial.

By then end, your protocol folder should look like:

```
my-protocol/
├── main.tx3
└── trix.toml
```

**Trix** is to **Tx3** what **Npm** is to **NodeJS**

## Your First Tx3 Protocol


The `trix init` command generates a `main.tx3` file with a very basic example. Open the file in your code editor of choice so that we can inspect the contents together.

TIP: if use VSCode, there's a [Tx3 extensions]() in the marketplace that provides many QoL features like syntax highlighting and error diagnostics.


```tx3
party Sender;
party Receiver;

tx transfer(
    quantity: Int
) {
    input source {
        from: Sender,
        min_amount: Ada(quantity),
    }
    
    output {
        to: Receiver,
        amount: Ada(quantity),
    }

    output {
        to: Sender,
        amount: source - Ada(quantity) - fees,
    }
}
```

The above code represent a basic protocol with only one operation that transfers ADA from one party o another. 

Lets break it down to understand each part.

### Party Definitions

```tx3
party Sender;
party Receiver;
```

Defines two parties that will participate in the transaction. These are placeholder names that will be bound to actual addresses at runtime.

In this particular scenario, we have a `Sender` that is transferring some ADA to a `Receiver` party.

You're  not limited to only two parties, in more advanced scenarios you'll learn how to define any number of parties to fully describe your protocol.

### Transaction Template

```tx3
tx transfer(quantity: Int)
```

This snippet defines a transaction template named `transfer`. You can think of these templates as functions that your code will call to build concrete transactions.

This template takes a single parameter name `quantity` of type `Int`. In this particular scenario, this params determines how much ADA to transfer.

Inside this template you'll need to define how to connect inputs and outputs to fully describe your UTxO transaction.

### Input Block

```tx3
input source {
    from: Sender,
    min_amount: Ada(quantity),
}
```

This snippet defines an input block named `source`. An input block specifies the criteria that we need to use to query UTxO at runtime to fullfil the requirements of the transaction.

In this particular scenario we're requiring the input to come from the address belonging to the `Sender` and must contain at least the `quantity` of ADA specified as a parameter of the transaction.

Notice that we specified the name `source` to identify this particular input. Templates can have multiple input blocks. The name will be useful to reference this particular input from other parts of the transaction.

The `Ada(quantity)` syntax is an asset constructor. In more advanced scenarios you'll learn how to use other type of assets.

### Output Block

```tx3
output {
    to: Receiver,
    amount: Ada(quantity),
}
```

This snippet defines an output block. An output block describes how to construct UTxO that will be included at runtime as part of the concrete Tx.

In this particular scenario we're specifying that the UTxO should be locked in the address of the `Receiver` party and that the amount it should contain the exact `quantity` of ADA specified as a parameter.

### Another Output Block

```tx3
output {
    to: Sender,
    amount: source - Ada(quantity) - fees,
}
```

This snippet contains another output block. This time it represent the "change" we need to give back to the `Sender` party to balance the transaction.

Notice that the `amount` is now a computed value that takes whatever values was found in the `source` input and then subtracts the transferred value and the fees for the transaction.

The `fees` keyword is a special placeholder that will be replaced by the computed at runtime to match the minimum fees required for the concrete transaction being built.

## Checking for Errors

The above code should be work out-of-the-box, but if you are building your own protocol you might want to check that everything compiles correctly.

The Tx3 compilers comes with a parser and semantic analysis logic to ensure that there aren't any errors in your code.

To execute these checks, run the following command from the root of your protocol folder:

```bash
trix check
```

TIP: The VSCode extension comes with a live error diagnostics that use the same logic.

## Building Transactions

Ok, here's where things get interesting. Now that we have our protocol interface, we'll be generating concrete transactions just by calling the transaction templates as if they were functions.

Remember that we asked you to choose the "Typescript" option? We need it for the next step.

Bindings are just glue code that allows you to execute the transaction building logic by calling language-specific functions that represent the templates in your protocol.

Run the following command from your CLI:

```bash
trix bindgen
```

If things went well, you should see the following new file:

```
my-protocol/
└── gen/
    └── typescript/
        └── my-protocol.ts
```

That new .ts file has the required types and functions to call your protocol from a Typescript code. It works on the backend and also in the browser (with the corresponding bundling).

```js
import { protocol, TransferParams } from "./my-protocol";

const params: TransferParams {
  sender: "addr_test1vpgcjapuwl7gfnzhzg6svtj0ph3gxu8kyuadudmf0kzsksqrfugfc",
  receiver: "addr_test1vpry6n9s987fpnmjqcqt9un35t2rx5t66v4x06k9awdm5hqpma4pp",
  quantity: 100000000
};

const cbor = protocol.transferTx(params);
console.log(cbor);
```

## What's next

If you made it so far you should have a pretty good idea of what is Tx3 about, but this just barely scratched the surface.

Check the following section to continue your journey:

- Learn the details of the language in our [Language Guide](./language/).
- Check a more complex scenario in our [Example Catalog](./examples/).
- Learn how to run an [Ephemeral Devnet]() to try out your protocol.
- Learn how to install and use our [VSCode extension]().
- Understand the [Tx3 Architecture](./architecture/) making this work.