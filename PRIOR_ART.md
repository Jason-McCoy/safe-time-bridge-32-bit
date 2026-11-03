# Prior work

This file records the prior work behind Safe Time Bridge and what this project may add to it. The prior-art attribution was first documented on October 14, 2026, in the private v0.12 package, five days before this public repository was created on October 19, 2026. This file publishes that attribution here as a later update. See the revision history at the end.

This is not a complete prior-art search.

## Statement from Jason McCoy

I apologize that my documentation was not complete. The prior work below was identified and credited on October 14, 2026, but that credit was not carried into this public repository when it was created on October 19, 2026. Until this file was added, anyone reading this repository could not see the work that came before mine, and they should have been able to. This file corrects that, and the revision history at the end records what happened and when.

— Jason McCoy, November 3, 2026

## 1. Prior work as documented on October 14, 2026 (v0.12)

The text in this section is carried over word for word from `PRIOR_ART.md` in the v0.12 package. "This project" in this section refers to the v0.12 code of that date.

> Checked October 14, 2026. This is not a complete prior-art search.
>
> **The central technique is not new**
>
> - **The 28-year shift was patented during Y2K remediation.** [US Patent 5,852,824](https://patents.justia.com/patent/5852824), "Apparatus and method for processing year-date data in computer systems" (Roger W. Brown; filed May 22, 1997; issued December 22, 1998), describes "system encapsulation": resetting the system clock by an offset such as 28 years and converting stored dates, without modifying the application. The patent has expired. The technique is free to use and cannot be claimed as new.
> - **It was sold commercially in 1999.** Y2K "encapsulation" products shifted application dates back 28, 56, or 84 years, relying on the same calendar alignment, and named the same limit this project's migrator encountered: dates before 1929 could not be handled. ([Enterprise Systems Journal, Jan. 11, 1999](https://esj.com/Articles/1999/01/11/For-Fast-Fixes-Vendors-Offer-a-28Year-Solution.aspx); [ESJ, Mar. 5, 1999](https://esj.com/Articles/1999/03/05/Precision-Software-Licenses-Program-Encapsulation-from-TOCS.aspx))
> - **Clock interception is established.** [libfaketime](https://www.linux.com/?p=506853) changes the time a Linux program sees through the same `LD_PRELOAD` mechanism this project's helper uses.
> - **The kernel route was declined.** In 2020, Linux maintainers declined to add a real-time clock offset to time namespaces for Y2038 testing, describing a correct implementation as "a massive trainwreck" ([LKML, Oct. 2020](https://lkml.iu.edu/hypermail/linux/kernel/2010.3/10260.html)). Time shifting is left to user-space tools.
>
> **Origin of this work**
>
> Jason McCoy recalls recognizing the date-overflow problem and the shape of a solution around 1988, with no one to share it with at the time; there is no surviving record from that period. The 28-year shift was independently published by others during Y2K remediation. This work arrived at the same shift independently in 2026 and adds the safety engineering around it.

The 1988 recognition is Jason McCoy's recollection. It is not documentary evidence and is not offered as prior art.

**Note on the patent statement (added later).** The October 14 text says the patent has expired and that the technique is free to use. The quotation is kept as written. The expiration of one patent does not by itself establish freedom to operate: other patents or rights could apply, and this file is not legal advice. Google Patents lists US 5,852,824 as "Expired - Fee Related" with an anticipated expiration of May 22, 2017, and notes that its legal status is an assumption, not a legal conclusion.

## 2. What is known, and what this project may add

**Previously known (not claimed by this project):**

- shifting a program's clock by a whole number of 28-year cycles so the calendar lines up
- converting stored dates to match a shifted clock
- intercepting a program's time calls in user space with `LD_PRELOAD`

**What this project may add** (the October 23, 2026 build in this repository):

These are possible contributions in safety engineering around the known technique. Their novelty has not been established by a prior-art review. Each describes what the helpers are designed to do and what the project's own tests checked (see Section 3); none is independently verified.

- **Program time only, and a full stop instead of a bad time.** The reference clock for a helped program is its *program time*: the true time minus the configured shift.

  The shift is a fixed number of seconds, a whole multiple of 883,612,800 seconds per 28-year cycle. That is 10,227 days: 28 years containing 7 leap days. Subtracting it lands on the same calendar date and weekday only while each 28-year span involved contains exactly 7 leap days, within the supported date windows tested by this project. The implementation enforces a safety cutoff at 2100-02-28 23:59:59 UTC (true time) and refuses later dates rather than risk calendar misalignment (2100 is not a leap year). A shift that is not a whole multiple of 883,612,800 seconds is refused at load.

  The design rules are:
  - every time a program is given is in program time, so it is not given the true time, which is 28 or more years ahead of its own clock;
  - a program's time does not go backward.

  When a helper cannot give a time that meets both rules, it is designed to stop the program (a full stop: exit 134 from a helper, or 137 from a supervisor) rather than hand it an error value or a wrong date.
- **Helpers matched to the program.** Separate helpers cover x86-64 programs, i386 programs with 32-bit time, and i386 programs built with 64-bit time. A fail-closed launcher (`stb_run`) refuses to start a program its helper cannot protect.
- **Raw system calls covered.** Two supervisors and a crossing guard answer a program's own raw file-date calls in program time. They also refuse and declare any attempt to set the clock, and close other routes to true time: boot time, the RTC device and files, and socket timestamps.
- **A clock coordinator (shim 4).** It gives every helper one shared shift. Shift changes (28 to 56 years) are made between runs, not to a live program: stop, migrate stored data, restart. A value that cannot be translated into the program's window is shuttered, not wrapped.
- **Local time on the true year's rules,** with a current time-zone database carried in the image.
- **Limits stated and tested.** The 2100-02-28 calendar limit is tested on every path, and known limits are recorded as limits, not counted as passes.

The v0.12 package of October 14 also described an authority-verified time source, cross-checking against an RFC 3161 timestamp authority, and journaled migration with double-migration guards. Those are not listed above because this repository's test results do not cover them.

## 3. Verification status

- **Self-tested.** The 122 of 122 result for this image, and every result in [TEST_RESULTS.md](TEST_RESULTS.md), comes from the project's own test runs in QEMU and on GitHub Actions. It is evidence from the project's own testing, not independent certification.
- **Outside review.** Nori, an AI reviewer recruited by Jason McCoy, tested the October 22 build without the source code. Nori's findings led to changes in the October 23 build. That review is not an independent verification of the October 23 build.
- **Independent verification.** None is recorded in this repository. Testers with real 32-bit hardware are invited to boot the image and report results, including failures.

## 4. Sources

Each link below was opened and checked against the statement it supports on November 3, 2026.

- US Patent 5,852,824, "Apparatus and method for processing year-date data in computer systems," Roger W. Brown; filed May 22, 1997; issued December 22, 1998.  
  https://patents.justia.com/patent/5852824  
  https://patents.google.com/patent/US5852824
- "For Fast Fixes, Vendors Offer a 28-Year Solution," Enterprise Systems Journal, January 11, 1999 (TOCS encapsulation: dates set back 28, 56 or 84 years; dates before 1929 not handled).  
  https://esj.com/Articles/1999/01/11/For-Fast-Fixes-Vendors-Offer-a-28Year-Solution.aspx
- "Precision Software Licenses Program Encapsulation from TOCS," Enterprise Systems Journal, March 5, 1999.  
  https://esj.com/Articles/1999/03/05/Precision-Software-Licenses-Program-Encapsulation-from-TOCS.aspx
- "Changing what time a process thinks it is with libfaketime," Linux.com (libfaketime works through `LD_PRELOAD`).  
  https://linux.com/?p=506853
- Linux kernel mailing list, "Re: [Y2038][time namespaces] Question regarding CLOCK_REALTIME support plans in Linux time namespaces," Lukasz Majewski, October 30, 2020, quoting Thomas Gleixner's reply that declines a CLOCK_REALTIME offset ("Doing it correct is a massive trainwreck").  
  https://lkml.iu.edu/hypermail/linux/kernel/2010.3/10260.html

## 5. Acknowledgments

This file was prepared by Jason McCoy with two AI assistants. Claude (Anthropic) drafted the text, checked each source and checked the wording against the code. ChatGPT (OpenAI) reviewed each draft. Jason McCoy carried ChatGPT's reviews to Claude and Claude's drafts back to ChatGPT, and approved every change. ChatGPT's reviews recommended:

- the three requirements this file follows: historical accuracy, technical accuracy and publication integrity;
- separating the project's own testing from independent verification (Section 3);
- a revision history showing that the attribution was documented before this repository existed (Section 6);
- the note qualifying the patent statement, the full source addresses (Section 4), and a defined reference clock (Section 2);
- checking the fixed-seconds wording against the code, and describing 2100-02-28 as the implementation's enforced safety cutoff;
- that Jason McCoy approve his statement word for word, and that the exact changes be checked before publishing.

Neither assistant's review is independent verification of Safe Time Bridge.

## 6. Revision history

| Date | Event |
|:--|:--|
| October 14, 2026 | Prior-art attribution first documented, in the private v0.12 package: Part VI "Prior work" and "Origin of this work" in the v0.12 PDF, and `PRIOR_ART.md` in the v0.12 code archive. Not published in this repository at that time. |
| October 15, 2026 | One-page Scope Statement written for v0.14 (a Claude Docs document, not part of this repository), covering what had been tested, the known limits, and credit to the prior art above. Reason: an AI-generated pitch for the project had made claims it could not support, including award figures with no source, a "new product category," and coverage the project did not have. The Scope Statement was written so that a reviewer could see exactly what Safe Time Bridge did and did not cover, with the prior art credited, before relying on any claim about it. Whether it was ever published is not recorded here. |
| October 19, 2026 | This public repository created (test image, checksum, manifest, README). |
| November 3, 2026 | This file added, with a statement from Jason McCoy apologizing for the incomplete documentation, and acknowledgments to Claude (Anthropic) and ChatGPT (OpenAI). Section 1 carries over the October 14 text unchanged, with a later note on the patent statement. Sections 2 and 3 are new and describe the October 23, 2026 build. Section 4 lists every source with its full address, checked November 3, 2026. The same commit changes "Proven by booting" to "Tested by booting" in README.md, so the README does not declare certainty the project's own testing cannot give. Documentation only: no code, image or test results changed. |

---

Licensed under the GNU General Public License v3.0 only (GPL-3.0-only); see [LICENSE](LICENSE).

*Don't trust me or anyone else; trust the best truth available.* — Jason McCoy
