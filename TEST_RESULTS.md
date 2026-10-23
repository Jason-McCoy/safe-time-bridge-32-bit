# Test results: bootable image, October 23, 2026 build

This page records what the published image was tested for and the result, so you can check it without access to the private test lab. Don't take our word for it: the scenarios below are what you are invited to repeat on real 32-bit hardware.

| | |
|:--|:--|
| Image | `pfrg-stb-i386-test.iso` (27,768,832 bytes) |
| SHA-256 | `d92e9f1947d5d9867aa9f4e1ce9e408a8c50caf8c66ca7b707ec91946d1d4957` |
| Built from | lab commit `46e235f` (GitHub Actions, October 23, 2026 16:07 UTC) |
| Result | **122 of 122 tests passed**, 0 failed, 0 missing; 12 of 12 controls showed the 2038 problem without a helper; 3 of 3 checks of the test machine passed; 1 known limit recorded (still there) |
| Machine | QEMU, i686 32-bit kernel (Alpine Linux 6.12.114 LTS), booted with `vdso32=0`, guest clock set past 2038 in memory |

## How the results are counted

Changed on October 23, 2026, after Nori's review (below) showed that the earlier "107 of 107" could not reveal a missing test and counted controls and recorded gaps as passes.

- Every result the image must report is listed in its manifest (`/stb/expected_tests.txt` inside the image). A result that never reports is a failure; it can no longer just make the total smaller.
- **Tests** check the helper. **Controls** run *without* a helper and pass only when they show the 2038 problem, which proves the test would catch it. **Checks of the test machine** confirm the setup (a 32-bit kernel, a clock past 2038, the clock put back after the 2100 tests). A **recorded limit** passes only while the limit is still there. Only tests are counted as passed tests.
- Each table below is generated from that manifest and this exact ISO's own run, with the run's pass messages word for word.

## How it was tested

- **The image itself:** this exact ISO was booted in QEMU through its own GRUB menu ("ALL") and reported every expected result. The QEMU guests that run the same scenarios from the same commit also passed.
- **Before publishing:** the ISO was checked against its SHA-256; all 3 boot entries boot with `vdso32=0`; no test build of the helper or launcher is inside (the build refuses to make an image that would contain one).
- **Elsewhere in the same run (not in the image):** the launcher's supervisors on a 64-bit GitHub runner with x86-64, i386 and i386 64-bit-time programs, as an ordinary user and as root: 165 of 165 expected results passed; the helpers' own suites (shims 1, 2 and 3), including 83 directory-walk cases per build checked event for event against glibc; the i386 QEMU guest (129 expected results, 0 missing) and the live-crossing guest (9 expected, 0 missing); an x86-64 guest on a post-2038 clock; and the i386 time-boundary test. All passed, on lab branch i386-time32-shim at this commit.

Program time is the true time minus 28 years: on the test clock of 2040-02-15, programs see 2012-02-15 (same weekday). A **full stop** means the helper or launcher stops the program (exit 134, or 137 from a supervisor) rather than hand it a time it cannot trust. "2019 date" below means the `date` program from Debian 10 (2019), run as a legacy program; it does not mean the date 2019.

## New in this image (October 23, 2026)

Most of these changes answer an outside review of the October 22 build by **Nori**, an AI reviewer (a Grok bot recruited by Jason McCoy). Nori booted the published ISO with no access to the source code and found that a helped program could set the clock, several routes to true time, wrong local time in daylight-saving weeks, no lower bound on the 84-year shift, a test count that could not reveal a missing test, a 2100 limit tested on only one path, and shifts that were not whole 28-year cycles. The fixes and tests are ours; the findings are Nori's.

- **Raw file-date reads are covered.** A program's own `stat`-family system calls (and the C library's internal ones) bypassed the helper and returned true dates. `stb_run` now starts every program under two supervisors that answer those calls in program time, byte for byte, with a third process (the crossing guard) so a supervisor can die without a program ever getting true time (U section; Z5 is now closed).
- **Setting the clock is refused and declared**, so a program in program time cannot move the whole machine 28 years back (U20n).
- **Other true-time routes are closed:** the boot time in `/proc/stat`, `/proc/driver/rtc`, the `/sys` RTC files and the RTC device read in program time; socket receive timestamps are refused (U21n).
- **Local time uses the true year's daylight-saving rules**, including places with none (Arizona) or that ended it (Mexico 2022), and the image carries a current time-zone database for machines whose own is older (L section).
- **The shift must be whole 28-year cycles**, and a "now" before 1970 is refused (Y9, Y10, Y12).
- **The 2100-02-28 limit is tested on every path:** the kernel's own clock set to 2100 with no bridge, shims 2 and 3, and the supervisors, on both sides of the last second (Y13 to Y17).
- **The launcher's own 2038 bug is fixed:** its 32-bit build could not look at a file dated past 2038.

