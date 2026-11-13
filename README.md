# safe-time-bridge-32-bit
This is an ISO Safe Time Bridge 32 bit legacy test
Bootable 32-bit test image for PFRG Safe Time Bridge (legacy BIOS; USB stick or CD). SHA-256: d92e9f1947d5d9867aa9f4e1ce9e408a8c50caf8c66ca7b707ec91946d1d4957. Tested by booting this exact ISO in QEMU through its own menu: 122 of 122 tests passed, with every expected result reported (built from lab commit 46e235f on October 23, 2026; every scenario and its result is listed in TEST_RESULTS.md, because the test lab itself is private; controls, checks of the test machine and a recorded limit are counted separately from tests there). New in this image (October 23, 2026), most of it answering a review of the October 22 build by Nori, an AI reviewer (a Grok bot recruited by Jason McCoy) who tested the published ISO without the source code: a program started through stb_run now runs under two supervisors and a crossing guard that answer its own raw file-date system calls in program time (the known gap in the October 22 build), refuse and declare any attempt to set the clock, and close the other routes to true time (the boot time, the RTC device and files, socket timestamps); local time follows the true year's daylight-saving rules, and the image carries a current time-zone database for machines whose own is older; the shift must be whole 28-year cycles and a time before 1970 is refused; and the 2100-02-28 limit is tested on every path, including the kernel's own clock with no bridge running. Carried over from earlier builds: finding STB-CAL-001 (the 56-year shift's window ends at 2094-01-19 03:14:07 UTC; an 84-year shift carries programs to 2100-02-28, and past that date the helpers stop with no date); the fail-closed launcher, which refuses to start a program unless its helper matches the program's architecture and the system's own loader can load it; the kernel's clock shortcut (the vDSO) off in every boot entry; nftw, ftw and fts in program time; children through a dynamic 32-bit /bin/sh (2019 Debian dash); the 2019 Debian Redis 5 server as a long-running service; and shim 4, the clock coordinator, with a planned shift change from 28 to 56 years. Prior work: the 28-year shift at the center of this project is established Y2K technique ("system encapsulation": US Patent 5,852,824, filed 1997, now expired; commercial tools in 1999), and libfaketime already intercepts program time the same way; this project arrived at the shift independently and may add safety engineering around it, not yet independently verified (see PRIOR_ART.md). License: Pre-Failure Research Group's own work here, both the code in this image (the Safe Time Bridge helpers, the launcher and its supervisors, the bridge and the test and boot scripts) and the documentation in this repository (README.md, TEST_RESULTS.md and PRIOR_ART.md), is licensed under the GNU General Public License v3.0 only (GPL-3.0-only); see LICENSE. Contains third-party software under its own licenses: Linux kernel and busybox (GPL-2.0, sources from Alpine Linux 3.22), glibc (LGPL-2.1, Ubuntu 24.04), the time-zone database (tzdata, public domain, Ubuntu 24.04), and Debian 10 packages, each under the license in its Debian copyright file: coreutils and gawk (GPL-3.0), dash, Redis 5 and their libraries (every package, version and SHA-256 is listed in MANIFEST.txt; sources at sources.debian.org). Corresponding source for these components is available on request from Pre-Failure Research Group LLC.

## Independent Testing Invitation — Year 2038 Challenge

Can a legacy 32-bit Linux application continue operating beyond January 19, 2038, without modifying its original source code?

Safe Time Bridge is an experimental compatibility and safety architecture developed by Pre-Failure Research Group (PFRG). Linux developers, security researchers, retrocomputing enthusiasts, students, and independent investigators are invited to test the published system.

### What is available

- [Bootable 32-bit Linux test ISO](pfrg-stb-i386-test.iso) and [SHA-256 checksum](pfrg-stb-i386-test.iso.sha256).
- [Documented test scenarios and expected outcomes](TEST_RESULTS.md).
- [Architecture context and prior art](PRIOR_ART.md).
- Recorded limitations and known failure conditions in the test results.

The October 23, 2026 build reported **122 of 122 tests passing** in project testing. These results have not yet been independently reproduced by an unaffiliated human evaluator.

### Your challenge

Download the ISO, verify its checksum, and attempt to reproduce the published results in an isolated test environment. We particularly welcome investigations involving time-boundary failures, process supervision, clock access, compatibility with older applications, and unexpected system behavior.

Successful reproductions, negative results, and previously unknown failures are equally valuable. [Submit an independent test report](../../issues/new?template=independent-test-report.md) with the environment, steps, expected and observed behavior, and supporting evidence. Remove sensitive information before submitting logs.

**No endorsement is requested. Independent evidence is the objective.**

Internal PFRG test results and independently submitted results must remain clearly distinguished.
