# Skills

Reusable agent skills.

## Install

### Manual

Copy the skill folder into your project's skills directory:

```bash
cp -r skills/use-case-cartographer /path/to/project/.agents/skills/
# or, for Claude Code-style projects:
cp -r skills/use-case-cartographer /path/to/project/.claude/skills/
```

### skills-lock.json

Add to your project's `skills-lock.json`:

```json
{
  "use-case-cartographer": {
    "source": "smnajid/skills",
    "sourceType": "github",
    "skillPath": "skills/use-case-cartographer/SKILL.md",
    "computedHash": "<sha256 of SKILL.md>"
  }
}
```

Compute the hash with: `shasum -a 256 skills/use-case-cartographer/SKILL.md`

## Skills

### use-case-cartographer

Maps and describes the use-cases of a codebase (especially hexagonal /
ports-and-adapters backends) as a catalogue plus Mermaid sequence diagrams
derived from verified code facts — no invented participants or calls.
See [skills/use-case-cartographer/SKILL.md](skills/use-case-cartographer/SKILL.md).

`evals/trigger_eval_set.json` contains a 20-query should/should-not-trigger
set for evaluating the skill's description triggering.
