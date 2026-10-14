# runpack_trial

PyArmor **trial** runtime (runtime id `000000`, Linux x86_64) for the obfuscated Odoo modules of the FSDI ecosystem.

> **Development and testing only.**
> In production use **`runpack`**, the company runtime built with the registered PyArmor license.
> Do not ship customer-facing modules against this package.

## What it is

Odoo modules obfuscated with PyArmor import their runtime as a top-level package:

```python
from pyarmor_runtime_000000 import __pyarmor__
```

This repository provides that package (`pyarmor_runtime_000000`) so it can be installed once with `pip`,
instead of copying the runtime into every module. It is the same idea as Exo's `runpack`.

- Runtime id: `000000` (PyArmor trial, shared by every trial user).
- Platform: `linux.x86_64` (what Odoo.sh runs). It does not run on Windows or macOS.

The `.so` in `pyarmor_runtime_000000/` is specific to one CPython version: `pyarmor.cli.core` ships a
different binary per Python minor version, so a single tag can only serve one. Pin the tag that matches
the Python version of the Odoo.sh stage you are building for.

| Tag     | Python (Odoo.sh stage) | Odoo version seen so far |
|---------|-------------------------|---------------------------|
| `v0.1.0`| 3.12                     | Odoo 18 (CHILi)            |
| `v0.2.0`| 3.14                     | Odoo 20                    |

## Use it in a project

Add the line matching your stage's Python version to the `requirements.txt` at the root of the project
(the repository Odoo.sh builds):

```
runpack_trial @ git+https://github.com/FSDinvest/runpack_trial.git@v0.1.0   # Python 3.12
runpack_trial @ git+https://github.com/FSDinvest/runpack_trial.git@v0.2.0   # Python 3.14
```

Pin a tag so the runtime only changes when you decide. The build must be able to read this repository
(public, or a token in the URL for a private one). If the Odoo.sh stage's Python version does not match
the tag, the module fails to import with an `undefined symbol` error from `pyarmor_runtime.so`.

Odoo.sh installs `requirements.txt` on every build. Nothing else is needed: the modules find
`pyarmor_runtime_000000` on the Python path.

## Build modules against this runtime

Build with the **same Python version** as the runtime you are targeting (`pyarmor.cli.core` resolves its
own binary from the interpreter that runs it, and pulls in `pyarmor.cli.core.linux` for the cross-platform
`.so` on first use):

```bash
# use the Python interpreter matching the target tag above, e.g. a 3.14 install for v0.2.0
python -m pyarmor.cli gen -r -O dist/<module> --platform linux.x86_64 \
    --use-runtime /path/to/runpack_trial <module_source>
```

`--use-runtime` points at the **repository root** (where `.pyarmor.ikey` lives), not at the
`pyarmor_runtime_000000` folder itself.

Keep `__manifest__.py`, XML and CSV files in clear text; only the `.py` files are obfuscated.
Do not put a `pyarmor_runtime_000000` folder inside the modules.

## Moving to production

1. Buy the PyArmor license and build the production runtime in the `runpack` repository (it gets its own runtime id).
2. Rebuild the modules against it.
3. In each project replace the `runpack_trial` line by the `runpack` one in `requirements.txt`.

The two runtimes have different package names, so they can be installed side by side while projects are migrated one by one.

## Do not commit

The PyArmor registration file (`pyarmor-regfile-*.zip`) and any `.pyarmor.ikey` / `pyarmor.rkey` key files.
