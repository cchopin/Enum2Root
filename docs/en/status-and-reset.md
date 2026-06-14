# Status & RAZ

## Status tracking

Every methodology step has a **status box** (left of its title). Clicking it cycles through four
states:

| Icon | State | Meaning | Visual |
|:----:|-------|---------|--------|
| `·` | **To do** | default, untouched | neutral |
| `✓` | **Valid** | tried successfully | **green** left border, green title |
| `✗` | **Not valid** | tried without success / not exploitable | **red** left border, red title |
| `∅` | **Not applicable** | irrelevant for this box (not needed / out of scope) | **dimmed, struck-through** |

The click cycle is: **to do → valid → not valid → not applicable → to do**.
The next state's label appears in the box tooltip.

### Why four states?

A simple "done / not done" checkbox can't tell *"I succeeded"* apart from *"I tried, it doesn't
work"* or *"this doesn't apply"*. Those three carry very different meaning in a report and in
progress tracking.

### Effect on progress

- Per-section counters and the global percentage count **valid** steps.
- **Not applicable** steps are **excluded from the denominator**: the percentage reflects progress
  over what's actually relevant for the box.
- A `· N n/a` suffix shows how many steps are marked not applicable.

### In reports and backup

- The **Report .md** prefixes each documented step with its status
  (`[✓ valid]`, `[✗ not valid]`, `[∅ n/a]`).
- Statuses are included in the **JSON export** and scoped **per target** (IP).
- **Compatibility**: an old notebook (single "done" checkbox) is migrated automatically — each
  checked box becomes **valid**.

## RAZ — switch box

The **RAZ** button (toolbar, next to *Theme*) prepares the move to a new target. After
confirmation, it:

- **clears** the target fields: IP, Domain, DC, User, Password, URL, and the search box;
- **resets** the "What I have" context;
- **removes** the nmap filter (imported open ports).

It **keeps**:

- attacker-side fields: **LHOST, Interface, Wordlist** (same across boxes);
- **all notes and progress** — they're stored **per target IP**, so the previous box stays
  reachable: just type its IP again to get its notebook back.

!!! tip
    RAZ deletes nothing saved: it's just a "start clean" on the form. To wipe the whole tool,
    use a fresh browser profile or clear `localStorage`.
