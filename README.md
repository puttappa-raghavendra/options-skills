# Options Skills

Claude skills for rules-based options trading. Each skill lives in its own folder under
`skills/` and teaches Claude one complete strategy: when to enter, how to size, how to
manage the position, and when to exit.

## Skills

| Skill | Ticker | Strategy | Broker |
|---|---|---|---|
| [qqq-iron-condor-ladder](skills/qqq-iron-condor-ladder/) | QQQ | Weekly ~30 DTE iron condor ladder, trend-tilted, with mechanical exits | Interactive Brokers (IBKR connector) |

## Layout

```
skills/
  <skill-name>/
    SKILL.md      # instructions Claude follows (name + description in the frontmatter)
    README.md     # for humans: the strategy, the ticker, how to use it
    *.json        # config with default values, if the skill has one
```

## Adding a skill

1. Create `skills/<skill-name>/` with a `SKILL.md` whose frontmatter has `name` and `description`.
2. Add a `README.md` that explains the strategy, which ticker it trades, and how to use it.
3. Add a row to the table above.

## Installing a skill

Copy the skill folder into your Claude skills directory (for Claude Code,
`~/.claude/skills/<skill-name>/`), or upload it as a skill in Claude settings.

## Disclaimer

These skills are educational tools, not financial advice. Options trading carries
substantial risk of loss. Every order the skills produce is a ticket you review and
approve yourself; nothing is placed without your sign-off.
