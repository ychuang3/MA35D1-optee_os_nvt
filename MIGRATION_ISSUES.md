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
- Status: RESOLVED; no known-broken intermediate source commit created.
