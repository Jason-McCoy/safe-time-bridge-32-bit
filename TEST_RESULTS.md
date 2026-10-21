# Test results: bootable image, October 20, 2026 (evening build)

This page records what the published image was tested for and the result, so you can check it without access to the private test lab. Don't take our word for it: the scenarios below are what you are invited to repeat on real 32-bit hardware.

| | |
|:--|:--|
| Image | `pfrg-stb-i386-test.iso` (26,798,080 bytes) |
| SHA-256 | `b0a0bea0bba49eba4afbbca3d95340bf32c21c425ec4f4cb42d48aa4ebb07509` |
| Built from | lab commit `966c8d0` (GitHub Actions, October 21, 2026 02:57 UTC) |
| Result | **84 of 84 scenarios passed**, 0 failed |
| Machine | QEMU, i686 32-bit kernel (Alpine Linux 6.12 LTS), booted with `vdso32=0`, guest clock set past 2038 in memory |

## How it was tested

- **The image itself:** this exact ISO was booted in QEMU through its own GRUB menu ("ALL"); every scenario passed. The QEMU guests that run the same scenarios from the same commit also passed; the descriptions below are their own pass messages, word for word.
- **Before publishing:** the ISO was checked against its SHA-256; all 3 boot entries boot with `vdso32=0`; no test build of the helper is inside (the build refuses to make an image that would contain one).
- **Elsewhere in the same run (not in the image):** the helpers' test suites on 64-bit GitHub runners (shims 1, 2 and 3), including 83 comparison cases checking the helper's directory walks event for event against glibc; an x86-64 guest on a post-2038 clock. All passed.

Program time is the true time minus 28 years: on the test clock of 2040-02-15, programs see 2012-02-15 (same weekday). A **full stop** means the helper stops the program (exit 134) rather than hand it a time it cannot trust.

## Post-2038 suite (shims 2 and 3): 28 of 28 passed

