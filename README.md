# RostadVM

**This project is young. It is a work in progress. It will stay that way for a while.**

If you have views, share them.
Every serious opinion will be considered.
That is not a courtesy line. It is how good software gets built.
Update: For those really interested theres a cookbook up explaining how to use this.
---

## What this is

A virtual machine framework that does one thing unusually well.

If you destroy your system.
If a patch eats your environment.
If someone runs the wrong command.
If you run the wrong command.

You rebuild the entire system drive to its original state.

In five seconds.

The mechanism is a copy-on-write architecture kept deliberately simple.
It does not have a name yet.
It is patent pending.

It runs on any Linux system with Libvirt support.
No custom kernel.
No proprietary dependencies.
No 400-page manual.

The interface is point and click.
It was point and click in the prototype.
It will be point and click in the production version, with a better interface.

---

## Current state

The prototype is written in Bash.

Yes, Bash.

It works.
Bash proved the idea and is now the ceiling.

The rewrite is in Rust.
I am learning Rust while building it.
Which is either the correct way to learn a language or a sign I need more sleep.

The multi-agent AI engineering organisation that is building it is described below.
The org structure document is in this repository.
It is 32 pages.
It is detailed on purpose.

---

## How it is being built

This project uses a structured multi-agent AI system running inside Claude Code.

Not one AI assistant.
A designed organisation of fifteen roles.

Each role has defined authority, defined constraints, defined inputs, and defined outputs.
There is a Project Manager, a Software Designer, a PPM, multiple Sub System Managers, Tool Makers at subsystem and master level, Code Reviewers, Security, UI/UX, Documenters, Testers, Integrators, a Quality Manager, CMVC, and Infrastructure.

The coder and the reviewer are never the same agent.
The Quality Manager defines what done means before anyone starts.
The Software Designer cannot release specifications until the human has signed off on a reflection document confirming the analysis was understood correctly.

This structure comes from experience building complex systems with real teams.
It has been translated into an AI agent organisation because the same problems that make human teams fail also make single-agent AI systems fail.

The full role definitions, authority maps, and agent implementation notes are in `ORG_STRUCTURE.md`.

---

## Where this comes from

I spent years at Bofors Electronics building C3I systems.

Command, Control, Communications, and Intelligence.

These are not systems where you ship something broken and fix it later.
They are systems where the architecture has to be correct before a line of code is written.
Where every interface is specified.
Where every role knows exactly what it is responsible for and what it is not.
Where quality is measured, not felt.

That experience is the foundation of how this project is structured.

I also worked at Nokia Information Systems and VERSAL DATA, and across several other industries over a career that started in the 1980s.

The common thread across all of it: the projects that failed did not fail because the engineers were not smart enough. They failed because nobody had clearly defined who owned what, what done meant, and what happened when something went wrong.

This project is structured so that those failures are harder to make.

---

## Why open source

I believe software is an artform.

I spent most of my career building closed source systems for organisations that owned the output completely.
That work taught me a great deal.
But the knowledge stayed inside those organisations.

This project is different.

Everything here is free to use, study, modify, and build on.
If you learn from it, good.
If you build something better from it, even better.
The more people making things, the better the world gets.

That is not idealism.
That is just how art works.

---

## Who is building this

My name is Marko Tahvanainen.

I am an AI consultant based near Stockholm.
I have been building software since the 1980s.
I have Asperger's, which explains a few things about how I think about systems.

I am building this alone.
One human.
Fifteen AI roles.

The question I started with was whether that is even possible.

The answer is still being written.

---

## Contributing

This project is in early development.

If you have experience with Libvirt, Rust, copy-on-write filesystems, or systems engineering at scale, your input is welcome.

If you have opinions about the org structure design, the agent architecture, or the VM mechanism, open an issue and say so clearly.

If you find a bug in the prototype, report it with enough detail to reproduce it.

If you want to contribute code, read the org structure document first.
It will explain why things are designed the way they are.

---

## Status

| Component | Status |
|---|---|
| Bash prototype | Working |
| Rust rewrite | In progress |
| Agent org structure | Defined, v2.0 |
| Desktop UI | Designed, not yet built |
| Documentation | In progress |

---

*Views, questions, and contributions welcome.*
*This is young. Help make it less young.*
