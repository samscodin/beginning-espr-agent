# ESP-r model — working notes for Claude Code

Template for using an LLM coding agent on an ESP-r model through its text files.

Save a completed copy as `CLAUDE.md` in the root of the model directory. The
agent reads a file of that name automatically, without being asked. This
template is published under a different name so that a copy still carrying its
placeholders cannot load itself into a session by accident.

Derived from the context file used in the experiments reported in the dissertation
"Building energy modelling and quality assurance in ESP-r with a general-purpose
LLM agent" (University of Strathclyde, 2026). This template is not that file. The
file the experiments were given is reproduced verbatim in Appendix 1 of that work,
and the findings reported there attach to it and not to this template.

Sections marked **FILL IN** describe one model and one machine, and must be
rewritten for another. Everything else transfers to any ESP-r model on a
comparable installation.

## What this is

**FILL IN.** One or two lines: which model this is, what it is being used for,
and anything a reader would otherwise assume wrongly. Delete the example lines
below once you have replaced them.

> *Example: clean single-zone ESP-r prototype used for agent edit experiments.
> This is a working copy, and the original that ships with the engine is
> untouched.*

- Engine binaries (`prj`, `bps`, `res`): **FILL IN** the directory.
  *Example: `/usr/local/bin`, which is what `which bps` reports.*
- ESP-r source tree: **FILL IN** the path. The agent needs it to ground claims
  about file formats, and cannot follow the third verification rule without it.
  *Example: `~/ESP-r_V13.3.18_Src`*

## This model

**FILL IN.** Record what the agent would otherwise have to derive for itself, and
keep it short enough to stay true as the model changes.

- Where the model was copied from, and the zone names.
  *Example: copied from `/opt/esp-r/training/simple`. One zone, named `reception`.*
- The config file, and how many zones.
  *Example: `cfg/bld_simple.cfg`. One zone.*
- Constructions and glazing, named as they appear in the files.
  *Example: `extern_wall`, `roof_1`, `floor_1`, `doors` and `partition`, plus two
  windows, `glz_s` facing south and `east_glz` facing east, both `dbl_glz`.*
- The control file: setpoints, schedule, and the stated plant capacity.
  *Example: `ctl/bld_simple.ctl`. Heating only, convective, ideal control. 15 °C
  from 07h and 20 °C from 09h, maximum 3 kW. Cooling setpoint 100 °C with 0 W,
  so no cooling occurs.*
- Shading and PV. State these explicitly when they are absent, and not only when
  they are present. An absence cannot be read from a file, so an agent that is
  not told will assume one way or the other.
  *Example: no shading (`*shad_calc none`, `*insol_calc none`). No PV.*
- The climate file, the site latitude and longitude, and whether the two agree.
  A disagreement between them is a fault an agent can detect, so recording that
  they agree tells it what normal looks like on this model.
  *Example: `clm67`, whose header latitude is 52.00, against site
  `*latlong 51.700 -0.500`. The two agree to about 0.3 degrees.*
- Which databases the model uses (`*stdmat`, `*stdmlc` and the rest).
  *Example: the standard global databases, `*stdmat material.db` and
  `*stdmlc multicon.db`.*
- Baseline results, if a baseline exists. Read the fourth verification rule below
  before relying on one.
  *Example: full year with three start-up days, heating 838.21 kWh over 1001
  hours and no cooling. Re-derive this before comparing anything against it.*

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