| ID | Result |
|:--|:--|
| G0a | native 32-bit kernel (i686) |
| G0b | the guest kernel clock is past 2038-01-19 (no simulated time) |
| G1 | control: without a helper the program fails (this is the 2038 problem) |
| G2 | program time is the real post-2038 clock minus 28 years; +30 days works |
| G3 | every reading in program time |
| G3b | the program's file time is on disk as true post-2038 time |
| G4 | every wait correct on a post-2038 kernel clock |
| G5 | direct calls answered in program time; direct file-time sets converted |
| G6 | full stop (exit 134) |
| G7 | control: 64-bit time is right, but its 32-bit field wraps (why shim 3 still shifts) |
| G8a | 64-bit-time program on program time |
| G8b | every reading in program time |
| G8c | every wait correct |
| G8d | file time on disk is true post-2038 time |
| G9 | 32-bit-time waits under shim 3 |
| G-Wa | control: without the helper, glibc's 32-bit nftw cannot read a file dated past 2038 at all |
| G-Wb | shim 2: nftw walks the whole post-2038 tree, every entry in program time (2012) |
| G-Wc | shim 2: fts (the helper's own) reads and descends the post-2038 tree, program time (2012) |
| G-Wd | shim 2: a file dated 2030 reads 2002 through stat, fstat, nftw, ftw and fts |
| G-Wb_lfs | shim 2: nftw (large-file build: nftw64) walks the whole post-2038 tree, every entry in program time (2012) |
| G-Wc_lfs | shim 2: fts64 (the helper's own) reads and descends the post-2038 tree, program time (2012) |
| G-Wd_lfs | shim 2: a file dated 2030 reads 2002 through stat, fstat, nftw, ftw and fts |
| G-We | control: a 64-bit-time program without the helper reads the true post-2038 date |
| G-Wf | shim 3: a 64-bit-time program's nftw, ftw and fts give program time (2012; the 2030 file reads 2002) |
| G-Ra | no vDSO mapped (kernel option vdso32=0); every clock read is a system call the guard can see |
| G-Rb | raw execve keeping the helper's settings: the child runs on program time (2012), same as its parent |
| G-Rc | raw execve WITHOUT the helper's settings: the child is stopped by the inherited guard (SIGSYS, exit 159) before it reads any time; it never sees true time |
| G-Rd | the same under shim 3: child stopped, no time read |

## Bridge chain (the Safe Rust time bridge as the time source): 10 of 10 passed

| ID | Result |
|:--|:--|
| B0 | the 32-bit bridge starts (socket folder owned by its account, mode 700) |
| B1 | bridge -> shim 2 -> program: the 32-bit bridge read the post-2038 clock correctly and the program got it minus 28 years |
| B1b | bridge -> shim 3 -> 64-bit-time program |
| B2 | no expected bridge account set: full stop |
| B3 | bridge runs under a different account than expected: full stop |
| B3b | the bridge keeps serving after a client refused it and hung up |
| B4 | cache: 20,000 requests, 1 bridge contact(s), none refused or backward |
| B5 | 8 threads x 5,000: none refused, nothing backward |
| B6 | bridge stopped: full stop, no fallback to the host clock |
| B7 | bridge's own source frozen: bridge declares FAULT, helper relays it, full stop |

## Real 2019 Debian programs: 7 of 7 passed

| ID | Result |
|:--|:--|
| R0 | control: 2019's date cannot show the 2040 clock without a helper |
| R1 | 2019's date under shim 2: program time, weekday intact (2012-02-15 is a Wednesday, as is 2040-02-15) |
| R2 | 2019's touch sets a program date; the disk holds the true 2040 date |
| R3 | 2019's stat (old-glibc __xstat route) reads the file in program time |
| R3b | 2019's ls shows the file in program time |
| R4 | 2019's gawk: systime() in program time and +30 days correct |
| R5 | 2019's stat on a file dated 2070: full stop |

## Children through /bin/sh (2019 Debian dash): 8 of 8 passed

| ID | Result |
|:--|:--|
| K0 | control: system() -> dash -> 2019 date, no helper: no correct date (the 2038 problem) |
| K1 | shim 2: system() -> dash -> 2019 date shows program time 2012-02-15 |
| K2 | shim 2: popen() reads program time 2012-02-15 from a child |
| K3 | shim 2: exec of a #!/bin/sh script; its child shows program time |
| K4 | shim 2: three shells deep, the grandchild still shows program time |
| K5 | shim 2: a script that unsets the helper's settings still hands them to its child |
| K6 | shim 2: the shell starting a static program is a full stop; no date reaches the program |
| K7 | shim 3: a 64-bit-time program's system() -> dash -> 2019 date shows program time |

## 2019 Debian Redis 5 as a long-running service: 7 of 7 passed

| ID | Result |
|:--|:--|
| S0 | control: 2019 Redis cannot keep the true 2040 time without a helper |
| S1 | Redis under shim 2 starts and its TIME is program time (1329264056) |
| S1b | Redis's own log is stamped in program time (Feb 2012) |
| S2 | key timers: 2 s and 1.5 s keys expire on time; a 60 s key keeps its TTL |
| S3 | date math: EXPIREAT in program time; now+2 s, now+30 days and the past all behave |
| S4 | 20 s of steady use: time never steps back, short-lived keys keep expiring |
| S5 | stored time: expiry times written to the save file read back correctly after a restart |

## Shim 4: the clock coordinator and a planned shift change (28 to 56 years): 15 of 15 passed

| ID | Result |
|:--|:--|
| H0 | coordinating with a bridge that has no shift file: full stop |
| H1 | shim 2 takes its shift from the coordinator: 2019 date shows 2012-02-15 Wed |
| H1b | the planned change (56 years, at 2050-01-01) is announced to the program at load |
| H2 | a second shift source alongside the coordinator: full stop |
| H3 | Redis under shim 3 is registered (i386-time64, 28 years) and serves program time |
| H4 | a new-shift program while Redis still runs on the old shift: refused (full stop); Redis is not disturbed |
| H4b | the bridge is killed and restarted: it reloads its registry and still refuses the new-shift program |
| H5 | after the stop, shift_migrate moves stored data to 56 years and switches the shared shift |
| H6 | restarted on the new shift: 1984-02-15 Wed (same weekday as 2040-02-15), no clock jump inside any run |
| H7 | XLATE: the migrated value, translated back to 28 years, is exactly the original |
| H8 | XLATE of a 2070 time into the 28-year window is shuttered, not wrapped |
| H9 | KNOWN LIMIT, recorded: Redis's binary save file is not migrated (not @T text); a 10-minute key now lives 28 years. That is the boundary translator's job (option 1) |
| H10 | true time 2066-06-01 is past the 28-year window: registration refused, full stop |
| H11 | the same moment on 56 years runs: 2010-06-01 |
| H12 | a damaged registry file: the bridge refuses to start rather than forget processes |

## Live crossing of 2038-01-19 03:14:07 UTC: 9 of 9 passed

| ID | Result |
|:--|:--|
| C0 | the kernel clock crossed the 32-bit limit live (2147483590 -> 2147483730) with every program running |
| C1 | control: an unmodified 32-bit-time program loses the clock at the crossing |
| C2 | shim 2 straight through the crossing: no error, no backward step, no jump, no wrap; timers armed before it fired on time after it |
| C3 | shim 3 (64-bit-time program) straight through the crossing, same checks |
| C3c | control: a 64-bit-time program's clock is fine, but its 32-bit field wraps at the crossing |
| C4 | bridge -> shim 2 -> program straight through the crossing; bridge never faulted |
| C5a | control: 2019's date goes wrong at the crossing without a helper |
| C5b | 2019's date under shim 2, every second through the crossing: steady 2010 program time |
| C6 | 2019's gawk under shim 2, every second through the crossing: steady 2010 program time |

## Known limits (stated, not hidden)

- A program's own raw `stat`-family system calls still return true file dates (not yet closed).
- A child started by a raw `execve` that discards the helper's settings is stopped, not helped (G-Rc).
- Data a program stores in its own binary formats is not translated (H9 records this for Redis's save file).
- These results are evidence for the emulated machine above. Reports from real 32-bit PCs are welcome: please include the CPU, the boot menu choice, and the full results screen.

*"Don't trust me or anyone else; trust the best truth available."* - Jason McCoy
