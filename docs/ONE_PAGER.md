# MILMED Connect — One-Pager

## The problem

Dual-use medtech discovery is fragmented across five-plus public sources that nobody has unified:

- **MTEC** — 13,506 member organizations, searchable directory
- **SBIR/STTR** — every DoD award since program inception
- **CDMRP** — 22,456 awards, ~$19.7B, no public database
- **USAspending** — DoD medical contracts (CFDA 12.420)
- **TechLink** — ~1,945 medical technologies from 100+ DoD/VA labs

A company doing AI triage or wearable sensors might appear in three of these and be invisible in the rest. PitchBook covers this space poorly and costs $20k+/year. MILMED Connect's founders answer these questions manually, across sites, every day.

## The solution

**MILMED Connect** — a PitchBook-style intelligence platform for dual-use medical technology, built entirely on public DoD sources. Free, open-source, self-hostable, MCP-native.

- **One search layer** over all sources, with entity resolution merging the same company across MTEC, SBIR, and USAspending into a single profile.
- **Dual-use scoring engine** rating every company on MTEC focus-area fit (combat casualty care, operational medicine, infectious diseases, etc.), civilian traction, FDA status, and team readiness — with cited evidence and contradiction flags.
- **AI-queryable** via MCP server: ask "which hemorrhage-control companies have SBIR awards but no MTEC membership?" and get an answer in seconds.

## Why now

- All source data is public and scrapable today — no partnerships or licenses required.
- TechLink is shipping an AI search layer late 2026; being first with dual-use scoring on top of raw data is a window.
- MILMED Connect's network of DoD investigators and PIs turns a list into funded projects — the platform amplifies that, it doesn't replace it.

## What it is not

Not a replacement for MILMED Connect's relationships or judgment. The moat is scoring + relationships. The platform gets you the list; the phone call still comes from the person.

## The ask

A working demo answering three real queries MILMED Connect runs manually. Twelve-week build, local-first (no Docker), MIT-licensed, repo at github.com/seantei/milmed-connect.

---

*Contact: [your name] · [email] · [phone]*