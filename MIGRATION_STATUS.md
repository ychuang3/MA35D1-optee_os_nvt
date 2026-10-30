# MA35D1 Migration Status

## Current Stage

- Stage: all source patches ported; hardware validation pending
- Branch: `ma35d1-v4.11-migration`
- Source: MA35 snapshot `1736f59` plus 13 follow-up commits through `3780ac9`
- Target: OP-TEE OS `4.11.0`, upstream commit `f54190f22611dd5ad5d22546fd4209aa100f56bd`
- Next action: MA35D1 bring-up and basic functional validation

## Progress

| Work item | Result |
|---|---|
| Source and target repositories identified | PASS |
| Source and target working trees clean at start | PASS |
| Source upstream baseline identified | PASS |
| Root snapshot compared with upstream tree | PASS |
| All follow-up commits inventoried | PASS |
| Dedicated migration branch created | PASS |
| Root snapshot migrated | PASS |
| Follow-up commits migrated | PASS (12 migrated, 1 dropped, 0 pending) |

## Validation

| Level | Result | Notes |
|---|---|---|
| Static inventory | PASS | 22 root paths and 13 follow-up commits accounted for |
| Final source provenance | PASS | Final MA35 tree compared with `3780ac9`; remaining differences are documented v4.11/build/style adaptations |
| v4.11 API compatibility | PASS | All inventoried patches compile on v4.11; adaptations are recorded per commit |
| Compilation | PASS | MA35D1, MA35D0 and MA35H0 built with GCC 12.3.0 and `CFG_WERROR=y` |
| Boot | NOT TESTED | No board or boot log used yet |
| Functional | NOT TESTED | No `xtest`, crypto, TRNG, key-store or OTP execution yet |
| Checkpatch | PASS | P13: 0 errors, 0 warnings and 0 checks |

## Completed Commits

- `1474c02` — port root snapshot `1736f59`; MA35D1/D0/H0 build PASS.
- `e704772` — port `77be1f8`; conditional TSI patch loading; all flavors build PASS.
- `83d8263` — port `5499943`; payload hash verified; all flavors build PASS.
- `7938f67` — port `526c775`; successful-load state and retry path reviewed; all flavors build PASS.
- `8986272` — port `614e07f`; extended SHA commands and payload hash verified; all flavors build PASS.
- `ea3523c` — port `5c70df0`; secure command buffer and TRNG changes; all flavors build PASS; MI-007 deferred.
- `5a84660` — port `f48e55e`; AES/SHA secure SRAM and NS DRAM mapping; all flavors build PASS; MI-008 deferred.
- `90e8478` — port `4b165ce`; cascaded-SHA image payload hash verified; all flavors build PASS.
- `86f4f5f` — port `e2dbb45`; TRNG key-store write command and extended TSI PRNG command; payload hash verified; all flavors build PASS; MI-007 remains deferred.
- `ee13470` — port `49ace8b`; configure the MA35D1 TSI memory monitor; all flavors build PASS; MI-009 deferred.
- `063d465` — port `c6babbe`; validated TSI OTP programming and corrected OTP read sizing; all flavors build PASS; MI-010 deferred.
- `3b6313b` — port `69905c8`; direct OTP programming for non-TSI systems; all flavors build PASS; MI-010 and MI-011 deferred.
- `d129405` — port `3780ac9`; bound TSI AES buffers and add cascaded GCM/CCM support; all flavors build PASS; MI-008 remains deferred.

## Next Actions

1. Perform MA35D1 bring-up and capture the complete OP-TEE/Linux boot log.
2. Run basic TEE communication, crypto, TRNG, key-store and non-destructive OTP-read tests.
3. Exercise cascaded AES GCM/CCM with valid, boundary and oversized requests.
4. Review MI-007 through MI-011 only after bring-up and basic tests pass.
5. Keep destructive OTP programming NOT TESTED until a separately approved provisioning test is available.
