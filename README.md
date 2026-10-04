# Skills

[![skills.sh](https://skills.sh/b/Kzoeps/SKILLS)](https://skills.sh/Kzoeps/SKILLS)

A collection of reusable agent skills. Each skill is self-contained in its own folder under [`skills/`](./skills).

## Included skills

- [`bro`](./skills/bro) — Restates the previous message in plain, concise language.
- [`skill-creator`](./skills/skill-creator) — Creates, tests, evaluates, and improves agent skills.
- [`test-driven-development`](./skills/test-driven-development) — Chooses useful verification before applying focused test-first development.
- [`writing-atproto-lexicons`](./skills/writing-atproto-lexicons) — Designs, reviews, validates, and evolves AT Protocol Lexicon schemas.
- [`delegate-agents`](./skills/delegate-agents) — Coordinates workers and independent reviewers with bounded authority across available agent runners. Herdr is optional; no fixed model or provider is required.

## Install

Install the collection with the Skills CLI:

```sh
npx skills add Kzoeps/SKILLS
```

You can then select the skills you want to install.

## Structure

```text
skills/
├── bro/
│   └── SKILL.md
├── skill-creator/
│   ├── SKILL.md
│   ├── agents/
│   ├── assets/
│   ├── eval-viewer/
│   ├── references/
│   └── scripts/
├── test-driven-development/
│   ├── SKILL.md
│   ├── deciding-what-to-test.md
│   └── writing-good-tests.md
├── delegate-agents/
│   ├── SKILL.md
│   └── references/
└── writing-atproto-lexicons/
    ├── SKILL.md
    ├── references/
    └── evals/
```

Each skill starts with a `SKILL.md` containing its metadata and instructions. Supporting scripts, references, and assets live beside it and are loaded only when needed.

## License

Licensing information is included with individual skills where applicable.
