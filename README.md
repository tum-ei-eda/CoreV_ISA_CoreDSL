# CoreV_ISA_CoreDSL

`top.core_desc` defines the ETISS `RV32IMACFDXCoreV` architecture without XMNN.
Its base ISA is pinned independently by the `rv_base` submodule. Initialize it
with `git submodule update --init --recursive` before parsing the top file.

`tum_mod.core_desc` and `tum_rva.core_desc` provide the ETISS support definitions.
They were imported from `PhilippvK/xmnn-isax-flow` revision
`70e9950cfa5cd28c4ae5e798c948d806e77a4213` (`cdsl/`). The initial `rv_base`
pin, `bc210640cb8082e686982e29ba9ca950e5b3c3f2`, is also taken from that flow.
These inputs are maintained here; generation does not copy or modify them.

Imports use filenames directly. Pass the pinned base directory using M2-ISA-R's
`-I` option (run from this repository):

```sh
python -m m2isar.frontends.coredsl2.parser top.core_desc -I rv_base --infer-types --validate
```
