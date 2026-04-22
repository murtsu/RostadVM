# RostadVM

**This project is young. It is a work in progress. It will stay that way for a while.**

If you have views, share them.
Every serious opinion will be considered.
That is not a courtesy line. It is how good software gets built.
Update: For those really interested there's a cookbook up explaining how to use this.
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

# Contract Coding, Part 1. Explained with a wedding thats not fat, greek or from hell. 
The dinner party metaphor I used last day was almost right.

The problem with dinner parties is that when they go wrong, you can try again next Saturday. 

A wedding is better. 

A wedding has one shot. The agents are loaded at 14:00 and they run until the last guest leaves. There is no second run. There is no staging
environment. There is no rollback. 

If the seating chart is wrong on the day, the seating chart is wrong on the day. 

# The table 

Picture a wedding reception. One hundred and twenty guests. A long head table for the wedding party. Eight round tables for everyone else.

Each table seats fifteen. 

The people who need to be seated include: 

The groom’s family, half of whom have not spoken to the other half since an argument about a will in 2019. 

The bride’s parents, who are divorced and have each brought a new partner, and who agreed in writing to be civil but whose history suggests
optimism. 

The best man, who dated the maid of honour for three years and ended it badly, and who is seated with the wedding party regardless because
protocol demands it. 

Four colleagues of the groom who know each other from work, speak only to each other at parties, and should under no circumstances be
placed near the grandfather who will ask each of them what they do for a living and then explain why it is not a real job. 

A table of university friends who have not seen each other in twelve years and will either have the best night of the decade or relitigate a
decade of unresolved grievances within forty minutes of the first glass. 

And the cousin. Who is always the cousin. Who does not need further description. 

This is your agent organisation. 


One hundred and twenty participants. Multiple incompatibilities. Defined roles for some, undefined expectations for others. Shared history that
creates both connection and risk. A strict sequence of events — arrival, drinks, seating, speeches, dinner, dancing — that must coordinate
across all participants simultaneously. 

Nobody is going to supervise this in real time. 

Not even the wedding planner. 

The wedding planner wrote the seating chart. 

# What the seating chart actually is 

It is a contract. 

It specifies who sits next to whom and, implicitly, what conversations are permitted to occur. 

It specifies who faces the head table and who does not — a hierarchy decision disguised as logistics. 

It specifies which tables receive which courses first — a sequencing decision that affects the timing of speeches. 

It specifies where the exits are relative to the grandfather, and where the bar is relative to the cousin. Both are non-trivial architectural
decisions. 

A good seating chart has been reviewed three times and has caught at least eleven things that would have ruined the evening. The divorced

parents cannot share a sightline. The best man and maid of honour must be physically separated during the speeches. The university friends

table must be near the dancing floor because they are unpredictable and should be allowed to move. 

A bad seating chart was done the night before the wedding because there was too much else to do. 
You can tell the difference by the third round of drinks. 

# The speeches are also contracts 


There is a protocol for speeches. Best man goes third. Father of the bride goes first. Someone from the groom’s side goes second. Each speech
has a rough duration and a defined tone.This protocol exists because an uncontracted speech situation produces one of two outcomes. 

Either everyone looks at each other waiting for someone to go first, which creates a silence that feels much longer than it is, until the most

confident person in the room takes over, which is not always the right person. 

Or everyone tries to go at once, which produces the verbal equivalent of a race condition. 

The speech protocol is a message schema. It defines senders, recipients, content guidelines, and sequencing. The best man is not permitted to

send a message of type “embarrassing_childhood_story” until after the father of the bride has sent a message of type “formal_welcome.” The

cousin is not in the approved sender list at all. 

This is CLAUDE.md. 

Where it goes wrong without contracts 

Here is a real failure mode from the world of weddings. 

Two tables are adjacent. Both tables include guests from the groom’s extended family. Nobody told table seven that table eight contains the

branch of the family they stopped speaking to. Nobody wrote this down because it felt awkward to write down. 

Both tables receive their starters at the same time. 

Someone at table seven makes eye contact with someone at table eight. 

The evening recovers. Barely. Not because the wedding planner intervened — the wedding planner is managing a catering crisis in the kitchen.

It recovers because one guest at table seven quietly de-escalates what is about to happen. 

The wedding planner got lucky. 

In a multi-agent system, there is no luck. There is no quiet guest who de-escalates. There is just the message that was sent and the message

that was not expected and the agent that does not know what to do with the message it received. 

The system stalls. 

The agents sit in their rooms holding notes they do not understand. 

The reception is technically still going but nobody is dancing. 

What contract coding produces at a wedding 

The wedding planner has done the following: 

She has specified every table’s composition with rationale. Not just “table six: these eight people” but “table six: colleagues from the bride’s

workplace who share the common denominator of not knowing anyone else at the wedding and will bond over this.” 

She has specified every adjacency that cannot happen, with explanation. “Bride’s father and bride’s stepfather: not adjacent, not sightline, not

same table.” 

She has specified the speech sequence, the timing, the signal for when each speaker should rise. 

She has specified the escalation path. If something goes wrong that the waiting staff cannot handle, they tell the wedding coordinator, not the

bride. 

She has reviewed the seating chart three times and found nine conflicts she resolved before the day. 

She has published the seating chart to the venue, the catering manager, the band, and the ushers. 

And on the day of the wedding she sits at the edge of the room with a glass of wine and watches the evening conduct itself. 

The contracts are the reason she gets to sit down. 

The question every engineer avoids 

Everyone who has ever run a complex system — a wedding, a software project, a kitchen during a busy service, a battlefield — knows that the

contracts matter. 

The question they avoid is: how complete do the contracts need to be before you run the system? 

The honest answer is that the contracts need to be complete enough that the most likely failure modes are handled, and specific enough that

the agents can follow them without asking you what you meant. 

Not perfect. Complete enough. 

For the wedding planner this means: every known incompatibility is addressed. Every role has a defined responsibility. Every escalation path

has a named handler. Every sequence has a defined trigger.For contract coding this means the same things. 

The difference between “complete enough” and “done” is the bugs you find in review rather than at runtime. 

I have found eighty-eight of them so far. 

The wedding had not started yet. 

Part 2 will be about what happens when the wedding starts anyway 


At some point you stop reviewing the seating chart and the guests arrive. 

That is first orchestration run. 

I do not know when it is. 

But the contracts are getting there. 

The duck approves 

github.com/murtsu/RostadVM 

Apache 2.0. The seating chart is open. Anyone can check the adjacencies. 

If you find a conflict I missed, open an issue. 

The wedding is not yet scheduled. 

The planning is almost done


*Views, questions, and contributions welcome.*
*This is young. Help make it less young.*
