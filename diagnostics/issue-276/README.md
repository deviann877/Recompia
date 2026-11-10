# Response material for issue #276, comment 6738274479

Prepared for the maintainer request at:
https://github.com/Dittobyte/Recompia/issues/276#issuecomment-6738274479

Nothing in this folder has been posted automatically. No game data, saves,
binaries, debug symbols or credentials are included.

## Exact patch and provenance

Base: 59607eccafa9d9565d1f8a76230c4d58271117f2 (v0.8.3-beta.1).

- `candidate-readback.patch`: original patch file retained from the tested build.
- `candidate-readback-portable.patch`: Git-generated version of the same four
  runtime implementation changes, with portable patch line endings.
- `patch-validation.json`: checksums and validation result. The portable patch was
  applied to the three original upstream source files fetched by exact commit;
  all four resulting files were compared with the hardware-tested candidate,
  normalizing CRLF/LF only. There were no runtime code differences.

Use the portable patch for applying to a clean checkout:

```text
git checkout 59607eccafa9d9565d1f8a76230c4d58271117f2
git apply --check candidate-readback-portable.patch
git apply candidate-readback-portable.patch
```

The patch contains no local symbol-generation CMake switch. The tested build
used that separate build-only switch to retain a PDB/map; runtime source was as
described here. PR #358 contains the same runtime changes with additional docs,
changelog and policy checks:
https://github.com/Dittobyte/Recompia/pull/358

Tested candidate DLL SHA256:
8b8c430e2fb3a4d4c0682da445ec059ddf54fa9f3ff6847fbc0668c79ee4de3d

## Configuration / environment name

The original test set `[nvn] readback_defer_texture = 286` in `pkp.ini`.
It did not add an environment override to Steam launch options.

The runtime's standard `config_env_name("nvn", "readback_defer_texture")`
mapping produces:

```text
NXRT_NVN_READBACK_DEFER_TEXTURE=286
```

Related environment names are `NXRT_NVN_READBACK_DEFER` and
`NXRT_NVN_READBACK_LOG`. Environment values take precedence over INI values.
The selector is read by `config_get_int`, cached in the static readback config;
a full game restart is required when changing it. Selector 0 disables the
candidate's experiment. The environment key is candidate-specific, not an option
to enable on the previous official implementation.

## Logs and limits of the existing comparison

All files retain every available log line and the original order, including
startup, loading, warnings, detailed hitches and ten-second frame summaries.
Only home-directory prefixes and device UUIDs were redacted in these files;
counts and original/sanitized hashes are in `logs-manifest.json`.

| File | Contents and limitations |
| --- | --- |
| `candidate-off-full.log` | Entire preserved candidate-off log, 7,745 lines, ending around 219 seconds. Readback timing logging was OFF. Startup reports internal cap 0 and no later internal cap change; effective external cap was not recorded. |
| `candidate-on-longer-full.log` | Entire preserved later candidate-on log, 10,955 lines, ending around 647 seconds. Readback timing logging ON. Internal cap changes from 0 to 45 at t=106.367628 seconds. File was completed before LSFG was added to Testing. |
| `candidate-on-first-snapshot.log` | Original initial enabled-run snapshot, 3,309 lines, about 239 seconds. Captured while the game was running; do not describe it as an end-of-session log. Internal cap 0, no later cap change recorded. |

The full-log wording means the entire available file was preserved; it does not
assert an explicit shutdown event was logged. The game was closed when the
longer logs were recovered. No log has been trimmed to selected favorable windows.

The off log and longer on log both predate LSFG addition. A subsequent LSFG run
was recovered privately but is intentionally excluded from this no-LSFG package.
The source log metadata and prior shortcut-change checkpoint establish this
chronology; the logs alone are not a complete external-limiter/LSFG environment
capture.

`log-analysis.json` indexes cap changes, readback configuration, poll counts and
the final frame/readback summaries. It is an aid to navigation, not a substitute
for the complete logs. The initial on snapshot has 23 ten-second frame summaries;
the longer on log contains 64; the off log contains 21. These counts describe available log
entries, not validated measurement windows.

These sessions are NOT the requested repeated, matched-cap alternating A/B:
logging differed, the cap changed during the longer run, route/measurement
boundaries were not marked, and save/load/transition coverage was not recorded
systematically. CPU readback timing is not GPU-copy duration or complete pacing.

The original official deferred path remains excluded from the test plan.
Only the custom candidate is to be alternated; Gaming stays unchanged.
