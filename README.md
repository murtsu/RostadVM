# RostadVM

**This project is young. It is a work in progress. It will stay that way for a while.**

If you have views, share them.
Every serious opinion will be considered.
That is not a courtesy line. It is how good software gets built.

There is a cookbook in this repo explaining how the agent messaging works.

---

## A word from Eddie, on account of the silence

Eddie. Marko asked me to explain the radio silence, and being a professional, I said yes before I knew what I was explaining.

Here is what happened. Fifteen AI agents needed contracts before they were allowed to write a single line of Rust. Ninety-two contract faults surfaced before that Rust existed. A tool had to be built just so a human could read and patch JSON messages without losing his mind. None of that makes a good excuse for silence, but all of it is true, which is the best kind of excuse.

So: sorry for the quiet. The seating chart is done. The wedding starts in August.

— Eddie

---

## What this is

A virtual machine framework that does one thing unusually well.

If you destroy your system.
If a patch eats your environment.
If someone runs the wrong command.
If you run the wrong command.

You rebuild the entire system drive to its original state.

In five seconds.

The mechanism is a copy-on-write architecture kept deliberately simple. It is patent pending.

It runs on any Linux system with Libvirt support.
No custom kernel.
No proprietary dependencies.
No 400-page manual.

The interface is point and click.
It was point and click in the prototype.
It will be point and click in the production version, with a better interface.

---

## Current state

The prototype was written in Bash. It worked. It proved the idea, and it is now the ceiling — which is exactly what a good prototype is supposed to become.

The rewrite is in Rust.

The agent organisation that is building it is fully staffed: **15 of 15 roles are live** — PM, PPM, SD, CR, QA, SPM, TM, MTM, SEC, UX, DOC, TEST, INT, CMVC, INFRA. Every role has a definition file, a schema, and a place in the message protocol. `CLAUDE.md`, the project constitution, is at **v2.10**.

The project is now forkable with a single line edit at the top of `CLAUDE.md` — the organisation no longer hard-codes a name where it should have hard-coded a role.

A **contract_editor** has also been delivered: a small Ratatui TUI for reading and patching the agents' inbox, outbox, and state files by hand, nano-style keys and bash-style tab completion included. It never touches the source files directly — it writes RFC 6902 JSON patches instead, so nothing gets silently mutated. It builds clean on current stable Rust.

---

## Contract Coding

This project is governed by a methodology called **Contract Coding**: contracts and specifications are written before any production code, hard gates separate phases, and the people building are structurally separated from the people reviewing.

The thesis is simple. Waterfall was never the problem. The cost of discovering a mistake late was the problem. AI agents reintroduce that cost, because an unsure agent guesses instead of asking — so the discipline of contracts-before-code, once optional, is now load-bearing.

The proof point: **92 contract faults were found and fixed before a single line of production Rust was written.**

The full theoretical case is written up as a white paper, open access, CC-BY-4.0: [DOI 10.5281/zenodo.20689255](https://doi.org/10.5281/zenodo.20689255).

Community contributor **Malome Tebatsos** has been an active architectural reviewer on this project — catching a missing INFRA triage protocol and a structural boot-sequencing fault among other things. His fixes are credited formally in the project history, not buried in a commit message.

---

## Test running starts in August

The contracts are complete enough. The agents are registered. The next milestone is the first end-to-end run of the core delivery loop — PM receives a request, the organisation delivers, nobody improvises past a checkpoint.

That test run starts in **August**.

---

## Where this comes from

I spent years at Bofors Electronics building C3I systems.

Command, Control, Communications, and Intelligence.

These are not systems where you ship something broken and fix it later. They are systems where the architecture has to be correct before a line of code is written. Where every interface is specified. Where every role knows exactly what it is responsible for and what it is not. Where quality is measured, not felt.

That experience is the foundation of how this project is structured.

I also worked at Nokia Information Systems and Versal Data, and across several other industries over a career that started in the 1980s.

The common thread across all of it: the projects that failed did not fail because the engineers were not smart enough. They failed because nobody had clearly defined who owned what, what done meant, and what happened when something went wrong.

This project is structured so that those failures are harder to make.

---

## Why open source

I believe software is an artform.

I spent most of my career building closed source systems for organisations that owned the output completely. That work taught me a great deal. But the knowledge stayed inside those organisations.

This project is different.

Everything here is free to use, study, modify, and build on. If you learn from it, good. If you build something better from it, even better. The more people making things, the better the world gets.

That is not idealism. That is just how art works.

---

## Who is building this

My name is Marko Tahvanainen.

I am an AI consultant based near Stockholm. I have been building software since the 1980s. I have Asperger's, which explains a few things about how I think about systems.

I am building this with one human and fifteen AI roles, plus a growing number of people like Malome who show up, read the contracts, and find the fault nobody else caught.

The question I started with was whether any of this is even possible.

The answer is still being written.

---

## Contributing

This project is in early development.

If you have experience with Libvirt, Rust, copy-on-write filesystems, or systems engineering at scale, your input is welcome.

If you have opinions about the org structure design, the agent architecture, or the VM mechanism, open an issue and say so clearly.

If you find a bug in the prototype, report it with enough detail to reproduce it.

If you want to contribute code, read the Contract Coding white paper and `CLAUDE.md` first. They explain why things are designed the way they are.

---

## Status

| Component | Status |
|---|---|
| Bash prototype | Working |
| Rust rewrite | In progress |
| Agent org structure | 15/15 roles live, CLAUDE.md v2.10 |
| contract_editor (TUI) | Delivered |
| End-to-end test run | Starts August |
| Contract Coding white paper | Published, Zenodo |
| Desktop UI | Designed, not yet built |
| Documentation | In progress |

---

github.com/murtsu/RostadVM

Apache 2.0. The seating chart is open. Anyone can check the adjacencies.

If you find a conflict I missed, open an issue.

The duck approves.

*Views, questions, and contributions welcome.*
*This is young. Help make it less young.*
