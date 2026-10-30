# MA35D1 Migration Issues

## MI-001: Source root commit has no upstream ancestry

- Affected patch: `1736f599914c5aaf58bc904a00ab07c310229711`
- Symptom: the commit has no parent and the source repository does not contain `6b5d112`.
- Analysis: this is a full source snapshot rather than a normal patch.
- Resolution: compare its tree against the official upstream `6b5d1120816429cfa56ff32383b07070a0335edd` tree in an isolated analysis clone.
- Status: RESOLVED; 22 changed paths identified.

## MI-002: Upstream 4.11.0 signing key is unavailable locally

- Affected baseline: annotated tag object `8963e60a7daa9d430f56d8976ba2477f3e0c9375`.
- Symptom: the tag contains a PGP signature, but GPG and GitHub report an unknown public key.
- Analysis: the official remote tag and GitHub API both resolve to source commit `f54190f22611dd5ad5d22546fd4209aa100f56bd`; the local baseline tree matches that commit exactly.
- Status: DOCUMENTED; official repository identity and object IDs verified, full PGP trust-chain verification unavailable.

## MI-003: MA35 build environment

- Affected patches: all.
- Resolution: reused GCC 12.3.0 from
  `/home/osboxes/buildroot/ma35d1/upstream/output/host/bin/aarch64-nuvoton-linux-gnu-`.
  Builds use `PLATFORM=ma35`, the applicable `PLATFORM_FLAVOR`, and
  `CFG_WERROR=y`; no external repository was modified.
- Status: RESOLVED; MA35D1, MA35D0 and MA35H0 compilation passed for `1474c02`.

## MI-004: Hardware validation environment not yet established

- Affected patches: all hardware-facing changes.
- Required validation: TSI and non-TSI boot, crypto, TRNG, key store, OTP, HUK/secure storage and power-cycle behavior.
- Status: OPEN; compilation must not be reported as hardware success.

## MI-005: Security-sensitive legacy behavior requires preservation review

- Affected areas: HUK fallback, device-enumerated PTAs, normal-world physical buffers, fixed secure-SRAM mappings and OTP programming.
- Analysis: changing these behaviors during a version migration could break deployed systems; copying them without bounds and access review could preserve vulnerabilities.
- Proposed resolution: retain externally visible behavior, add only required validation and compatibility fixes, and stop for approval if security semantics must change.
- Status: OPEN, under per-patch review.

## MI-006: P02 probes TSI before hardware-presence check

- Affected patch: `f70a4de49e2599b873c99a4c44f640e35e1aee81`.
- Symptom: `TSI_Get_Version()` is called before `SYS_CHIPCFG.TSIEN` is checked;
  an early success also skips loading the TSI patch image.
- Analysis: later patch `526c775` removes this probe and tracks successful image
  loading explicitly. The final v4.2 source therefore does not retain P02's
  implementation.
- Resolution: classify P02 as DROP and port P04's replacement separately.
- Status: RESOLVED; replacement ported as `7938f67`, with no known-broken
  intermediate source commit.

## MI-007: P06 fixed secure-SRAM buffer lacks bounds and serialization

- Affected patches: `5c70df00ffcbf43ac91392b7d45b14f7edbf1cdd`
  and `e2dbb45a048a10e7761f490ca38b533b70c012b0`.
- Symptom: the TRNG PTA passes caller-controlled `rq_size` to an 8 KiB fixed
  buffer at `0x28023000` without an upper bound. AES and TRNG also share this
  buffer without cross-PTA serialization, and TRNG copies output before checking
  the TSI command result. P09's key-store command also uses the shared buffer to
  seed its TSI PRNG operation.
- Analysis: requests larger than `TSI_CMD_BUFF_SIZE` can make TSI write beyond
  the registered secure SRAM region. Concurrent AES/TRNG operations can overwrite
  each other's command data. The final v4.2 source does not correct these cases.
- Proposed resolution: process TRNG output in bounded chunks, serialize every
  transaction using the shared command buffer, and copy output only after command
  success. This preserves arbitrary-length requests but changes synchronization
  and failure behavior relative to the historical patch.
- Decision: preserve the historical behavior during migration. Revisit bounds,
  serialization, and failure ordering only after all patches are ported and the
  platform passes bring-up and basic functional testing.
- Status: DEFERRED; tracked for post-bring-up security review. This is not a
  claim that the behavior is safe. Historical behavior was migrated in
  `ea3523c` and P09 was migrated in `86f4f5f`.

## MI-008: P07 SHA buffer and normal-memory mapping assumptions

- Affected patch: `f48e55ee1302d7b437f0a896c1ea7bebed8e5c69`, migrated as
  `5a84660`.
- Symptom: caller-controlled SHA data length is copied into a fixed 32 KiB
  secure-SRAM buffer without a bound. AES has the same issue at P07. Calls to
  `phys_to_virt()` request a fixed 1 MiB span rather than the actual operation
  length.
- Analysis: P13 adds AES input/output bounds, but the final source still has no
  equivalent SHA bound and retains the fixed mapping length. The NS mapping ends
  at `CFG_TZDRAM_START`; static review found no overlap with secure TZDRAM.
- Decision: preserve historical behavior for the migration. Review SHA bounds
  and mapping-length validation after full porting, bring-up, and basic tests.
- Status: DEFERRED; this is not a claim that the current behavior is safe.
