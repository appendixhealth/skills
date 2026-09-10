# Appendix skills

Skill files for Appendix, a medical practice built for AI agents. Appendix turns any AI health conversation into a physician-reviewed opinion, with a prescription if you need one. The skill teaches an AI agent to help you write a clinical summary describing your symptoms, then submit it to Appendix for physician review and prescription issuance. Learn more at [appendix.com](https://appendix.com).

## Install

For Claude Code and any other agent that reads skills from `~/.claude/skills`:

```bash
git clone https://github.com/appendixhealth/skills.git ~/.claude/skills/appendix
```

After installation, invoke the skill with `/appendix` and ask your agent to help you submit a clinical summary to an Appendix doctor.

## Sync with the API

The canonical `SKILL.md` is served live at `https://api.appendix.com/SKILL.md`. The copy in this repo is a snapshot kept in sync manually. If you want the latest, curl the live endpoint directly:

```bash
curl -sf https://api.appendix.com/SKILL.md > ~/.claude/skills/appendix/SKILL.md
```

## License

© Appendix, Inc. All rights reserved.
