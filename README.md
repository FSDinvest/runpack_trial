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
- Tested with: Python 3.12 (Odoo.sh stage).

## Use it in a project

Add this line to the `requirements.txt` at the root of the project (the repository Odoo.sh builds):

```
runpack_trial @ git+https://github.com/FSDinvest/runpack_trial.git@v0.1.0
```

Pin a tag so the runtime only changes when you decide. The build must be able to read this repository
(public, or a token in the URL for a private one).

Odoo.sh installs `requirements.txt` on every build. Nothing else is needed: the modules find
`pyarmor_runtime_000000` on the Python path.

## Build modules against this runtime

The modules must be generated with this runtime instead of embedding one:

```bash
pyarmor gen -r -O dist/<module> --platform linux.x86_64 \
    --use-runtime /path/to/runpack_trial/pyarmor_runtime_000000 <module_source>
```

Keep `__manifest__.py`, XML and CSV files in clear text; only the `.py` files are obfuscated.
Do not put a `pyarmor_runtime_000000` folder inside the modules.

## Moving to production

1. Buy the PyArmor license and build the production runtime in the `runpack` repository (it gets its own runtime id).
2. Rebuild the modules against it.
3. In each project replace the `runpack_trial` line by the `runpack` one in `requirements.txt`.

The two runtimes have different package names, so they can be installed side by side while projects are migrated one by one.

## Do not commit

The PyArmor registration file (`pyarmor-regfile-*.zip`) and any `.pyarmor.ikey` / `pyarmor.rkey` key files.
