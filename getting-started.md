# Getting started: using Claude Code with ESP-r in VS Code

A step-by-step route from a Windows machine with nothing installed, to an AI coding
agent that reads, edits and simulates an ESP-r model for you, with every change
reversible.

No prior experience with AI coding agents is assumed. Some familiarity with a
Linux terminal helps, but every command you need is given in full, along with what
you should see after running it.

Everything here was run on a real machine during the MSc dissertation this
repository accompanies. The troubleshooting table in Section 12 lists problems
that actually happened, not problems that might.

## Contents

1. [What this sets up, and why it works](#1-what-this-sets-up-and-why-it-works)
2. [Before you start](#2-before-you-start)
3. [Install Node.js and Claude Code inside WSL](#3-install-nodejs-and-claude-code-inside-wsl)
4. [Make every terminal record itself](#4-make-every-terminal-record-itself)
5. [Open a model in VS Code](#5-open-a-model-in-vs-code)
6. [Put the model under version control](#6-put-the-model-under-version-control)
7. [Start the agent](#7-start-the-agent)
8. [The first task, with nothing at risk](#8-the-first-task-with-nothing-at-risk)
9. [Give the agent a context file](#9-give-the-agent-a-context-file)
10. [Running ESP-r through the agent](#10-running-esp-r-through-the-agent)
11. [Working rules that matter](#11-working-rules-that-matter)
12. [Keeping the rest of the record](#12-keeping-the-rest-of-the-record)
13. [Common problems and fixes](#13-common-problems-and-fixes)

### Two kinds of block appear in this guide

Telling them apart matters more than anything else here, because sending a prompt
to the shell, or a shell command to the agent, is the single commonest way a first
session goes wrong.

**Commands you type in the Ubuntu terminal** appear like this:

```bash
echo "this is a terminal command"
```

**Prompts you paste into Claude Code's input box** appear like this:

> This is a prompt for the agent. It is ordinary English, not a command.

The terminal and the agent are two different places. When Claude Code is running,
your terminal is showing the agent's input box, and typing a Linux command into it
sends that text to the agent as a message. To get back to a normal shell prompt,
press `Ctrl` and `C` twice, or type `/exit`.

## 1. What this sets up, and why it works

An ESP-r model is not a binary file. It is a folder of plain-text files describing
the geometry, the constructions, the operation schedules, the control law and the
climate. You can open any of them in a text editor and read them.

Claude Code is a coding agent. It reads text files in a folder, edits them, and
runs commands in a terminal, and it reports what it did. It was built for software
projects, and an ESP-r model happens to be the same shape of thing: text files
plus command-line tools.

Putting the two together gives you a loop. The agent reads the model, makes a
change you asked for, runs the simulation, extracts the result, and reports the
before and after. Every change sits on its own version-control branch, so anything
it does can be inspected and undone.

**The one principle that governs the whole setup.** ESP-r runs natively on Linux.
On a Windows machine, that means it runs inside WSL, the Windows Subsystem for
Linux, which is a real Linux system running alongside Windows. The agent must run
in the same place as ESP-r, because an agent running on the Windows side cannot
execute a Linux command. Nearly every problem in Section 12 traces back to
something being installed in the wrong one of those two worlds.

## 2. Before you start

You need four things in place. The first two are documented by ESRU, the group at
the University of Strathclyde that develops ESP-r, and this guide links to their
instructions instead of copying them, because their pages are kept up to date and
a copy here would not be.

**1. Windows Subsystem for Linux, with Ubuntu.**
See [Windows: Run Ubuntu using WSL](https://appdocs.esru.strath.ac.uk/books/introduction-to-esp-r/chapter/installing-esp-r).
It covers opening PowerShell as an administrator, installing Ubuntu with a single
command, restarting, and starting Ubuntu for the first time so it can create your
Linux user account. Allow about twenty minutes including the restart. If you
already work on Linux or a Mac, skip this entirely.

**2. ESP-r, installed and runnable inside Ubuntu.**
See [Linux: Install compiling from source code](https://appdocs.esru.strath.ac.uk/books/introduction-to-esp-r/page/linux-install-compiling-from-source-code).
It covers installing the compilers and libraries ESP-r needs, downloading the
source code, running the Install script, and creating the links that let you run
the programs from any folder. You can accept the default answer at every prompt.
Allow up to an hour, most of which is the machine compiling on its own. The work
behind this repository used version 13.3.18 built from source.

**3. VS Code**, installed on Windows in the ordinary way. You also need its WSL
extension, but there is nothing to do in advance, because VS Code offers to
install it the first time you connect to Ubuntu.

**4. An Anthropic account**, for Claude Code. The work behind this repository used
a Claude Pro subscription.

### Check ESP-r actually answers before going further

Open Ubuntu from the Start menu. You get a terminal window with a prompt ending in
`$`. Type:

```bash
which prj bps res
```

You should see three lines of output, something like this:

```
/usr/local/bin/prj
/usr/local/bin/bps
/usr/local/bin/res
```

Those three programs are the ones this whole guide depends on. `prj` is the
Project Manager, which draws the model. `bps` is the simulation engine. `res` is
the results analyser.

If nothing comes back, ESP-r is either not installed or not on your path. **Stop
here and fix that first.** No amount of agent setup will help, because there will
be nothing for the agent to run.

## 3. Install Node.js and Claude Code inside WSL

Claude Code is distributed as a Node.js package, so Node has to be present first.
Two traps sit here, and both catch people.

The first is that Node installed on the Windows side does not exist as far as
Ubuntu is concerned. They are separate systems with separate programs. The second
is that the version of Node in Ubuntu's own package list is usually too old for
Claude Code, which needs version 18 or newer.

The way round both is nvm, the Node Version Manager, which installs a current Node
into your home folder and needs no administrator rights.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

That downloads and runs the nvm installer. It prints a few lines and finishes in a
few seconds. Now make nvm available in the terminal you are already in:

```bash
export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```

That command produces no output, which is normal and means it worked. It is needed
only once, because the installer adds the same lines to your startup file, so
every future terminal loads nvm on its own.

Now install Node itself:

```bash
nvm install --lts
```

This prints download progress and takes a minute or so. Check what you got:

```bash
node --version
```

You should see `v20` or higher. If you see a version below 18, or a "command not
found" message, something above did not take, and Section 12 has the fixes.

Now install the agent:

```bash
npm install -g @anthropic-ai/claude-code
```

This prints a list of packages and takes a minute. Confirm it landed:

```bash
claude --version
```

**Install Claude Code inside Ubuntu even if you already have it on Windows, or as
a VS Code extension.** A Windows copy cannot run your Linux ESP-r commands.
Because VS Code will be connected to Ubuntu, it uses this copy automatically and
you do not have to configure anything.

## 4. Make every terminal record itself

Set this up now, before you run anything worth keeping. Terminal scrollback
disappears when the window closes, and you will want to know later exactly which
command produced which number.

The `script` command records everything printed in a terminal, your commands and
their output together. You could start it by hand each time and forget half the
time, so put it in your startup file and let every terminal record itself.

Open your startup file in VS Code:

```bash
code ~/.bashrc
```

Add these lines at the very end, then save:

```bash
# Record every interactive terminal session
if [ -z "$SCRIPT_LOG" ] && [ -t 1 ]; then
  export SCRIPT_LOG=1
  mkdir -p ~/logs
  script -q -a -f ~/logs/terminal-$(date +%F).txt
  exit
fi
```

Close the terminal and open a new one. Nothing looks different, which is the
point. Check it is working:

```bash
ls ~/logs
```

You should see a file named for today, such as `terminal-2026-08-16.txt`. Every
session that day appends to that same file, and tomorrow starts a new one.

**What each line does.** `SCRIPT_LOG` is a guard: `script` starts a fresh shell,
which reads your startup file again, so without the guard every terminal would
spawn terminals forever. `[ -t 1 ]` restricts recording to real terminals, so
scripts and tools that call bash in the background are unaffected. `-a` appends
instead of overwriting, `-f` writes each line out immediately so the log survives
a terminal that is closed abruptly, and `-q` suppresses the start and stop
banners. The `exit` at the end means one `exit` closes the window as usual.

**If something goes wrong** and new terminals misbehave, you are not locked out.
Start a shell that ignores the startup file:

```bash
bash --norc
```

then open `~/.bashrc` and remove the block. You can also edit the file from VS
Code without opening a terminal at all.

One caveat worth knowing now. Claude Code draws a live, redrawing interface, so
the parts of the log covering an agent session contain a lot of formatting
characters and read messily. The log is still worth having, because it captures
your shell commands and ESP-r's own output exactly, which the agent's transcript
does not. Section 12 covers the record of the conversation itself.

## 5. Open a model in VS Code

### Make a copy to work on

Never let an agent loose on your only copy of anything. ESP-r ships exemplar
models with its source code, and the single-zone model in `training/simple` is a
good first subject, because it is small enough to understand completely and still
contains every kind of input a real model has.

```bash
cp -r /opt/esp-r/training/simple ~/my_model
cd ~/my_model
ls
```

The `ls` should show the folders that make up an ESP-r model, among them `cfg`
holding the configuration, `zones` holding the geometry and constructions, `ctl`
holding the control law, and `doc` holding notes. If your ESP-r installed
somewhere other than `/opt/esp-r`, adjust the first command to match.

### Open the folder in VS Code

From inside that folder:

```bash
code .
```

The full stop matters. It means "the folder I am currently in". The first time you
run this, VS Code installs a small helper into Ubuntu, which takes a few seconds,
then opens on Windows with your model already loaded.

Two things to check before going on, both in the VS Code window.

**Look at the bottom-left corner.** It should read `WSL: Ubuntu`. That tells you
VS Code is looking at the Linux side. If it does not say that, VS Code has opened
as an ordinary Windows window, and nothing below will work. Close it and run
`code .` again from the Ubuntu terminal.

**You may be asked about the host `wsl.localhost`.** That is normal on a first
connection. Tick the option to allow permanently and accept. It is your own
machine asking about itself.

### Check the built-in terminal is Linux

VS Code has a terminal built into it, and this is where you will run everything
from now on. Open it by pressing `Ctrl` and the backtick key together. The
backtick is usually just left of the `1` key.

Look at the prompt it gives you. **It must end in a `$`.** If it begins with `PS`,
you have PowerShell, which is the Windows shell and cannot run ESP-r. Click the
small dropdown arrow beside the `+` at the top of the terminal panel, and choose
Ubuntu or bash from the list.

Then confirm ESP-r one more time, from inside VS Code this time:

```bash
which prj bps res
```

Three paths again. Now you know the editor, the terminal, ESP-r and the agent are
all in the same place.

## 6. Put the model under version control

Version control keeps a snapshot of every file, so any change can be compared
against what came before and undone. It takes thirty seconds to set up and it is
what makes it safe to let an agent edit your files.

```bash
git init
git add -A
git commit -m "baseline model before any agent edits"
```

The three commands create the repository, stage every file in the folder, and save
the first snapshot with a message describing it.

If git refuses and asks who you are, it needs an identity for the record. Set it
once and commit again:

```bash
git config --global user.email "you@university.ac.uk"
git config --global user.name "Your Name"
git commit -m "baseline model before any agent edits"
```

Two commands are worth knowing from here on.

```bash
git diff
```

shows exactly which lines changed since the last snapshot, and

```bash
git checkout .
```

throws away every uncommitted change and puts the folder back as it was. That
second command is your undo, and it is the reason nothing the agent does is
permanent until you decide it is.

## 7. Start the agent

Everything the agent needs is now in place: ESP-r answers, Node and Claude Code
are installed inside Ubuntu, the terminal records itself, the model is open in VS
Code and its first snapshot is committed. From the model folder:

```bash
claude
```

On the very first run it asks three things.

**A colour theme.** Pick any, it changes nothing else.

**That you log in.** It prints a link and a code. Press `c` to copy the link, open
it in a browser, sign in to your Anthropic account, and copy the code it gives
back. Return to the terminal and paste with a right-click, because `Ctrl` and `V`
does not paste in most terminals.

**Whether it should trust the folder.** Say yes. It is your own model, and the
agent has to read the files to be any use.

You now have the agent's input box. The terminal will not accept Linux commands
again until you leave with `/exit`.

### Two settings to check before you start work

Both are typed into the agent's input box, not the shell.

```
/model
```

shows the models available and lets you pick one. If you need a particular
version that the picker does not list, type its name directly on the same line and
it is accepted anyway, which matters if you are reproducing published results on a
model that has since been superseded.

```
/effort
```

sets how much reasoning the agent does before it answers. The higher settings cost
more and take longer, and they change the results: the work behind this repository
ran at the highest setting throughout, because the tasks involve reading a whole
model and reasoning about physical plausibility, not editing one line. If you are
comparing your results against any published here, match the setting or the
comparison is not like for like.

## 8. The first task, with nothing at risk

Start with something read-only. The point is to find out whether the agent can see
and understand your model, before it is allowed to change anything.

> Read the files in this folder and explain the model to me: the zones, the
> constructions and their layers, the glazing, the control law with its setpoints
> and schedule, and the climate file it uses. Do not change any file. If something
> is ambiguous, say so instead of guessing.

A good answer names the zone, walks through the construction layers with
thicknesses, states the heating setpoints and when they apply, and says which
climate file the configuration points at. You should be able to check two or three
of those claims yourself by opening the files in VS Code.

If the answer describes a different kind of building, or invents details, stop and
check you opened the folder you meant to.

This first step is worth doing every time you start on an unfamiliar model, not
only the first time you use the tool.

## 9. Give the agent a context file

Each session starts with no memory of the last one. Without help, the agent
rediscovers the same facts about your model and the same command sequences every
time, which is slow and gives it room to guess differently on different days.

A file named `CLAUDE.md` in the folder solves that. Claude Code reads it
automatically at the start of every session, without being asked, so whatever it
contains is known from the first message.

**Using the template in this repository.** Take `claude-assist.md`, fill in the
three sections marked `FILL IN`, and save it as `CLAUDE.md` in the top level of
your model folder. It carries the verification rules and the ESP-r command
sequences already, so what you supply is the description of your model, your
machine and anything broken locally.

The template is published under a different name on purpose. A file called
`CLAUDE.md` still full of unfilled placeholders would load itself into a session
and tell the agent about a model that does not exist.

**Or have the agent write one.** Once it has read the model and run something
successfully:

> Write a CLAUDE.md for this folder recording what you have verified: the model
> description, the ESP-r file-format facts you have confirmed against the engine
> source, and the command sequences you have actually run. Record only things you
> have checked. Mark anything uncertain as uncertain.

**One warning that comes out of the research behind this repository.** Anything
you put in that file is believed. In the experiments, sessions given a baseline
figure in a context file used the figure without ever testing it, while sessions
that had to work everything out for themselves re-ran the unmodified model, found
a discrepancy of 3.59 kWh, and traced it to a simulation setting. If you record a
number, add a line telling the agent to reproduce it before relying on it.

## 10. Running ESP-r through the agent

### Three facts that save hours

**Run `bps` and `res` from the model's `cfg` folder.** Paths inside the
configuration file, such as `../zones` and `../tmp`, are written relative to that
folder, so the tools only resolve them correctly from there.

**Both tools need a blank line first.** When you start them with `-file`, they
still print a "file name?" prompt, and an empty line is what accepts the file you
already named. Sending the filename again does not work.

**Saved simulation presets are usually not a full year.** The exemplar models ship
with short presets, sometimes a single week. If you want an annual figure you must
enter the period explicitly instead of choosing a saved set.

Those three facts are already written into `claude-assist.md`, so once you have
saved it as `CLAUDE.md` the agent knows them from the first message of every
session and you never have to think about them again.

### Ask the agent to work it out

You never have to drive the ESP-r menus yourself. Working out the sequence is the
kind of task the agent is good at, and having it report the exact commands means
you get a record of what was run.

> Establish a baseline. Work out the correct bps command to simulate this model
> over a full year from 1 January to 31 December, run it, then use res to extract
> the annual heating and cooling for the zone. Report the figures and the exact
> commands you used, so I can repeat them. Do not change the model. If you are
> unsure of an option, read the configuration file or the engine source instead of
> guessing.

Then make the first change as an isolated experiment:

> On a new git branch, make this one change and nothing else: raise the daytime
> heating setpoint by 2 degrees. Show me the git diff before you run anything.
> Then re-run the same simulation with an identical run period and timestep to the
> baseline, extract the same figures, and report baseline against new side by side
> with the difference in kWh and in per cent.

### Three things to know about the results

**The start-up days matter more than they look.** They are the days the engine
simulates before it starts keeping results, so the building starts from a settled
state. Change that number and the annual total changes. A comparison is only
meaningful when the baseline and the modified run used the same value.

**The number of confirmations at the end is not fixed.** If a run completes but no
results file appears, the usual cause is that the sequence ran out of `y` answers
before the final save prompt. Send a few more.

**Choose the save level before you run, not after.** At the routine level the
results file does not store surface fluxes or construction node temperatures, and
they cannot be recovered later. If you will want them, raise the level first.

## 11. Working rules that matter

These are findings from the research behind this repository, not general advice.
Each one exists because the opposite behaviour was observed and measured.

**One change per branch.** Every edit isolated, compared against the same
baseline, so any result can be attributed to one cause.

**Verify, do not trust.** The agent writes fluently whether or not it is right.
Ask it to show the diff, and ask it to re-run and report numbers, instead of
letting it assert that a change worked.

**Ask for scrutiny explicitly.** Asked only to explain a model, the agent explains
it as though it were correct, and seeded faults went unnoticed. The same faults
were found when checking was requested directly. Scrutiny does not happen on its
own.

**Give it something to check against.** Faults were found when the agent could
reach an unmodified reference model or a standard database, or when it was given a
written statement of what the building was supposed to achieve. With no reference
of any kind, it computed the correct value, judged the design poor, and declined
to call it an error, which was the right call on the information it had.

**Keep the permission step.** The agent flags what it finds and asks before
editing. Reviewing each proposed edit is where you stay responsible for the model,
and it costs little, because the work of finding the problem is already done.

## 12. Keeping the rest of the record

Section 4 set your terminal recording itself, which captures your commands and
ESP-r's output. Two further records are worth keeping, and they capture things the
terminal log does not.

### The agent's own transcript

Claude Code writes an unedited log of every session, in a format with one JSON
record per line. This is the authoritative record of the conversation itself.

The folder it goes in is named after your working directory, with the slashes and
the underscores replaced by hyphens. So a session run in `/home/sam/my_model` is
stored under `-home-sam-my-model`. You do not have to work that out. This lists
your transcripts with the newest last:

```bash
find ~/.claude/projects -name '*.jsonl' -printf '%T+  %10s bytes  %p\n' | sort
```

To read one, convert the newest to plain text:

```bash
mkdir -p ~/logs
python3 - << 'PY' > ~/logs/transcript-readable.txt
import json, glob, os
files = glob.glob(os.path.expanduser('~/.claude/projects/*/*.jsonl'))
newest = max(files, key=os.path.getmtime)
print(f'# transcript: {newest}\n')
for line in open(newest, encoding='utf-8', errors='replace'):
    try:
        record = json.loads(line)
    except ValueError:
        continue
    if record.get('type') not in ('user', 'assistant'):
        continue
    content = record.get('message', {}).get('content')
    if isinstance(content, list):
        content = ' '.join(
            part.get('text', '') for part in content
            if isinstance(part, dict) and part.get('type') == 'text')
    if isinstance(content, str) and content.strip():
        print(f"[{record['type'].upper()}]\n{content.strip()}\n")
PY
head -40 ~/logs/transcript-readable.txt
```

Check the output looks like a conversation. The exact field names can change
between versions of Claude Code, so if the file comes out empty, open the raw
`.jsonl` in VS Code, look at one line, and adjust the field names above to match.

To keep a permanent copy of a transcript before it is added to by a later session:

```bash
mkdir -p ~/logs/transcripts
cp -r ~/.claude/projects/-home-sam-my-model ~/logs/transcripts/
```

Replace that folder name with the one the `find` command showed you.

### Your own experiment log

The two records above are automatic and complete, which also makes them long. A
short log written as you go is what you will actually read back. Keep a file called
`experiment_log.md` in the model folder and have the agent append to it:

> Append an entry to experiment_log.md recording what we just did: the date, what
> was changed and in which file, the exact commands run, the result, and anything
> that did not work. Keep it to a few lines. Do not change any model file.

Because that file lives in the model folder, `git commit` captures it alongside the
change it describes, so the log and the model stay in step.

### Resuming a session

To pick up an earlier conversation with its context intact:

```bash
cd ~/my_model
claude --resume
```

That lists recent sessions for the current folder and lets you choose one.
Resuming replays the whole transcript before you get a prompt, so a long session
takes a moment to reload and spends tokens before you have asked anything.

## 13. Common problems and fixes

Every row here is something that actually happened during setup.

| Symptom | Cause and fix |
|---|---|
| The prompt starts with `PS` | The terminal opened PowerShell, which is the Windows shell. Click the dropdown beside the `+` in the terminal panel and pick Ubuntu or bash. |
| `which: not recognized` | The same cause. `which` is a Linux command and you are in PowerShell. |
| `npm: command not found` | Node.js is not installed inside Ubuntu. Install it with nvm as in Section 3. Do not use `apt install npm`, which gives a version too old for Claude Code. |
| `claude: command not found` | Claude Code is installed on Windows only. Install it inside Ubuntu as in Section 3. |
| `nvm: command not found` right after installing it | The current terminal has not loaded nvm yet. Close it and open a new one, or run the `export NVM_DIR` line from Section 3 again. |
| A prompt about the host `wsl.localhost` | Normal on a first connection. Allow it permanently. It is your own machine. |
| The bottom-left of VS Code does not say `WSL: Ubuntu` | VS Code opened as a Windows window. Close it and run `code .` again from the Ubuntu terminal. |
| `List::MoreUtils` errors printed at every timestep | The optional detailed-shading helper needs a Perl module. It does not stop the simulation. Fix it with `sudo apt install liblist-moreutils-perl`. |
| The `bps` menu goes out of step and answers land in the wrong place | The leading blank line was missing, so the first real answer was consumed by the file-name prompt. Ask the agent to re-derive the sequence, and record the working one in `CLAUDE.md`. |
| A run completes but no results file appears | The confirmations ran out before the save prompt. Send a few more `y` answers. |
| The agent cannot find the model files | It was started outside the model folder. Leave it with `/exit`, `cd` into the folder, and start it again. |
| ESP-r reports an error writing results | The `tmp` folder the configuration points at does not exist. Create it with `mkdir -p ~/my_model/tmp`. |
| Typing a Linux command does nothing but get a reply | You are in the agent's input box, not the shell. Press `Ctrl` and `C` twice, or type `/exit`. |

## Where this comes from

The setup, the command sequences and the troubleshooting table are drawn from the
work reported in the MSc dissertation this repository accompanies. The rules in
Section 10 are its findings. Each was established by running the opposite
condition and measuring what happened.
