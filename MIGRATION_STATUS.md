# MA35D1 Migration Status

## Current Stage

- Stage: P07 secure-SRAM crypto migration complete
- Branch: `ma35d1-v4.11-migration`
- Source: MA35 snapshot `1736f59` plus 13 follow-up commits through `3780ac9`
- Target: OP-TEE OS `4.11.0`, upstream commit `f54190f22611dd5ad5d22546fd4209aa100f56bd`
- Next action: port `4b165ce` (cascaded SHA TSI image update)

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
| Follow-up commits migrated | IN PROGRESS (6 migrated, 1 dropped, 6 pending) |

## Validation

| Level | Result | Notes |
|---|---|---|
| Static inventory | PASS | 22 root paths and 13 follow-up commits accounted for |
| v4.11 API compatibility | PASS | Pager retention and C declaration placement adapted; source provenance diff reviewed |
| Compilation | PASS | MA35D1, MA35D0 and MA35H0 built with GCC 12.3.0 and `CFG_WERROR=y` |
| Boot | NOT TESTED | No board or boot log used yet |
| Functional | NOT TESTED | No `xtest`, crypto, TRNG, key-store or OTP execution yet |
| Checkpatch | PASS | 0 errors; 348 inherited style warnings and 20 checks retained to avoid unrelated refactoring |

## Completed Commits

- `1474c02` — port root snapshot `1736f59`; MA35D1/D0/H0 build PASS.
- `e704772` — port `77be1f8`; conditional TSI patch loading; all flavors build PASS.
- `83d8263` — port `5499943`; payload hash verified; all flavors build PASS.
- `7938f67` — port `526c775`; successful-load state and retry path reviewed; all flavors build PASS.
- `8986272` — port `614e07f`; extended SHA commands and payload hash verified; all flavors build PASS.
- `ea3523c` — port `5c70df0`; secure command buffer and TRNG changes; all flavors build PASS; MI-007 deferred.
- `5a84660` — port `f48e55e`; AES/SHA secure SRAM and NS DRAM mapping; all flavors build PASS; MI-008 deferred.

## Next Actions

1. Port `4b165ce` and verify the cascaded-SHA TSI image payload.
2. Preserve MI-007 for review after complete porting and successful bring-up.
3. Compile all affected flavors with `CFG_WERROR=y`.
4. Record boot and functional validation as NOT TESTED until hardware is used.
