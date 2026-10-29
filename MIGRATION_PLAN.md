# MA35D1 OP-TEE OS Migration Plan

## Baselines

- Source repository: `/home/osboxes/work/git_all/ma35d1/optee_os_ma35`
- Source head: `3780ac911666a879ed2b18af62dc25482ad46adb`
- Source snapshot: `1736f599914c5aaf58bc904a00ab07c310229711`
- Source upstream base: OP-TEE commit `6b5d1120816429cfa56ff32383b07070a0335edd`
- Target release: official OP-TEE OS `4.11.0`
- Target upstream commit: `f54190f22611dd5ad5d22546fd4209aa100f56bd`
- Local target baseline: `c747f81f08dacfd0559d9f38e19f8032f1ab5cee`
- Migration branch: `ma35d1-v4.11-migration`

The source snapshot is a root commit with no upstream parent. Its provenance is therefore reconstructed by comparing its Git tree directly with `6b5d112`. That comparison contains 22 changed paths: 5,110 insertions and 254 deletions.

## Strategy and Sequence

Keep `core/arch/arm/plat-ma35` and migrate one original commit at a time. Adapt code to v4.11 APIs without importing obsolete upstream files or changing external repositories.

1. Port the `1736f59` dependency group: build/configuration, memory map, UART, TSI transport and patch image, HUK, crypto PTA, key-store/OTP PTA, and TRNG PTA.
2. Build MA35D1, MA35D0, and MA35H0 where toolchain support permits.
3. Port commits `77be1f8` through `3780ac9` in dependency order, preserving original authorship when verified.
4. After each patch, inspect the diff, run static checks, compile affected flavors, and update tracking documents.
5. Perform boot and functional validation only when the board, firmware stack, client, and tests are available.

## Validation Strategy

- Static: review memory attributes, normal/secure boundaries, PTA parameters, physical-address conversions, cache maintenance, timeouts, and TSI synchronization.
- Build: clean builds with `PLATFORM=ma35` and each supported flavor; do not suppress new warnings.
- Boot: verify TF-A loading, OP-TEE initialization, Linux TEE probe, and client communication.
- Functional: exercise crypto, TRNG, key store, OTP, HUK-dependent storage, TSI and non-TSI paths, and `xtest` where applicable.

## Known Risks

- The root snapshot combines several security-sensitive features and cannot be cherry-picked.
- TSI uses an embedded firmware patch and fixed secure-SRAM addresses.
- Crypto PTAs consume normal-world supplied register maps and physical addresses.
- Existing HUK fallback behavior and PTA visibility must be preserved unless a separately approved security change is required.
- Hardware validation and the exact MA35 cross-toolchain are not yet confirmed.
