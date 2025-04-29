---
title: Rust
---

To integrate with Rust you need to make sure that your _trix_ project has the `rust` bindings generation enabled in the config:

```toml
[[bindings]]
plugin = "rust"
output_dir = "./gen/rust"
```

With the binding generation enabled, run the following command from your CLI to trigger the code generator:

```
trix bindgen
```

If things went well, you should see the following new files:

```
my-protocol/
└── gen/
    └── rust/
        ├── my-protocol.rs
        ├── test.rs
        └── Cargo.toml
```

That new `my-protocol.rs` file has the required types and functions to call your protocol from a Rust code.

The `test.rs` file is a simple test file that you can use to run your protocol.

The `Cargo.toml` file is a standard file for Rust projects.

Now, we can use the generated bindings to build a transaction using our protocol.

First access to the `gen/rust` folder
```bash
cd gen/rust
```

We need to open `my-protocol.rs` and update the TRP endpoint to point to the correct URL. The default value is `http://localhost:3000/trp`, but if you're using different port or endpoint, make sure to update it.

Finally, we can run the test file to build a transaction using our protocol:
```bash
cargo run --bin test
```
If everything went well, you should see a message like this:

```bash
TxEnvelope { tx: "84a400d9010281825820705e5d956d318264043baf8031e250c14b8703b69947ab2d3212d67210c4b240010182a200581d60464d4cb029fc90cf720600b2f271a2d433517ad32a67eac5eb9bba5c011a05f5e100a200581d605189743c77fc84cc571235062e4f0de28370f6273ade37697d850b40011b0000000248151b86021a0005833d0f00a0f5f6" }
```