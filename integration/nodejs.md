---
title: NodeJS Backend
sidebar:
  label: NodeJS Backend
---

To integrate with NodeJS you need to make sure that your _trix_ project has the `typescript` bindings generation enabled in the config:

```toml
[[bindings]]
plugin = "typescript"
output_dir = "./gen/typescript"
```

With the binding generation enabled, run the following command from your CLI to trigger the code generator:

```
trix bindgen
```

If things went well, you should see the following new files:

```
my-protocol/
└── gen/
    └── typescript/
        ├── my-protocol.ts
        ├── test.ts
        ├── package.json
        └── tsconfig.json
```

That new my-protocol.ts file has the required types and functions to call your protocol from a Typescript code. It works on the backend and also in the browser (with the corresponding bundling).

The `test.ts` file is a simple test file that you can use to run your protocol.

The `package.json` and `tsconfig.json` files are standard files for Typescript projects.

Now, we can use the generated bindings to build a transaction using our protocol.

First access to the `gen/typescript` folder and install the dependencies:
```bash
cd gen/typescript && npm install
```
**NOTE**: You can use yarn, pnpm or any other package manager you prefer.

We need to open `my-protocol.ts` and update the TRP endpoint to point to the correct URL. The default value is `http://localhost:3000/trp`, but if you're using different port or endpoint, make sure to update it.

Finally, we can run the test file to build a transaction using our protocol:
```bash
npm run test
```
If everything went well, you should see a message like this:

```bash
{
  tx: '84a400d9010281825820705e5d956d318264043baf8031e250c14b8703b69947ab2d3212d67210c4b240010182a200581d60464d4cb029fc90cf720600b2f271a2d433517ad32a67eac5eb9bba5c011a05f5e100a200581d605189743c77fc84cc571235062e4f0de28370f6273ade37697d850b40011b0000000248151b86021a0005833d0f00a0f5f6'
}
```
