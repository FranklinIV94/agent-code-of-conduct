# Agent Code of Conduct

**Current Version:** 2.1  
**License:** MIT  
**Last Updated:** August 2026  

A governance framework for AI agent infrastructure — from policy document to enforceable operating system.

## What This Is

Most agent frameworks publish guidelines. This one publishes mechanisms. Every rule maps to a technical control that makes violation architecturally impossible, gated by a separate system, or automatically caught by audit. The goal: governance you can query, not just read.

## Repository Structure

```
agent-code-of-conduct/
├── README.md                  # You are here
├── CHANGELOG.md               # Versioned amendment history
├── v1.0-archive.md            # Original v1.0 document (reference)
├── policy/
│   ├── code-of-conduct.md     # Internal Code v2 (full document)
│   ├── client-deployable.md   # Template for client deployments
│   └── client-summary.md      # 2-page plain language summary
├── schemas/
│   ├── agent-registry.schema.json   # Machine-readable agent registry
│   ├── violation-log.schema.json    # Violation record format
│   └── config-version.schema.json   # Config versioning format
├── templates/
│   ├── scope-definition.md          # What §1.2 "Scope" looks like filled in
│   ├── incident-report.md           # §7.5 timeline template
│   ├── sub-agent-delegation.md      # §1.4 pre-spawn checklist
│   └── code-injection-digest.md     # Condensed Code injected per session
├── controls/
│   └── control-matrix.md            # Every rule → Hard/Gated/Soft + owning layer
├── tests/
│   └── red-team-matrix.md           # Every rule → the test that tries to break it
└── docs/
    └── handoff-v2-audit.md          # Gap analysis & build plan (source of v2 changes)
```

## Quick Start

1. **Read the Code:** `policy/code-of-conduct.md`
2. **Check the controls:** `controls/control-matrix.md` — see how each rule is enforced
3. **Register an agent:** Use `schemas/agent-registry.schema.json` as the format
4. **Onboard:** Follow `templates/scope-definition.md` for the scope definition, `templates/code-injection-digest.md` for the session injection

## Key Concepts

- **Hard controls** make violation architecturally impossible (e.g., no payment credentials on agents → A4 cannot be violated)
- **Gated controls** require pre-action approval from a separate system (e.g., external comms relay holds messages for release)
- **Soft controls** are behavioral rules caught by audit (e.g., scope creep detected by Warden review)
- **Action Authority** classifies every action as Autonomous, Propose, or Escalate per agent
- **Config lifecycle** is the true unit of governance — agents are stateless configurations, not employees

## Version History

- **v2.0** (Jul 2026) — Mechanisms over rules. Injection defense. Config lifecycle enforcement. Productization split.
- **v1.0** (May 2026) — Original framework. See `v1.0-archive.md`.

## License

MIT — Use, adapt, distribute freely. Attribution appreciated but not required.

---

*Developed from production experience governing AI agent infrastructure at Prospyr 305.*