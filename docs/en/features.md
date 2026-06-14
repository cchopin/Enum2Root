# Features

**Dynamic target fields.** The header fields — IP, Domain/FQDN, DC, Username, Password/hash,
**Your IP (LHOST/listener)**, Interface, Wordlist, Web URL — are substituted live into every
command (shown as highlighted tokens).

**Prerequisite system ("What I have").** Each action declares what it needs: nothing, a username,
valid credentials, an NT hash, local admin, a shell, or elevated domain privileges. Requirement
badges turn green (met) or red (missing).

**Unavailable actions — Show all / Reduce / Hide.** Decides what to do with actions you can't run
yet: visible but dimmed, **reduced** to a single line (default), or hidden entirely.

**Focus mode.** One click filters the whole page to your context: runnable commands get a colored
highlight, ones you lack prerequisites for collapse, and fully-locked sections fold away.

**Nmap import.** Load a `.nmap`, `.gnmap` or `.xml` scan and the page keeps only the sections for
open ports, pulls the target IP/host, and the graph filters too.

**Collapsible navigation.** Per-column chevrons, individually collapsible phases, a per-phase
**"+ N hidden"** link to reveal on demand.

**Search.** Live filter across tools, ports and techniques; it auto-expands whatever it matches.

**Per-command notebook.** Click the pencil on any command to paste and save its output. Entries
are timestamped, kept as history, scoped **per target** (the IP).

**Notebook views — Commands / Timeline / Findings.** Mark any entry as a **finding** with ★.

**Status tracking & progress.** Multi-state status per step
(to do / valid / not valid / not applicable). Details: [Status & RAZ](status-and-reset.md).

**RAZ (switch box).** Clears the target, context and nmap filter to start on a new box while
keeping notes and progress. Details: [Status & RAZ](status-and-reset.md).

**Reports & backup.** **Report .md** exports a Markdown report for the current target (with a
`## Findings` section at the top and each documented step's status). **Export/Import JSON** backs
up and restores the whole notebook (notes + findings + progress).

**HackTricks links.** Every relevant section has a **HackTricks ↗** button.

**Graph view.** An Obsidian-style force-directed graph: target → phases → sections, colored by
availability. Wheel = zoom, drag = pan, click = jump to a section.

**Light/Dark theme.** Toggle, remembered between sessions.
