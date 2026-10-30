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
  equivalent SHA bound and retains the fixed mapping length. P13's final
  cascaded GCM/CCM DMA may copy `AES_CNT + 16` bytes to a normal-world physical
  destination whose actual allocation size is not available to the PTA, and TSI
  output is copied before its return status is checked. The NS mapping ends at
  `CFG_TZDRAM_START`; static review found no overlap with secure TZDRAM.
- Decision: preserve historical behavior for the migration. Review SHA bounds
  and mapping-length validation after full porting, bring-up, and basic tests.
- Status: DEFERRED; P13 was migrated as `d129405`. This is not a claim that the
  remaining behavior is safe.

## MI-009: P10 TSI monitor range and lifecycle are undocumented

- Affected patch: `49ace8b639b6212d536b26c0cc65bb0cd9785efc`, migrated as
  `ee13470`.
- Symptom: `ma35d1_tsi_init()` requests a 4 KiB monitor at physical address
  `0x28025000` on every invocation, ignores command failure, and does not retain
  the returned monitor ID.
- Analysis: the range begins immediately after the registered 0x25000-byte TSI
  secure-SRAM buffer. No local BSP definition explains its ownership or purpose.
  Since the TSI receives a physical range, lack of an OP-TEE virtual mapping is
  not by itself proof of an error. Repeated successful initialization may consume
  additional monitor slots until the TSI reports `ST_MON_FULL`.
- Decision: preserve the historical address, interval, call placement, and error
  behavior during migration. Confirm the range from MA35 documentation and test
  repeated initialization after complete porting and successful bring-up.
- Status: DEFERRED; monitor coverage and lifecycle have not been hardware tested.

## MI-010: OTP programming PTA lacks an explicit authorization policy

- Affected patch: `c6babbe2e7fe55d0403e2a65fac8c9d603a010dc`, migrated as
  `063d465`; P12 extends the same command to non-TSI systems.
- Symptom: the device-enumerated key-store PTA exposes an irreversible OTP
  programming command. It validates the parameter type, aligned address ranges,
  and one-hot bit mask, but has no explicit caller authorization, version-update
  policy, confirmation mechanism, or read-back verification.
- Analysis: the allowed ranges are documented as version counters, which limits
  impact, but any normal-world client able to open the PTA can request permitted
  bits to be burned. Adding authorization or a monotonic version policy could
  change the deployed provisioning/update workflow and cannot be inferred safely.
- Decision: preserve the historical PTA interface and validation during
  migration. Do not execute OTP programming as part of automated validation.
  Review the intended caller and lifecycle policy after successful bring-up.
- Status: DEFERRED; compilation passed, destructive hardware behavior NOT TESTED.

## MI-011: Direct OTP controller access is not serialized

- Affected patch: `69905c848c8c1e16fff9da07d47c07ec26a4b14c`, migrated as
  `3b6313b`.
- Symptom: the non-TSI OTP read and program paths share controller address,
  data, control, and status registers without a software lock. The program path
  clears status and writes a new command without first rejecting an already-busy
  controller.
- Analysis: concurrent PTA invocations could interleave register sequences or
  clear another operation's status. Whether the current OP-TEE invocation model
  serializes this PTA sufficiently has not been established. Adding a lock or
  pre-command BUSY handling changes timing and concurrency behavior.
- Decision: preserve the historical direct-controller sequence during migration.
  Review serialization and BUSY handling after bring-up; do not exercise
  concurrent or destructive OTP tests until the intended policy is confirmed.
- Status: DEFERRED; compilation passed, direct OTP hardware behavior NOT TESTED.
