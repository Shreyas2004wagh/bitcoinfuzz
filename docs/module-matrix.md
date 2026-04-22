# Module Matrix

This matrix inventories the modules currently present under `modules/` in this
revision of the repository.

- Total module directories audited: 28
- Base interface source: `include/bitcoinfuzz/basemodule.h`
- Compose service coverage source: `docker-compose.yml`
- Flag-to-directory rule source: `auto_build.py`

Notable gaps in the current tree:

- `bitcoinkernelvariant` is buildable but is not referenced by any compose
  service.
- `tinyminiscript` is buildable but is not referenced by any compose service.

| Module | Flag | Runtime | FFI mechanism | BaseModule targets | Compose services |
| --- | --- | --- | --- | --- | --- |
| `bitcoin` | `BITCOIN_CORE` | C++ | Direct native C++ integration against vendored Bitcoin Core sources and static archives | `script_parse`, `deserialize_block`, `script_eval`, `verify_script`, `descriptor_parse`, `miniscript_parse`, `address_parse`, `psbt_parse`, `addrv2_parse`, `cmpctblocks_parse`, `transaction_eval`, `bip32_master_keygen`, `bip32_deserialize_extended_key`, `bip32_derive_from_path` | `address_parse`, `addrv2`, `bip32_derive_from_path`, `bip32_deserialize_extended_key`, `bip32_master_keygen`, `cmpctblocks_parse`, `descriptor_parse`, `deserialize_block`, `miniscript_parse`, `psbt_parse`, `script`, `script_eval`, `transaction_eval`, `verify_script` |
| `bitcoinj` | `BITCOINJ` | JVM (Java/Kotlin) | JNI calls into an embedded JVM wrapper built with Gradle | `bip32_master_keygen`, `bip32_deserialize_extended_key` | `bip32_deserialize_extended_key`, `bip32_master_keygen` |
| `bitcoinkernel` | `BITCOINKERNEL` | C++ | Direct native C++ API against `libbitcoinkernel.a` | `kernel_block`, `kernel_transaction` | `kernel_block`, `kernel_transaction` |
| `bitcoinkernelvariant` | `BITCOINKERNEL_VARIANT` | C++ | Direct native C++ API against a second `libbitcoinkernel.a` archive with prefixed symbols | `kernel_block`, `kernel_transaction` | `none` |
| `btcd` | `BTCD` | Go | Go `c-archive` bridge via cgo-exported functions | `deserialize_block`, `verify_script`, `address_parse`, `psbt_parse`, `addrv2_parse`, `parse_p2p_message`, `transaction_eval`, `bip32_master_keygen`, `sign_schnorr`, `decode_ellswift`, `schnorr_verify` | `addrv2`, `bip32_master_keygen`, `decode_ellswift`, `deserialize_block`, `parse_p2p_message`, `psbt_parse`, `schnorr_verify`, `sign_schnorr`, `transaction_eval`, `verify_script` |
| `clightning` | `CLIGHTNING` | C | Direct native C API against Core Lightning sources | `deserialize_invoice`, `deserialize_offer`, `parse_p2p_lightning_message`, `decode_onion` | `decode_onion`, `deserialize_invoice`, `deserialize_offer`, `parse_p2p_lightning_message` |
| `decredsecp256k1` | `DECRED_SECP256K1` | Go | Go `c-archive` bridge via cgo-exported functions | `private_to_public_key`, `sign_compact`, `sign_der`, `sign_verify`, `ecdh` | `ecdh`, `private_to_public_key`, `sign_compact`, `sign_der`, `sign_verify` |
| `eclair` | `ECLAIR` | JVM (Java/Scala) | JNI calls into an embedded JVM wrapper compiled with `javac` | `deserialize_invoice`, `deserialize_offer` | `deserialize_invoice`, `deserialize_offer` |
| `embit` | `EMBIT` | Python | Embedded CPython interpreter via the Python C API importing `main.py` | `descriptor_parse`, `miniscript_parse`, `psbt_parse`, `bip32_master_keygen`, `bip32_deserialize_extended_key` | `bip32_deserialize_extended_key`, `bip32_master_keygen`, `descriptor_parse`, `miniscript_parse`, `psbt_parse` |
| `gocoin` | `GOCOIN` | Go | Go `c-archive` bridge via cgo-exported functions | `script_eval`, `verify_script` | `script_eval`, `verify_script` |
| `ldk` | `LDK` | Rust | Rust static library with `extern "C"` entry points | `deserialize_invoice`, `deserialize_offer`, `parse_p2p_lightning_message`, `decode_onion` | `decode_onion`, `deserialize_invoice`, `deserialize_offer`, `parse_p2p_lightning_message` |
| `libwallycore` | `LIBWALLY_CORE` | C | Direct native C API against `libwallycore` and its bundled `libsecp256k1` | `psbt_parse`, `bip32_master_keygen`, `bip32_deserialize_extended_key` | `bip32_deserialize_extended_key`, `bip32_master_keygen`, `psbt_parse` |
| `lightningkmp` | `LIGHTNING_KMP` | JVM (Kotlin) | JNI calls into an embedded JVM wrapper built with Gradle | `deserialize_invoice`, `deserialize_offer` | `deserialize_invoice` |
| `lnd` | `LND` | Go | Go `c-archive` bridge via cgo-exported functions | `deserialize_invoice`, `parse_p2p_lightning_message`, `decode_onion` | `decode_onion`, `deserialize_invoice`, `parse_p2p_lightning_message` |
| `nbitcoin` | `NBITCOIN` | .NET | Self-contained native shared library produced by `dotnet publish` and called through a C ABI header | `script_eval`, `verify_script`, `descriptor_parse`, `miniscript_parse`, `psbt_parse`, `bip32_master_keygen`, `sign_schnorr`, `bip32_deserialize_extended_key` | `bip32_deserialize_extended_key`, `bip32_master_keygen`, `descriptor_parse`, `miniscript_parse`, `psbt_parse`, `script_eval`, `sign_schnorr`, `verify_script` |
| `nbitcoinsecp256k1` | `NBITCOIN_SECP256K1` | .NET | Self-contained native shared library produced by `dotnet publish` and called through a C ABI header | `private_to_public_key`, `sign_compact`, `sign_der`, `sign_verify`, `ecdh`, `schnorr_verify` | `ecdh`, `private_to_public_key`, `schnorr_verify`, `sign_compact`, `sign_der`, `sign_verify` |
| `nlightning` | `NLIGHTNING` | .NET | Self-contained native shared library produced by `dotnet publish` and called through a C ABI header | `deserialize_invoice` | `deserialize_invoice` |
| `pybitcoinkernel` | `PYBITCOINKERNEL` | Python | Embedded CPython interpreter via the Python C API importing `main.py` | `kernel_block`, `kernel_transaction` | `kernel_block`, `kernel_transaction` |
| `pycoin` | `PYCOIN` | Python | Embedded CPython interpreter via the Python C API importing `main.py` | `bip32_master_keygen`, `bip32_deserialize_extended_key` | `bip32_deserialize_extended_key`, `bip32_master_keygen` |
| `rustbitcoin` | `RUST_BITCOIN` | Rust | Rust static library with `extern "C"` entry points | `script_parse`, `deserialize_block`, `address_parse`, `psbt_parse`, `addrv2_parse`, `cmpctblocks_parse`, `parse_p2p_message`, `bip32_master_keygen`, `bip32_deserialize_extended_key`, `bip32_derive_from_path` | `address_parse`, `addrv2`, `bip32_derive_from_path`, `bip32_deserialize_extended_key`, `bip32_master_keygen`, `cmpctblocks_parse`, `deserialize_block`, `parse_p2p_message`, `psbt_parse`, `script` |
| `rustbitcoinkernel` | `RUSTBITCOINKERNEL` | Rust | Rust static library with `extern "C"` entry points | `kernel_block`, `kernel_transaction` | `kernel_block`, `kernel_transaction` |
| `rustk256` | `RUST_K256` | Rust | Rust static library with `extern "C"` entry points | `private_to_public_key`, `sign_compact`, `sign_der`, `sign_verify`, `ecdh`, `sign_schnorr` | `ecdh`, `private_to_public_key`, `sign_compact`, `sign_der`, `sign_schnorr`, `sign_verify` |
| `rustminiscript` | `RUST_MINISCRIPT` | Rust | Rust static library with `extern "C"` entry points | `descriptor_parse`, `miniscript_parse` | `descriptor_parse`, `miniscript_parse` |
| `rustpsbt` | `RUST_PSBT` | Rust | Rust static library with `extern "C"` entry points | `psbt_parse` | `psbt_parse` |
| `rustreexo` | `RUSTREEXO` | Rust | Rust static library with `extern "C"` entry points | `stump_modify_add` | `stump_modify_add` |
| `secp256k1` | `SECP256K1` | C | Direct native C API against upstream `libsecp256k1` | `private_to_public_key`, `sign_compact`, `sign_der`, `sign_verify`, `ecdh`, `sign_schnorr`, `decode_ellswift`, `schnorr_verify` | `decode_ellswift`, `ecdh`, `private_to_public_key`, `schnorr_verify`, `sign_compact`, `sign_der`, `sign_schnorr`, `sign_verify` |
| `tinyminiscript` | `TINY_MINISCRIPT` | Rust | Rust static library with `extern "C"` entry points | `descriptor_parse` | `none` |
| `utreexo` | `UTREEXO` | Go | Go `c-archive` bridge via cgo-exported functions | `stump_modify_add` | `stump_modify_add` |
