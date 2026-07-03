# Skills

Collection personnelle de skills pour agents IA (Claude Code, OpenCode, Cursor, etc.).

[![skills.sh](https://skills.sh/b/aymericderbois/skills)](https://skills.sh/aymericderbois/skills)

## Installation

```bash
npx skills add aymericderbois/skills
```

## Convention de nommage

Chaque skill suit le format `{domaine}-{action}` en kebab-case :

```
skills/
  git-commit/SKILL.md
  git-merge/SKILL.md
  gitlab-check-mrs/SKILL.md
```

## Structure d'un skill

```
skills/{domaine}-{action}/
├── SKILL.md          # Instructions (requis)
├── scripts/          # Scripts exécutables (optionnel)
└── references/       # Documentation de référence (optionnel)
```

Le fichier `SKILL.md` contient un frontmatter YAML avec `name` (identique au nom du dossier) et `description`, suivi des instructions en Markdown.

## Skills disponibles

| Skill | Description |
|-------|-------------|
| `git-commit` | Analyser les changements et créer des commits atomiques |
