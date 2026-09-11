---
name: dev-journal
description: Tient le journal de bord d'un ticket dans .task/<ticket>.journal.md, au fil de l'eau. À utiliser pendant tout travail sur un ticket dès qu'une décision technique se prend, qu'une décision en remplace une autre, qu'un point à savoir apparaît (migration longue, commande à exécuter, comportement à connaître), et à chaque tâche terminée pour tenir le résumé à jour. Aussi invocable via /dev-journal pour créer le journal ou y consigner une entrée.
argument-hint: "[optionnel : décision ou point à consigner]"
allowed-tools: Read, Write, Edit, Glob, Bash(date *), Bash(git branch *), Bash(mkdir *)
---

## Rôle

Tenir, par ticket, un **journal de bord** que le développeur relit d'un coup avant la MR ou la QA, et qu'un agent
qui reprend le ticket lit pour se remettre dans le contexte. Trois contenus : ce qui a été fait, les décisions
prises, les points à savoir.

Le journal capte ce qui s'évapore : le **pourquoi** des choix, les revirements, et les actions à ne pas oublier
(migration à lancer, commande à rejouer). Le git log porte le quoi, le journal porte le reste.

**En reprise d'un ticket, lire le journal en premier** — c'est sa raison d'être.

Ce skill ne se relance pas tout seul au fil du travail. Pour l'écriture continue, le projet ajoute une règle dans
son `CLAUDE.md` (ex. « après chaque décision technique ou tâche terminée, invoque le skill `dev-journal` »).

## Contexte

- Branche courante : !`git branch --show-current`
- Date et heure : !`date '+%F %H:%M'`
- Argument reçu : $ARGUMENTS

## Le fichier

- Emplacement : `.task/<ticket>.journal.md`. `.task/` est un dossier à la racine du projet, réservé aux fichiers
  de travail par ticket. Il peut aussi contenir une trame du ticket écrite par le développeur (`.task/<ticket>.md`) ;
  le journal se met à côté. Créer `.task/` s'il manque.
- `.task/` n'est pas versionné par défaut : l'ajouter au `.gitignore` du projet s'il n'y figure pas, sauf si le
  développeur veut partager le journal.
- `<ticket>` : premier identifiant au motif `LETTRES-CHIFFRES` (ex. `ABC-123`) dans le nom de la branche courante.
  Sans identifiant, prendre le nom de la branche avec les `/` remplacés par `-`.
- Créé à la première entrée si absent, à partir du gabarit ci-dessous — sections vides, sans puce d'exemple.
- Un seul rédacteur côté agents : **l'agent principal**. Les sous-agents n'écrivent pas dedans ; l'agent principal
  consigne leurs conclusions utiles au moment où il reçoit leur rapport. Le développeur, lui, écrit ce qu'il veut.

### Gabarit

```markdown
# <ticket> — Journal de bord

## Ce qui a été fait

## Décisions

## À savoir
```

## Quand écrire

**Au fil de l'eau.** Chaque entrée s'écrit au moment où l'information apparaît, jamais en une passe de rattrapage
en fin de session.

| Moment | Section | Action |
|---|---|---|
| Une décision technique se prend | Décisions | Ajouter une puce horodatée **en fin de liste** (ordre chronologique) |
| Une décision en remplace une autre | Décisions | Ajouter la nouvelle puce en fin de liste, **barrer** l'ancienne (`~~…~~`) avec le renvoi « remplacée : voir plus bas » |
| Un point à savoir apparaît | À savoir | Ajouter la puce immédiatement |
| Une tâche se termine | Ce qui a été fait | Mettre à jour le résumé, tâche par tâche |
| L'agent exécute une action cochable (commande lancée, migration passée) | À savoir | Cocher la case ; sinon elle reste au développeur |

Une **tâche** est une unité de travail livrée : une tâche du plan `.planify/` si un plan existe, sinon un lot de
changements de la taille d'un commit.

Si `$ARGUMENTS` n'est pas vide : consigner son contenu dans la section qui correspond (décision ou à savoir).
S'il est vide : mettre le journal à niveau depuis le contexte de la conversation.

L'horodatage reprend la date et l'heure du bloc Contexte ; relancer `date '+%F %H:%M'` si plusieurs entrées
s'écrivent à des moments éloignés dans la même session.

## Format des entrées

### Décisions

Une puce par décision : `- <date heure> — <choix> — <pourquoi en quelques mots>`. Quelques mots, pas un paragraphe.

```markdown
- 2026-09-11 14:32 — Champ city sur Provider plutôt qu'une table Address — une seule adresse par tiers
- ~~2026-09-11 15:03 — Parsing des adresses en Python pur~~ (remplacée : voir plus bas)
- 2026-09-11 15:47 — Parsing des adresses via libpostal — les regex maison cassaient sur les adresses étrangères
```

### À savoir

- **Actionnable** → case à cocher, commande exacte en inline code, prête à copier :
  `- [ ] Lancer la migration 0042 — ~5 min sur la prod, table providers`
- **Informatif** → puce simple :
  `- L'export garde l'ancien format tant que le feature flag est off`

### Ce qui a été fait

Quelques phrases, comme un développeur l'expliquerait à un collègue — le **quoi d'ensemble et le comment**, là où
le git log ne donne que des messages de commit isolés. Un diagramme Mermaid **seulement si un schéma aide vraiment**
(flux entre composants, enchaînement d'états) — pas de diagramme qui paraphrase deux phrases.

## Garde-fous

- Écrire court : mots courts, voix active, pas de figure de style. Le lecteur est un développeur pressé.
- Une décision sans pourquoi n'apporte rien — le pourquoi est la charge utile de l'entrée.
- Ne pas réécrire l'historique : une décision annulée se barre, elle ne se supprime pas.
- Le journal ne remplace ni la trame du ticket ni un fichier de plan — il les complète.
