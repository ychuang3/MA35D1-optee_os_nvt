# MA35D1 Patch Inventory

Status values are `PENDING`, `IN PROGRESS`, `DONE`, or `BLOCKED`. Classification is relative to OP-TEE OS v4.11.0.

## Root Snapshot: 1736f59

Migrated as `1474c02` (`plat-ma35: add MA35D1/D0/H0 support`).

| ID | Area / files | Purpose | Class | Dependencies | Status |
|---|---|---|---|---|---|
| R01 | `plat-ma35/conf.mk`, `platform_config.h`, `sub.mk` | Add MA35D1/D0/H0 platform configuration, DRAM layout, peripheral addresses and build integration | ADAPT | v4.11 ARM platform config | DONE |
| R02 | `plat-ma35/main.c` | Register secure/non-secure mappings, initialize TSI, load its patch, provide console and HUK | ADAPT | R01, R03, R04 | DONE |
| R03 | `ma35_uart.[ch]`, `core/drivers/sub.mk` | Add the MA35 UART console driver | ADAPT | R01 | DONE |
| R04 | `tsi_cmd.[ch]`, `tsi_patch.c`, `whc.h` | Implement WHC-based TSI commands and embed the MA35D1 TSI patch | ADAPT | R01 memory map | DONE |
| R05 | `pta/crypto_pta*` | Expose MA35 AES, SHA, ECC and RSA services using TSI or direct hardware | ADAPT | R01, R04 | DONE |
| R06 | `pta/ks_pta*` | Expose key-store and OTP operations using TSI or direct hardware | ADAPT | R01, R04 | DONE |
| R07 | `pta/trng_pta*` | Expose MA35 TRNG initialization and random data | ADAPT | R01, R04 | DONE |
| R08 | top-level `Makefile` | Change the global default platform from `vexpress` to `ma35` | DROP | None | DONE |
| R09 | `.checkpatch.conf` | Remove the upstream `CFG_` Kconfig-prefix check | DROP | None | DONE |
| R10 | Mbed TLS `.uncrustify.cfg` | Delete an upstream formatting configuration file | DROP | None | DONE |

R08-R10 are source-snapshot artifacts unrelated to MA35 runtime support. The target will retain the v4.11 upstream versions and use explicit `PLATFORM=ma35` builds.

Root validation: all three flavors compile with `CFG_WERROR=y`; checkpatch
reports 0 errors (legacy warnings retained); TSI payload SHA-256 is
`c329e7001387389e5f9118c441b100146696ea47fd9352ffd588ec3f2f67e37d`.
Boot and hardware functionality are not tested.

## Follow-up Commits

| ID | Original commit | Purpose | Class | Depends on | Planned validation | Status |
|---|---|---|---|---|---|---|
| P01 | `77be1f8` | Load the TSI patch only when TSI hardware is present | ADAPT | R02, R04 | TSI and non-TSI build/review | DONE (`e704772`) |
| P02 | `f70a4de` | Avoid repeated TSI initialization | DROP | P01 | history and non-TSI path review | DONE (superseded by P04) |
| P03 | `5499943` | Update TSI patch and set its system tick | ADAPT | P01 | patch hash, build | DONE (`83d8263`) |
| P04 | `526c775` | Track successful patch loading independently of TSI version | ADAPT | P03 | retry-path review | DONE (`7938f67`) |
| P05 | `614e07f` | Use extended SHA commands for multiple TSI sessions | ADAPT | P03-P04 | multi-session SHA test | PENDING |
| P06 | `5c70df0` | Move TSI command data to secure SRAM and fix TRNG reads | ADAPT | P05 | mappings, TRNG test | PENDING |
| P07 | `f48e55e` | Move AES/SHA TSI buffers to secure SRAM and map NS DRAM | ADAPT | P06 | boundary/cache review, crypto test | PENDING |
| P08 | `4b165ce` | Update TSI patch for cascaded SHA correctness | KEEP | P05-P07 | patch hash, cascade SHA test | PENDING |
| P09 | `e2dbb45` | Generate a TRNG-backed key into key-store SRAM | ADAPT | P06-P08 | owner validation, KS test | PENDING |
| P10 | `49ace8b` | Configure a TSI memory monitor | ADAPT | P03-P09 | monitored-range review | PENDING |
| P11 | `c6babbe` | Add validated TSI OTP programming and correct OTP read sizing | ADAPT | R06, P03 | allowed-range and failure tests | PENDING |
| P12 | `69905c8` | Add direct OTP programming for non-TSI parts | ADAPT | P11 | non-TSI OTP test | PENDING |
| P13 | `3780ac9` | Bound TSI AES buffers and support cascaded GCM/CCM | ADAPT | P07 | bounds and cascade tests | PENDING |

No item is classified REDESIGN at inventory time. This will be revised if v4.11 compatibility or security review proves otherwise.

P02 is not applied as a transient source commit. It probes TSI before checking
whether hardware is present and can return before the patch image is loaded.
P04 removes that probe and replaces it with explicit successful-image state,
which preserves the intended no-repeat behavior without those regressions.
