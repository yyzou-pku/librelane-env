# LibreLane 3.0.16 pixi environment

A reproducible [pixi](https://pixi.sh) environment that runs
[LibreLane](https://github.com/librelane/librelane) 3.0.16 with the **exact**
open-source EDA tool versions that upstream LibreLane 3.0.16 ships with.

## Sources

| what | where | how |
|------|-------|-----|
| LibreLane + its Python deps (`ciel`, `lln-libparse`, `yamlcore`, ...) | **PyPI** (`[pypi-dependencies]`) | prebuilt wheels, no build step |
| EDA tools | **`unic-cass` / `duyhieu`** conda channels | exact pinned commits |
| non-EDA runtime libs (`libffi 3.4.8`, `tk 8.6.15`, `cairo`/`icu`/`qt`) | **Anaconda `defaults`** | solver metadata only, irrelevant versions |

The EDA tools are the only thing that needs exactness; everything Python comes
from PyPI at whatever version satisfies LibreLane.

## Usage

```bash
pixi install

pixi run librelane --version
pixi run version          # alias
pixi run smoke-test       # full RTL -> GDS smoke test (downloads sky130A)
```

## Exact EDA tool versions

| tool           | pixi version                  | upstream commit / version                               |
|----------------|-------------------------------|----------------------------------------------------------|
| OpenROAD       | `2.0_28825_gdcf36133a3`       | `dcf36133a369abc8f3c5e5738cd4d82e4903c0e0` (2026-02-17) |
| OpenSTA        | bundled `sta` from OpenROAD   | 2.7.0 (`857316ff...`, 2026-02-14)                       |
| Yosys          | `0.62_0_g7326bb7d6`           | 0.62 @ `7326bb7d6`                                      |
| KLayout        | `0.30.7_0_g6dd3e6b10`         | 0.30.7                                                  |
| Magic          | `8.3.623_0_gfd12c39c`         | 8.3.623 @ `fd12c39c`                                    |
| Netgen (LVS)   | `1.5.316_0_gb76c9a1`          | 1.5.316 @ `b76c9a1`                                     |
| Verilator      | `5.044_0_g8df584ca1`          | 5.044 @ `8df584ca1`                                     |
| Icarus Verilog | `0_8_7207_gdb82380ce`         | `db82380ce` (13.0 devel)                                |

These match the `ghcr.io/librelane/librelane:3.0.16` image on version
(`librelane --smoke-test` passes).

## The PyPI KLayout wheel

`pip`/PyPI `librelane` depends on the PyPI `klayout` package. That wheel is
pinned to `0.30.7` here to match the conda tool, but it is **shadowed** by the
exact conda KLayout bindings because `[activation.env]` puts
`$CONDA_PREFIX/lib/pymod` ahead of `site-packages` on `PYTHONPATH`. The imported
module and the `klayout` CLI are therefore both the conda 0.30.7 build.

## Why not `conda-pypi`?

`conda-pypi` (anaconda.org) is conda's PyPI-backed channel: it indexes
*pure-Python PyPI wheels* and is consumed through conda's `conda-pypi` plugin,
not as a regular conda channel (it has no `repodata.json`). pixi cannot use it;
pixi's native equivalent is `[pypi-dependencies]`, which is what this file uses.
Even under `conda`, `conda-pypi` would not help with `lln-libparse` (a compiled
wheel is not pure Python).

## Why Anaconda `defaults`?

The `unic-cass` / `duyhieu` EDA packages were built against Anaconda's C++
runtime, and their conda metadata reflects that:

* `magic` -> `cairo >=1.18.4` -> `icu >=75.1`
* `openroad` -> `libboost >=1.82,<1.83` -> conda-forge's Boost 1.82 needs `icu <74`
* `yosys` -> `libffi >=3.4.8,<3.5` -> conda-forge has `3.4.6`, then `3.5.2`
* `magic`/`netgen` -> `tk >=8.6.15,<8.7` -> conda-forge has `8.6.13`, then `9.0.x`

None of these are real ABI requirements (e.g. any `libffi 3.4.x` provides the
same `libffi.so.8`), but conda's solver enforces the metadata, and `defaults`
happens to provide matching versions.

## Notes

* The machine exports `all_proxy=socks5://...`; `socksio` is included so
  `httpx` (used by `ciel`) can speak SOCKS.