## Post-2038 suite (shims 2 and 3): 22 of 22 tests passed, plus 2 setup checks, plus 4 controls

| ID | Kind | Result |
|:--|:--|:--|
| G0a | setup check | native 32-bit kernel (i686) |
| G0b | setup check | the guest kernel clock is past 2038-01-19 (no simulated time) |
| G1 | control | control: without a helper the program fails (this is the 2038 problem) |
| G2 | test | program time is the real post-2038 clock minus 28 years; +30 days works |
| G3 | test | every reading in program time |
| G3b | test | the program's file time is on disk as true post-2038 time |
| G4 | test | every wait correct on a post-2038 kernel clock |
| G5 | test | direct calls answered in program time; direct file-time sets converted |
| G6 | test | full stop (exit 134) |
| G7 | control | control: 64-bit time is right, but its 32-bit field wraps (why shim 3 still shifts) |
| G8a | test | 64-bit-time program on program time |
| G8b | test | every reading in program time |
| G8c | test | every wait correct |
| G8d | test | file time on disk is true post-2038 time |
| G9 | test | 32-bit-time waits under shim 3 |
| G-Wa | control | control: without the helper, glibc's 32-bit nftw cannot read a file dated past 2038 at all |
| G-Wb | test | shim 2: nftw walks the whole post-2038 tree, every entry in program time (2012) |
| G-Wc | test | shim 2: fts (the helper's own) reads and descends the post-2038 tree, program time (2012) |
| G-Wd | test | shim 2: a file dated 2030 reads 2002 through stat, fstat, nftw, ftw and fts |
| G-Wb_lfs | test | shim 2: nftw (large-file build: nftw64) walks the whole post-2038 tree, every entry in program time (2012) |
| G-Wc_lfs | test | shim 2: fts64 (the helper's own) reads and descends the post-2038 tree, program time (2012) |
| G-Wd_lfs | test | shim 2: a file dated 2030 reads 2002 through stat, fstat, nftw, ftw and fts |
| G-We | control | control: a 64-bit-time program without the helper reads the true post-2038 date |
| G-Wf | test | shim 3: a 64-bit-time program's nftw, ftw and fts give program time (2012; the 2030 file reads 2002) |
| G-Ra | test | no vDSO mapped (kernel option vdso32=0); every clock read is a system call the guard can see |
| G-Rb | test | raw execve keeping the helper's settings: the child runs on program time (2012), same as its parent |
| G-Rc | test | raw execve WITHOUT the helper's settings: the child is stopped by the inherited guard (SIGSYS, exit 159) before it reads any time; it never sees true time |
| G-Rd | test | the same under shim 3: child stopped, no time read |

## Bridge chain: 10 of 10 tests passed

| ID | Kind | Result |
|:--|:--|:--|
| B0 | test | the 32-bit bridge starts (socket folder owned by its account, mode 700) |
| B1 | test | bridge -> shim 2 -> program: the 32-bit bridge read the post-2038 clock correctly and the program got it minus 28 years |
| B1b | test | bridge -> shim 3 -> 64-bit-time program |
| B2 | test | no expected bridge account set: full stop |
| B3 | test | bridge runs under a different account than expected: full stop |
| B3b | test | the bridge keeps serving after a client refused it and hung up |
| B4 | test | cache: 20,000 requests, 1 bridge contact(s), none refused or backward |
| B5 | test | 8 threads x 5,000: none refused, nothing backward |
| B6 | test | bridge stopped: full stop, no fallback to the host clock |
| B7 | test | bridge's own source frozen: bridge declares FAULT, helper relays it, full stop |

## 2019 Debian programs: 6 of 6 tests passed, plus 1 control

| ID | Kind | Result |
|:--|:--|:--|
| R0 | control | control: 2019's date cannot show the 2040 clock without a helper |
| R1 | test | 2019's date under shim 2: program time, weekday intact (2012-02-15 is a Wednesday, as is 2040-02-15) |
| R2 | test | 2019's touch sets a program date; the disk holds the true 2040 date |
| R3 | test | 2019's stat (old-glibc __xstat route) reads the file in program time |
| R3b | test | 2019's ls shows the file in program time |
| R4 | test | 2019's gawk: systime() in program time and +30 days correct |
| R5 | test | 2019's stat on a file dated 2070: full stop |

## Children through /bin/sh (dash): 7 of 7 tests passed, plus 1 control

| ID | Kind | Result |
|:--|:--|:--|
| K0 | control | control: system() -> dash -> 2019 date, no helper: no correct date (the 2038 problem) |
| K1 | test | shim 2: system() -> dash -> 2019 date shows program time 2012-02-15 |
| K2 | test | shim 2: popen() reads program time 2012-02-15 from a child |
| K3 | test | shim 2: exec of a #!/bin/sh script; its child shows program time |
| K4 | test | shim 2: three shells deep, the grandchild still shows program time |
| K5 | test | shim 2: a script that unsets the helper's settings still hands them to its child |
| K6 | test | shim 2: the shell starting a static program is a full stop; no date reaches the program |
| K7 | test | shim 3: a 64-bit-time program's system() -> dash -> 2019 date shows program time |

## 2019 Redis service: 6 of 6 tests passed, plus 1 control

| ID | Kind | Result |
|:--|:--|:--|
| S0 | control | control: 2019 Redis cannot keep the true 2040 time without a helper |
| S1 | test | Redis under shim 2 starts and its TIME is program time (1329264055) |
| S1b | test | Redis's own log is stamped in program time (Feb 2012) |
| S2 | test | key timers: 2 s and 1.5 s keys expire on time; a 60 s key keeps its TTL |
| S3 | test | date math: EXPIREAT in program time; now+2 s, now+30 days and the past all behave |
| S4 | test | 20 s of steady use: time never steps back, short-lived keys keep expiring |
| S5 | test | stored time: expiry times written to the save file read back correctly after a restart |

## Shim 4 coordinator, shift change: 14 of 14 tests passed, plus 1 recorded limit

| ID | Kind | Result |
|:--|:--|:--|
| H0 | test | coordinating with a bridge that has no shift file: full stop |
| H1 | test | shim 2 takes its shift from the coordinator: 2019 date shows 2012-02-15 Wed |
| H1b | test | the planned change (56 years, at 2050-01-01) is announced to the program at load |
| H2 | test | a second shift source alongside the coordinator: full stop |
| H3 | test | Redis under shim 3 is registered (i386-time64, 28 years) and serves program time |
| H4 | test | a new-shift program while Redis still runs on the old shift: refused (full stop); Redis is not disturbed |
| H4b | test | the bridge is killed and restarted: it reloads its registry and still refuses the new-shift program |
| H5 | test | after the stop, shift_migrate moves stored data to 56 years and switches the shared shift |
| H6 | test | restarted on the new shift: 1984-02-15 Wed (same weekday as 2040-02-15), no clock jump inside any run |
| H7 | test | XLATE: the migrated value, translated back to 28 years, is exactly the original |
| H8 | test | XLATE of a 2070 time into the 28-year window is shuttered, not wrapped |
| H9 | recorded limit | KNOWN LIMIT, recorded: Redis's binary save file is not migrated (not @T text); a 10-minute key now lives 28 years. That is the boundary translator's job (option 1) |
| H10 | test | true time 2066-06-01 is past the 28-year window: registration refused, full stop |
| H11 | test | the same moment on 56 years runs: 2010-06-01 |
| H12 | test | a damaged registry file: the bridge refuses to start rather than forget processes |

## Longer shifts, 2094 and 2100 limits: 19 of 19 tests passed, plus 1 setup check

| ID | Kind | Result |
|:--|:--|:--|
| Y1 | test | 56-year shift, 7 s before its window ends: program time 2038-01-19 Tue |
| Y2 | test | 56-year shift, 1 s past its window (2094-01-19 03:14:08): full stop, no date reaches the program |
| Y2b | test | the same moment on an 84-year shift runs: 2010-01-19 Tue |
| Y3 | test | 84-year shift in 2098: program time 2014-06-01 Sun, same weekday |
| Y3b | test | 84-year shift in 2098: a legacy program's +30-day date math works |
| Y4 | test | 84-year shift on the last calendar-exact day, 2100-02-28: program time 2016-02-28 Sun |
| Y5 | test | past the calendar limit (2100-03-01): full stop, no date reaches the program |
| Y6 | test | a file dated 2100-02-28 12:00 UTC reads 1456660800, 2016-02-28 12:00 UTC (84 years) |
| Y7 | test | a file dated 2100-03-01: full stop |
| Y8 | test | the program sets 2016-02-29 (true 2100-03-01): full stop; that date is never written |
| Y9 | test | a shift one second off 28 years (883612801): refused, full stop |
| Y10 | test | 84-year shift at true 2040: the clock is refused (it would be 1956), full stop |
| Y11 | test | a file dated 2030 on an 84-year shift reads -743083200 (1946): file dates before the window pass through unchanged, by design |
| Y12 | test | 84-year shift where its window starts (true 2054-01-01): program time 1970-01-01 Thu |
| Y13-shim2 | test | file at 2100-02-28 23:59:59 reads exactly 1456703999 (2016-02-28 23:59:59); one second later: full stop |
| Y13-shim3 | test | file at 2100-02-28 23:59:59 reads exactly 1456703999 (2016-02-28 23:59:59); one second later: full stop |
| Y14 | test | supervisors (raw statx and raw stat64): 2100-02-28 23:59:59 reads exactly 1456703999; one second later: full stop (137) |
| Y15 | test | kernel clock in the last minute before the limit: shims 2 and 3 run, program time 1456703940 / 1456703940 (2016-02-28 23:59, a Sunday) |
| Y16 | test | kernel clock at 2100-03-01, no bridge: shims 2 and 3 full stop on the clock read, no time handed over |
| Y17 | setup check | kernel clock restored after the 2100 tests |

## Local time (true-year rules): 8 of 8 tests passed, plus 1 control

| ID | Kind | Result |
|:--|:--|:--|
| L1 | test | 2019 date, Chicago, 56-year shift: 1984-03-20 13:00 CDT (the 1984 rules said 12:00 CST) |
| L2 | test | 2019 date, Sao Paulo, 28-year shift: 12:00 -0300 (the 2012 rules said 13:00 -0200) |
| L3 | test | ltime under shim 2: localtime, ctime and mktime agree; 30 local days later is still CDT, exactly 30 days |
| L4 | test | ltime_t64 under shim 3: localtime, ctime and mktime agree; 30 local days later is still CDT, exactly 30 days |
| L5 | test | 2019 date, Mexico City (no daylight saving since 2022), 28-year shift, July 2040: 12:00 CST (the 2012 rules said 13:00 CDT) |
| L6 | test | 2019 date, Arizona (no daylight saving), 56-year shift, July 2040: 11:00 MST |
| L7c | control | control: a machine with an old tzdata, no carried copy: 13:00 CDT, daylight saving Mexico ended in 2022 |
| L7 | test | the same through stb_run: the carried tzdata beside the launcher is used (declared): 12:00 CST |
| L8 | test | no TZ set, the machine's default zone is an old copy: named from /etc/localtime and taken from the carried tzdata: 12:00 CST |

## Fail-closed launcher: 12 of 12 tests passed, plus 1 control

| ID | Kind | Result |
|:--|:--|:--|
| Z1 | test | valid helper through stb_run: 2019 date shows program time 2012-02-15 Wed |
| Z1b | test | valid helper through stb_run: +30-day date math works |
| Z1c | test | shim 3 through stb_run with a 64-bit-time program |
| Z2 | test | missing helper: refused (126), the program never started |
| Z3 | test | helper the loader cannot load (missing library): refused by the system loader's check, program never started |
| Z3b | test | an x86-64 helper for an i386 program: refused |
| Z3c | test | a statically linked program: refused |
| Z4a | control | control (CentOS 4 behaviour): the loader ignores a wrong-architecture helper and the program runs unhelped, exit 0, wrong date |
| Z4b | test | the same ignored helper under the guard: the kernel stops the program at its first clock read (SIGSYS), no date |
| Z4c | test | no helper at all under the guard: stopped at its first clock read, no date |
| Z4d | test | a missing helper file under the guard: the legacy program is stopped before any date math |
| Z4e | test | a helper missing one of its own libraries: the loader itself stops the program (127) before it runs; the launcher (static) is unaffected |
| Z5 | test | GAP CLOSED (was KNOWN GAP): with no helper, a 64-bit-time program's own file-date read (2212876899 on disk, post-2038) is answered by the supervisors in program time (1329264099) |

## stb_run supervisors: 12 of 12 tests passed

| ID | Kind | Result |
|:--|:--|:--|
| U1n | test | every raw stat-family call answered (byte-for-byte check: host, after boot) |
| U3n | test | raw statx of a 2030 file: 2002 |
| U4n | test | a 2070 file: full stop (137), no value handed over |
| U14n | test | io_uring_setup -> ENOSYS |
| U15n | test | /proc/thread-self resolved for the calling thread |
| U9n | test | supervisor A killed mid-run: B answered all 60 reads (2002) |
| U12n | test | both supervisors killed: the crossing guard declared a FAULT and stopped the program (134) |
| U20n | test | all 8 ways to set the clock refused (not permitted) and declared; adjtimex reads in program time |
| U21n | test | boot time, /proc/driver/rtc, /sys rtc files and RTC_RD_TIME all in program time (2040 reads 2012); socket timestamps refused and declared |
| U19n_time32 | test | stb_run with the helper: every route reads 2002, including the raw statx that was the known gap |
| U19n_lfs | test | stb_run with the helper: every route reads 2002, including the raw statx that was the known gap |
| U19n_t64 | test | stb_run with the helper: every route reads 2002, including the raw statx that was the known gap |

## Live 2038 crossing: 6 of 6 tests passed, plus 3 controls

| ID | Kind | Result |
|:--|:--|:--|
| C0 | test | the kernel clock crossed the 32-bit limit live (2147483587 -> 2147483725) with every program running |
| C1 | control | control: an unmodified 32-bit-time program loses the clock at the crossing |
| C2 | test | shim 2 straight through the crossing: no error, no backward step, no jump, no wrap; timers armed before it fired on time after it |
| C3 | test | shim 3 (64-bit-time program) straight through the crossing, same checks |
| C3c | control | control: a 64-bit-time program's clock is fine, but its 32-bit field wraps at the crossing |
| C4 | test | bridge -> shim 2 -> program straight through the crossing; bridge never faulted |
| C5a | control | control: 2019's date goes wrong at the crossing without a helper |
| C5b | test | 2019's date under shim 2, every second through the crossing: steady 2010 program time |
| C6 | test | 2019's gawk under shim 2, every second through the crossing: steady 2010 program time |


## Known limits (stated, not hidden)

- **Use the launcher.** The supervisors (raw file dates, clock setting, the other true-time routes) protect a program started through `stb_run`. A program started with only the helper preloaded still gets true file dates from its own raw `stat` calls.
- **Kernel requirements:** Linux 5.1 or later for the helpers; seccomp user notification (Linux 5.0) for the supervisors, 5.5 to check file opens, and 5.14 to hand over the rewritten time files (older kernels refuse those files instead); no vDSO (boot with `vdso32=0`). On an older kernel, for example CentOS 4's 2.6.9, the launcher refuses to start the program rather than run it unprotected.
- **Supervised raw `stat` calls cost tens of microseconds** each, and every file open is checked. Calls through the C library go through the helper instead.
- The supervisors must be allowed to reach the program's memory (Yama `ptrace_scope` 0 or 1, the program dumpable); where they cannot, every raw `stat` is a full stop. If both supervisors and the crossing guard die, a remaining program process gets an error from a raw `stat`: not true time, but not a full stop either.
- A program process that changes its root directory or namespaces is stopped at its next raw `stat`. `io_uring` is answered "not available", so programs fall back to ordinary calls.
- Raw `timerfd_settime`, `timer_settime` and `futex` calls with an absolute wall-clock deadline are not translated (the C library's versions are).
- **Local time:** true dates outside 1900-03-01 to 2100-02-28 use the program year's rules; glibc's internal `strftime %s` is not covered. The carried time-zone database is only as current as the last image build, and a default zone whose `/etc/localtime` is a plain copy with no `/etc/timezone` cannot be named, so its old rules stay (declared).
- The 84-year shift is the only one that reaches 2098 to 2100; nothing passes 2100-02-28.
- A child started by a raw `execve` that discards the helper's settings is stopped, not helped (G-Rc).
- Data a program stores in its own binary formats is not translated (H9 records this for Redis's save file).
- Not protection against hostile code: a program can forge the helper's marker.
- These results are evidence for the emulated machine above. Reports from real 32-bit PCs are welcome: please include the CPU, the boot menu choice, and the full results screen.

*"Don't trust me or anyone else; trust the best truth available."* - Jason McCoy
