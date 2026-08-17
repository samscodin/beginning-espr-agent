ESP-r agent context file

A template CLAUDE.md for using an LLM coding agent, such as Claude Code, on an ESP-r building energy model through the model's own text files.

What it is for

An agent dropped into an ESP-r model directory has to work out the file formats, the command-line sequences and the conventions of the installation before it can do anything useful, and it will often guess instead. A context file hands it that knowledge up front. A file named CLAUDE.md is read from the working directory automatically, without anyone asking for it.

What is in it

Verification rules. Commit before editing, re-run the simulation after any edit and check the numbers are sensible, ground any claim about file formats in the documentation or the source and not in keyword names, and treat a recorded baseline as unverified until the agent has reproduced it.

Command-line sequences. The keystrokes for driving bps and res in text mode, including the leading blank line both tools need at the file prompt, a full-year run, heating and cooling totals, solar gains and the zone energy balance. Each one carries the trap that goes with it, such as solar gains being reported as a mean power and not as an energy total.

Sections marked FILL IN. The description of the model, of the machine, and of anything broken locally. Those describe one model on one installation and have to be rewritten for another. Each section lists what to record, so a new user is told what the agent needs instead of facing a blank heading.

Where it comes from

The template was derived from the context file used in an MSc dissertation at the University of Strathclyde, which tested whether a general-purpose LLM coding agent can modify and scrutinise ESP-r models by working on their text files.

The template is not the file those experiments used. The tested file is reproduced verbatim in Appendix 1 of the dissertation, and the results reported there attach to that file. The template differs from the tested file in two ways. The parts describing one model and one machine have been replaced by placeholders. Three rules have also been added that came out of the findings: that a supplied baseline must be re-derived before anything is compared against it, that the number of prompts at the end of a simulation run is not fixed, and that the save level chosen before a run decides what the results file can answer afterwards.

Using it

Copy CLAUDE.md into the root of your model directory, fill in the three marked sections, and start the agent from there.
