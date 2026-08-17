# The guide for the context file, claude-assist.md

# Using an LLM coding agent with ESP-r

Two files that let you point a coding agent, such as Claude Code, at an ESP-r
building energy model and have it read, edit and simulate the model by working on
its plain-text files.

## What is here

**[`getting-started.md`](getting-started.md)** is a step-by-step tutorial, from a
Windows machine with nothing installed to an agent running annual simulations and
reporting results. It assumes no experience with coding agents. It covers the WSL
and ESP-r prerequisites, installing Node and Claude Code in the right place,
setting your terminal to record every session, version control as an undo, the
first read-only task, and a troubleshooting table of problems that actually
happened.

**[`claude-assist.md`](claude-assist.md)** is a template context file. Fill in the
three marked sections and save it as `CLAUDE.md` in your model folder.

Start with the tutorial if any of this is new. Take the template on its own if you
already run agents and only want the ESP-r specifics.

## What the context file is for

An agent dropped into an ESP-r model folder has to work out the file formats, the
command-line sequences and the conventions of the installation before it can do
anything useful, and it will often guess instead. A context file hands it that
knowledge up front. A file named `CLAUDE.md` is read from the working directory
automatically, without anyone asking for it.

The template is published under a different name on purpose. A file called
`CLAUDE.md` still carrying its unfilled placeholders would load itself into a
session and describe a model that does not exist.

## What is in it

**Verification rules.** Commit before editing. Re-run the simulation after any
edit and check the numbers are sensible. Ground any claim about file formats in
the documentation or the source and not in keyword names. Treat a recorded
baseline as unverified until the agent has reproduced it.

**Command-line sequences.** The keystrokes for driving `bps` and `res` in text
mode: the leading blank line both tools need at the file prompt, a full-year run,
heating and cooling totals, solar gains and the zone energy balance. Each one
carries the trap that goes with it, such as solar gains being reported as a mean
power and not as an energy total.

**Sections marked FILL IN.** The description of the model, of the machine, and of
anything broken locally. Those describe one model on one installation and have to
be rewritten for another. Every bullet says what to record and shows a worked
example underneath, so you have a pattern to copy instead of a blank heading.
Delete the examples once you have replaced them.

## Where it comes from

Both files were derived from the setup and the context file used in an MSc
dissertation at the University of Strathclyde, which tested whether a
general-purpose LLM coding agent can modify and scrutinise ESP-r models by working
on their text files.

**The template is not the file those experiments used.** The tested file is
reproduced verbatim in Appendix 1 of the dissertation, and the results reported
there attach to that file. The template differs from it in two ways. The parts
describing one model and one machine have been replaced by placeholders and
examples. Three rules have also been added that came out of the findings: that a
supplied baseline must be re-derived before anything is compared against it, that
the number of prompts at the end of a simulation run is not fixed, and that the
save level chosen before a run decides what the results file can answer
afterwards.

## ESP-r itself

ESP-r is developed by ESRU at the University of Strathclyde. Installing it is
covered by their own documentation, which this repository links to and does not
duplicate:
[Introduction to ESP-r](https://appdocs.esru.strath.ac.uk/books/introduction-to-esp-r).
Nothing here replaces it. What is here starts where their instructions end.
