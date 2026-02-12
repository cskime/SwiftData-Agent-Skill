# Contributing

## Scope

This repository maintains one skill: `swiftdata-expert-skill`.

## Contribution Rules

1. Keep the skill focused on SwiftData for iOS 26+.
2. Keep `SKILL.md` procedural and concise.
3. Put deep technical detail into `references/*.md`.
4. Use clear examples and prefer "Wrong vs Correct" patterns.
5. Do not add optional directories (`agents/`, `assets/`) unless explicitly required.

## Agent Efficiency KPIs

- `swiftdata-expert-skill/SKILL.md` stays concise (target: <= 140 lines)
- `SKILL.md` includes a situation-to-reference routing matrix
- Every `references/*.md` file is indexed from `SKILL.md`
- Skill usage should typically require one primary reference (+ one secondary only when needed)

## Regression Checks

- CI (`.github/workflows/validate-skill.yml`) enforces:
  - required files/folders
  - `SKILL.md` reference link validity
  - full reference index coverage
  - duplicate heading detection in `SKILL.md`
  - conciseness and routing-matrix presence

## Pull Request Checklist

- [ ] `swiftdata-expert-skill/SKILL.md` frontmatter is valid (`name`, `description`)
- [ ] New reference files are linked from `SKILL.md`
- [ ] Existing links and paths resolve correctly
- [ ] Examples compile conceptually and match SwiftData terminology
- [ ] No unrelated file changes
