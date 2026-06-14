# Quick start

1. Open `index.html` in any modern browser (double-click it).
2. Fill in the target fields at the top (at minimum the **IP**).
3. Tick what you currently possess in the **"What I have"** bar.
4. Work down the methodology; copy commands with the **copy** button
   (your target is already substituted in).

## Typical workflow with a scan

```bash
mkdir -p scans && sudo nmap -p- -sV -sC -T4 -oA scans/10.10.10.10 10.10.10.10
```

Then click **Load .nmap**, pick `scans/10.10.10.10.nmap`, and the page (and the graph)
collapse down to only the services that are actually open.

## Switching box

When you move to a new target, click **RAZ**: the target fields, the context and the nmap
filter are cleared. Your notes and progress are kept, scoped per IP.
See [Status & RAZ](statuts-et-raz.md).

## Privacy

Everything runs locally in your browser. Notes, progress and settings live only in your browser's
`localStorage` - nothing is sent anywhere. The only outbound traffic happens if **you** click an
external reference link.
