# ESP-r model — working notes for Claude Code

Template for using an LLM coding agent on an ESP-r model through its text files.

Save a completed copy as `CLAUDE.md` in the root of the model directory. The
agent reads a file of that name automatically, without being asked. This
template is published under a different name so that a copy still carrying its
placeholders cannot load itself into a session by accident.

Derived from the context file used in the experiments reported in [DISSERTATION
REFERENCE]. It is not that file. The file the experiments were given is
reproduced verbatim in Appendix 1 of that work, and the findings reported there
attach to it and not to this template.

Sections marked **FILL IN** describe one model and one machine, and must be
rewritten for another. Everything else transfers to any ESP-r model on a
comparable installation.

## What this is

**FILL IN.** One or two lines: which model this is, what it is being used for,
and anything a reader would otherwise assume wrongly.

- Engine binaries (`prj`, `bps`, `res`): **FILL IN** the directory.
- ESP-r source tree: **FILL IN** the path. The agent needs it to ground claims
  about file formats.

## This model

**FILL IN.** Record what the agent would otherwise have to derive, and keep it
short enough to stay true.

- Where the model was copied from, and the zone names.
- The config file, and how many zones.
- Constructions and glazing, named as they appear in the files.
- The control file: setpoints, schedule, and the stated plant capacity.
- Shading and PV. State explicitly when they are absent, not only when present.
- The climate file, the site latitude and longitude, and whether the two agree.
  A disagreement between them is a fault an agent can find.
- Which databases the model uses (`*stdmat`, `*stdmlc` and the rest).
- Baseline results, if a baseline exists. Read the fourth verification rule
  below before relying on one.

## Verification rules

- Before **any** edit, make sure the tree is committed, or work on a branch.
  Every change must be reversible.
- After any model edit, run `bps` and confirm the model still simulates and
  gives sensible numbers. Never report success on "it ran" alone.
- Ground any claim about ESP-r file formats or behaviour in the documentation or
  the source. Do not infer meaning from keyword names.
- Treat any baseline recorded above as unverified. Re-run the unmodified model
  and reproduce the figure before comparing anything against it. A small
  discrepancy is more likely to come from the number of start-up days than from
  the change under test.
- Log every experiment: the date, what was tried, the exact commands, the
  result, and what worked or failed.

## Version control

- One branch per edit. Master holds the clean baseline model and the reference
  documents that should load on every branch, including this file.
- Each edit is a single commit on its own branch on top of master.
- Reference branches rather than commit hashes in any log, because rebases
  change hashes.
- `bps` and `res` leave Fortran scratch files in `cfg/` and results in `../tmp/`.
  Those are byproducts, not model files. Leave them untracked.

## ESP-r command-line workflow (text mode)

Run `bps` and `res` from the model's `cfg/` directory, because paths inside the
config such as `../zones` and `../tmp` are relative to it. Both tools need a
**leading blank line** to accept the `-file` default at the `...file name?`
prompt.

Saved simulation presets are often **not** full-year. Check the period before
using one. For an annual run, drive the menu and enter the period explicitly
rather than using `-p <set> silent`.

**Full-year run** (`bps`), results land in `../tmp/<n>.res`:

```
bps -file <model>.cfg -mode text
```

then feed: `<blank>` `c` `<n>.res` `1 1` `31 12` `3` (start-up days) `4`
(timesteps per hour) `y` (hourly integration) `s` (commence) `y` (use control
file) `<description>` `y` (continue) `y` (save results) `-` `-`.

The number of prompts at the end of a run is not fixed. If a run completes but
fails to save, pad the confirmations rather than assuming the sequence is wrong.

**Heating and cooling totals** (`res`): `<blank>` `d` (enquire about) `f`
(energy delivered) `-` `-` `-`. Reports per-zone sensible heating and cooling in
kWh with the hours required.

**Solar gains** (`res`): `<blank>` `d` `a` (summary statistics) `d` (solar
processes) `a` (entering from outside). Note that this reports a power statistic
in mean W, not an integral. Annual energy is mean_W × period_hours. Standard
ESP-r climates are 365-day, with a header reading `1,365`, giving 8760 h.

**Zone energy balance** (`res`): `<blank>` `d` `h` `b` (integrated) `b` (by gain
and loss). The air-point balance folds solar into surface convection, so it is
not the solar gain figure. Use "solar processes" for that.

**Save level** decides what the result file can answer later. At the level used
for a routine run, surface fluxes and surface and construction node temperatures
are not stored, and `res` will decline to report them. Raise the save level
before the run if those are wanted, because they cannot be recovered afterwards.

## Local environment

**FILL IN.** Record faults belonging to this machine rather than to ESP-r, so
the agent does not mistake one for the other, and so a later reader can tell the
two apart.

Example of the kind of entry this section is for: the optional `modish`
detailed-shading helper fails at every timestep with `Can't locate
List/MoreUtils.pm` unless the perl `List::MoreUtils` module is installed. It
does not stop the solver when shading is precomputed or disabled.
