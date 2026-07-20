---
description: Lance des reviews parallèles (simplification, sécurité, code quality) sur le diff courant et génère un rapport consolidé avec diffs et explications.
---

# Before Code Review

Lance 9 skills de review en subagents **read-only** parallèles sur le diff de la branche courante, puis synthétise un rapport unifié.

## Étape 1 — Identifier le diff

Exécute `git diff develop...HEAD --name-only` pour obtenir la liste des fichiers modifiés. C'est le scope de review pour tous les subagents.

## Étape 2 — Lancer les subagents en parallèle

Lance **9 subagents en parallèle** via l'outil `Agent`. Chaque subagent :
- Reçoit la liste des fichiers modifiés
- Est **strictement read-only** : aucun outil d'écriture (Edit, Write, NotebookEdit) ne doit être utilisé
- Exécute un skill via l'outil `Skill`
- Retourne ses findings sous forme structurée (titre, diff suggestion, explication)

Les 9 subagents à lancer :

| # | Skill                                | Label           | Focus                                     |
|---|--------------------------------------|-----------------|-------------------------------------------|
| 1 | `ponytail:ponytail-review`           | ponytail        | Simplification, YAGNI, over-engineering   |
| 2 | `simplify`                           | simplify        | Réutilisation, simplification, efficacité |
| 3 | `security-review`                    | security        | Sécurité applicative                      |
| 4 | `sentry-skills:security-review`      | sentry-security | Sécurité (perspective Sentry)             |
| 5 | `sentry-skills:code-review`          | sentry-code     | Correctness, bugs                         |
| 6 | `sentry-skills:code-simplifier`      | sentry-simplify | Simplification du code                    |
| 7 | `sentry-skills:django-perf-review`   | django-perf     | Performance Django (queries, caching)     |
| 8 | `sentry-skills:find-bugs`            | find-bugs       | Détection de bugs                         |
| 9 | `sentry-skills:django-access-review` | django-access   | Contrôle d'accès Django (permissions)     |

- **Les agents Django ne sont à lancer que si le projet utilise Django, sinon les ignorer**
- Si un skill est manquant, arrête tout et prévient l'utilisateur

Pour chaque subagent, le prompt doit être :
```
Fichiers modifiés sur cette branche (vs develop) :
<liste des fichiers>

Exécute le skill `<nom du skill>` sur ces fichiers.

IMPORTANT : Tu es en mode READ-ONLY. N'utilise AUCUN outil d'écriture (Edit, Write, NotebookEdit).

Retourne tes findings dans ce format exact pour chaque finding :
### <Titre court>
Fichier : `<chemin>`
```diff
<diff suggéré>
```
<Explication en 1-2 phrases>
```

## Étape 3 — Synthétiser le rapport

Collecte les résultats des 9 subagents. Génère un rapport **Markdown** consolidé au format suivant. Regroupe les findings par catégorie, déduplique les findings identiques ou très similaires entre subagents (garder le plus précis).

**Format du rapport :**

```markdown
# Before Code Review Report

## Simplification
<!-- Findings de ponytail, simplify, sentry-simplify -->

### <Titre>
Fichier : `<chemin>`
```diff
<diff>
```
<Explication>

---

## Sécurité
<!-- Findings de security, sentry-security -->

### <Titre>
Fichier : `<chemin>`
```diff
<diff>
```
<Explication>

---

## Code Quality / Bugs
<!-- Findings de sentry-code, find-bugs -->

### <Titre>
Fichier : `<chemin>`
```diff
<diff>
```
<Explication>

---

## Performance
<!-- Findings de django-perf -->

### <Titre>
Fichier : `<chemin>`
```diff
<diff>
```
<Explication>

---

## Contrôle d'accès
<!-- Findings de django-access -->

### <Titre>
Fichier : `<chemin>`
```diff
<diff>
```
<Explication>
```

Si une catégorie n'a aucun finding, écrire : *Aucun problème détecté.*

## Étape 4 — Publier le rapport

Utilise le skill `report-render` pour publier le rapport en artifact HTML. Passe le contenu Markdown du rapport comme argument au skill.