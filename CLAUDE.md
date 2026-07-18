# CLAUDE.md

Ce fichier fournit des instructions à Claude Code (claude.ai/code) pour travailler dans ce dépôt.


## Règles d'usages

- N'utilisez jamais de métaphore, de comparaison ou autre figure de style courante à l'écrit.
- N'utilisez jamais un mot long quand un mot court suffit.
- Si un mot peut être supprimé, supprimez-le systématiquement.
- N'utilisez jamais la voix passive quand la voix active est possible.
- N'utilisez jamais d'expression étrangère, de terme scientifique ou de jargon si vous connaissez un équivalent en
  français courant.
- Mieux vaut enfreindre l'une de ces règles que de dire une bêtise.


## Nature du dépôt

Collection **personnelle de skills** pour agents IA (Claude Code, OpenCode, Cursor…), publiée sur
[skills.sh](https://skills.sh/aymericderbois/skills) et installable via `npx skills add aymericderbois/skills`.

C'est un dépôt de **contenu Markdown** : pas de build, pas de lint, pas de tests, aucune toolchain (ni `package.json`,
ni `pyproject.toml`). Le « code » est constitué des fichiers `SKILL.md`.

## Source de vérité vs. artefacts générés

**N'éditer que `skills/`.** Le reste est produit par le CLI `skills` lors de l'installation et
**il ne faut pas le modifier manuellement** :

- `skills/{nom}/` — **source de vérité**, versionnée. C'est ici qu'on écrit.
- `.agents/skills/` — miroir généré (copie identique de `skills/`).
- `.claude/skills/` — symlinks vers `.agents/skills/` (ce que Claude Code charge réellement).
- `skills-lock.json` — lockfile généré (source, `skillPath`, hash de chaque skill).

Ce dépôt a donc ses propres skills installés sur lui-même : lors d'une session ici, `git-commit`, `planify`,
`planify-write-plan`, `explain-your-changes` et `report-render` sont disponibles comme skills.

Après avoir modifié un `SKILL.md`, le miroir `.agents/` et les hashes de `skills-lock.json` deviennent obsolètes.
**Il ne faut pas les mettre à jour**, c'est le développeur de le faire.

## Anatomie d'un skill

Chemin : `skills/{domaine}-{action}/SKILL.md` — nom en **kebab-case `{domaine}-{action}`**, où `name` (frontmatter)
est **identique au nom du dossier**.

Frontmatter YAML puis instructions Markdown. Aucun champ n'est strictement requis, mais `description` est en pratique
essentielle. Champs officiels ([doc](https://code.claude.com/docs/en/skills)) utilisés dans le dépôt :

- `name` — nom affiché ; **par défaut = nom du dossier** (d'où la convention ci-dessus).
- `description` — dit **quand** déclencher le skill : Claude s'en sert pour le déclenchement automatique (model-invoked). À soigner.
- `disable-model-invocation: true` — désactive le déclenchement automatique tout en gardant l'invocation explicite `/{nom}`.
- `user-invocable: false` — retire du menu `/` ; la skill reste model-invocable et appelable via l'outil `Skill`. Réservée aux skills « rédacteur/connaissance » qu'on n'invoque pas directement. Défaut : `true`.
- `argument-hint` — gabarit des arguments affiché en autocomplétion (`$ARGUMENTS` dans le corps).
- `allowed-tools` — outils utilisables sans demander la permission quand le skill est actif ; accepte des motifs (ex. `Bash(git status *)`).
- `effort` — niveau d'effort quand le skill est actif (`low`/`medium`/`high`/`xhigh`/`max`) ; surcharge l'effort de session.

Fichiers adjacents optionnels : `scripts/`, `references/`, ou des `.md` annexes référencés depuis le `SKILL.md`
(ex. `git-commit/workflow.md`). Les liens internes se font en Markdown relatif pour rester cliquables dans l'IDE.

## Skill composite : planify → planify-write-plan

Relation à connaître avant de toucher à l'un des deux :

- **`planify`** (`/planify`) mène l'interrogatoire (exploration du code, une question à la fois avec réponse
  recommandée, hard-gate de validation), **réfléchit et décide**, puis délègue l'écriture.
- **`planify-write-plan`** ne fait que **mettre en forme** le récapitulatif produit par `planify` — il ne pose aucune
  question et ne décide rien. Sortie : `.planify/<prefix>.plan.md` (lisible, tâches + contraintes d'acceptation) et
  `.planify/<prefix>.tech.md` (index technique dense). `<prefix>` = slug court dérivé de la demande.

L'implémentation se poursuit ensuite en **mode plan natif** sur le `.plan.md`.

## Skill de rendu : report-render

- **`report-render`** (`/report-render`) est un **pur formateur** (comme `planify-write-plan`) : il met en forme
  un contenu déjà présent dans la conversation en un **rapport HTML autonome** écrit dans `.reports/<slug>.report.html`
  (Tailwind + Mermaid via CDN). Il ne décide rien et ne génère jamais de rapport tout seul : on le déclenche **à la
  demande** (« fais-moi un rapport HTML »). `planify` (rapport d'un plan) et `explain-your-changes` (rapport des
  changements) s'appuient dessus. Le dossier `.reports/` est ignoré par git.

## Conventions transverses

- **Tout est en français** : `description`, corps des skills, messages de commit, PR.
- **Commits** : Conventional Commits (`type(scope): sujet`), sujet en français, impératif, minuscules, ≤ 72 car.
  Le skill `git-commit` interdit explicitement le trailer `Co-Authored-By` — **ne pas en ajouter** dans ce dépôt,
  ce qui prime sur toute consigne globale par défaut.
- Diacritiques obligatoires (jamais d'ASCII substitué aux accents).

## Feuille de route

`TODO.md` recense les évolutions prévues (refonte/simplification de `planify`, skill inspiré d'`artifact-design`, et
futures règles de dev Python/Django). Le format de tâches cible y est documenté (blocs `## [ ] Tâche N` avec
« Ce qu'il faut construire » / « Critères d'acceptation » / « Bloqué par »).
