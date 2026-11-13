# CORE-V instruction regression checks

`instruction_checks.c` uses raw `.word` encodings and baseline RV32IM code so
the checks do not depend on custom assembler syntax or compiler instruction
selection. It exposes the Embench-style `initialise_benchmark`, `warm_caches`,
`benchmark` and `verify_benchmark` interface. Link it to an Embench harness
that calls `benchmark()` and checks `verify_benchmark(result)`; compile the
fixture with RV32IM/ILP32 GCC, not with CORE-V optimizations enabled.

Expected output is `COREV_CHECKS checks=4592 failures=0`, and verification must
return true. Coverage includes signed/unsigned clipping (every immediate and
register bounds with bit 31 set), negative branch immediates, bit manipulation,
in-place halfword/byte shuffles, all halfword shuffle immediate selectors and
in-place packing. These are focused checks, not exhaustive ISA coverage.

The fixtures accompany fixes for:

- The out-of-range `Imm6[16]` selector in `CV_SHUFFLE_SCI_H` (use `Imm6[1]`).
- Reading SIMD inputs after an aliased destination lane was overwritten.
  Shuffle, pack and rotated-subtract behaviors snapshot their inputs.
- Clipping register bounds: bit 31 is ignored, and the signed lower bound is
  the complement of the zero-extended 31-bit upper bound. Immediate bounds
  use explicit 32-bit shift operands; source comparisons remain signed.
- Narrow counters/selectors and signed lane assignments: conversions are
  explicit rather than relying on implicit truncation or signedness changes.
- Branch PC assignment: explicitly retain XLEN bits without changing the
  conditional alignment check.

For ETISS, generate with strict validation and `--no-static-scalars` using an
M2-ISA-R version with the constructor and dynamic-PC-substitution fixes described
in SEAL5's `examples/corev/review/README.md`. The checked model comprised the
complete SIMD, bit manipulation and immediate-branch instruction groups plus
the four changed clipping behaviors, with the existing runtime's base ISA,
CSR and semihosting descriptions. Strict source/behavior validation and native
generation passed for this scope. After merging main at
`26138c3c87a5bdd3d96156afe2a4ccbb6274f8fe`, the complete merged XCoreV
instruction groups also pass strict source/behavior validation with the tested
runtime adapter. Main supplies the additional ALU/MAC conversion fixes.
The adapter includes the explicit-width JALR mask and reservation-address
initializer corrections documented by the SEAL5 review; this is not a claim
that every unmodified runtime description passes that validation.
CV_CLB remains disabled.

The PR #2 changes preserve its imports, encodings and assembly declarations;
the LLVM restoration branch is not merged here. Its wider arithmetic and
encoding corrections remain separate. Conflicts with the main head above
are resolved, retaining the signed clipping comparisons, SIMD input snapshots
and corrected halfword shuffle selector.
