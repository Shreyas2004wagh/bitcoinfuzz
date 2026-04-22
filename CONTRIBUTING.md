# Contributing

This repository integrates many Bitcoin and Lightning implementations behind a
single `BaseModule` interface. The quickest way to avoid broken wiring is to
treat a new module as a cross-cutting change, not just a new directory under
`modules/`.

## New Module Checklist

For a module that should build through the standard Docker workflow, these are
the six tracked integration points to check and usually update:

| File | Why it matters | What to add |
| --- | --- | --- |
| `modules/<module>/module.h` | Declares the `BaseModule` subclass and the targets it overrides. | A class deriving from `BaseModule` with the supported target methods marked `override`. |
| `modules/<module>/module.cpp` | Implements the adapter between bitcoinfuzz and the foreign library/runtime. | Constructor, target method bodies, and any bridge helpers needed by the module. |
| `modules/<module>/Makefile` | Builds the module-specific archive and any runtime bridge artifacts. | `module.a`, the foreign build step, `clean`, and format targets. |
| `Makefile` | Links the module into the root `bitcoinfuzz` binary when its `-D` flag is present. | A new `ifneq` block that appends `modules/<module>/module.a` to `MODULES`. |
| `auto_build.py` | Makes `CXXFLAGS=-D... ./auto_build.py` able to locate and build the module. | Confirm the default naming rule works for the new flag, or add a special case if it does not. |
| `docker-compose.yml` | Makes the module part of at least one runnable fuzz target. | Add the module flag to one or more service `CXXFLAGS` lines. |

Runtime-specific bridge files usually live next to the module as well, for
example Go wrappers, Rust crates, Python scripts, or .NET projects. Those are
part of the module implementation, but the six files above are the shared
integration surface every contributor should check first.

## Flag To Directory Rules

`auto_build.py:get_module_dir()` currently resolves flags using these rules:

| Flag pattern | Directory |
| --- | --- |
| `CUSTOM_MUTATOR_*` | `custommutator/` |
| `BITCOIN_CORE` | `modules/bitcoin/` |
| anything else | `modules/<flag.lower().replace("_", "")>/` |

Examples:

| Compile flag | BaseModule subclass | Directory |
| --- | --- | --- |
| `BITCOIN_CORE` | `Bitcoin` | `modules/bitcoin/` |
| `RUST_BITCOIN` | `Rustbitcoin` | `modules/rustbitcoin/` |
| `LIBWALLY_CORE` | `LibwallyCore` | `modules/libwallycore/` |
| `NBITCOIN_SECP256K1` | `NBitcoinSecp256k1` | `modules/nbitcoinsecp256k1/` |
| `BITCOINKERNEL_VARIANT` | `BitcoinKernelVariant` | `modules/bitcoinkernelvariant/` |
| `CUSTOM_MUTATOR_BOLT11` | custom mutator only | `custommutator/` |

## Practical Notes

- Start from an existing module in the same language/runtime family.
- Keep the compile flag spelling consistent across `Makefile`,
  `auto_build.py`, and `docker-compose.yml`.
- Use [docs/module-matrix.md](./docs/module-matrix.md) as the current ground
  truth for which targets each module implements and which compose services
  compile it.
- The current tree has a few legacy exceptions: not every existing module is
  referenced by a compose service yet. New modules should still be wired into at
  least one service so they are reachable through the documented Docker flow.
