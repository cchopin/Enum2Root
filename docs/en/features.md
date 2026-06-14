# Features

Enum2Root is a single HTML file, yet it covers a whole engagement: from entering the target to
tracking progress, with a per-command notebook and report export. Overview below.

## Target & context

<div class="grid cards" markdown>

-   :material-form-textbox: __Dynamic target fields__

    IP, Domain/FQDN, DC, Username, Password/hash, **Your IP (LHOST)**, Interface, Wordlist, URL -
    substituted live into every command (highlighted tokens). Change a field, everything updates.

-   :material-key-chain: __"What I have" prerequisites__

    Each action declares what it needs (nothing, a username, credentials, a hash, local admin, a
    shell, domain privileges). Tick what you hold: badges turn green (met) or red (missing).

-   :material-target: __Focus mode__

    One click filters the whole page to your context: runnable commands are highlighted, ones
    you lack prerequisites for collapse, locked sections fold away.

-   :material-eye-off: __Unavailable actions__

    Choose how to display actions you can't run: visible but dimmed, **reduced** to one line
    (default), or hidden.

</div>

## Filtering & navigation

<div class="grid cards" markdown>

-   :material-file-import: __Nmap import__

    Load a `.nmap`, `.gnmap` or `.xml` scan: the page keeps only open-port sections, pulls the
    target IP/host, and the graph filters too.

-   :material-magnify: __Live search__

    Instant filter across tools, ports and techniques; auto-expands matches.

-   :material-arrow-collapse-vertical: __Collapsible navigation__

    Per-column chevrons, collapsible phases, a per-phase **"+ N hidden"** link to reveal on
    demand.

-   :material-graph: __Graph view__

    Obsidian-style force-directed graph: target → phases → sections, colored by availability.
    Wheel = zoom, drag = pan, click = jump to a section.

</div>

## Tracking & reporting

<div class="grid cards" markdown>

-   :material-checkbox-multiple-marked: __Status tracking__

    Four-state status per step (to do / ✓ valid / ✗ not valid / ∅ not applicable), with progress
    that excludes "not applicable".

    [:octicons-arrow-right-24: Details](status-and-reset.md)

-   :material-broom: __RAZ button__

    Start clean on a new box (clears target, context and nmap filter) while keeping notes and
    progress.

    [:octicons-arrow-right-24: Details](status-and-reset.md)

-   :material-notebook-edit: __Per-command notebook__

    Paste and save each command's output. Timestamped entries, kept as history, scoped **per
    target** (IP). Flag an entry as a **finding** with ★.

-   :material-file-export: __Reports & backup__

    **Report .md** (Findings section + each step's status); **Export/Import JSON** of the whole
    notebook (notes + findings + progress).

</div>

## Comfort

**Reference links.** Every relevant section has a **HackTricks ↗** button; web-attack sections
also get a **Payloads ↗** button to PayloadsAllTheThings.

**Light/Dark theme.** Toggle remembered between sessions; command blocks stay dark for terminal
readability.

**100% local.** Notes, progress and settings live only in your browser's `localStorage` -
nothing is sent anywhere.
