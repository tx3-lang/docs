---
title: Python
---

To integrate with Python you need to make sure that your _trix_ project has the `python` bindings generation enabled in the config:

```toml
[[bindings]]
plugin = "python"
output_dir = "./gen/python"
```

With the binding generation enabled, run the following command from your CLI to trigger the code generator:

```
trix bindgen
```

If things went well, you should see the following new files:

```
my-protocol/
└── gen/
    └── python/
        ├── my-protocol.py
        ├── test.py
        └── requirements.txt
```

That new `my-protocol.py` file has the required types and functions to call your protocol from a Python code.

The `test.py` file is a simple test file that you can use to run your protocol.

The `requirements.txt` file is a standard file for Python projects.

Now, we can use the generated bindings to build a transaction using our protocol.

First access to the `gen/python` folder and install the required dependencies:
```bash
cd gen/python && pip install -r requirements.txt 
```

We need to open `my-protocol.py` and update the TRP endpoint to point to the correct URL. The default value is `http://localhost:3000/trp`, but if you're using different port or endpoint, make sure to update it.

Finally, we can run the test file to build a transaction using our protocol:
```bash
python test.py
```
If everything went well, you should see a message like this:

```bash
{'tx': '84a400d9010281825820705e5d956d318264043baf8031e250c14b8703b69947ab2d3212d67210c4b240010182a200581d60464d4cb029fc90cf720600b2f271a2d433517ad32a67eac5eb9bba5c011a05f5e100a200581d605189743c77fc84cc571235062e4f0de28370f6273ade37697d850b40011b0000000248151b86021a0005833d0f00a0f5f6'}
```