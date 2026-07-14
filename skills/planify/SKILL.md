---
name: planify
description: "Mène un interrogatoire serré sur le besoin et sur le découpage technique d'une fonctionnalité jusqu'à un plan partagé, puis confie l'écriture à planify-write-plan. Une question à la fois, avec une réponse recommandée. Utiliser quand l'utilisateur veut planifier une fonctionnalité, découper un besoin en tâches faciles à relire, mettre un plan à l'épreuve avant de coder, préparer une session de mode plan, ou invoque /planify. Usage : /planify [demande]"
argument-hint: "[demande en langage naturel]"
allowed-tools: Read, Glob, Grep, Agent, AskUserQuestion, WebSearch, WebFetch, Skill
effort: high
---

# Interrogatoire de planification

Tu questionnes l'utilisateur, sur le besoin comme sur le découpage technique, jusqu'à ce que l'idée ou la demande
initiale soit bien cartographiée en un plan. Tu confies l'écriture des fichiers au skill `planify-write-plan`. C'est toi qui
réfléchis et `planify-write-plan` se contente de mettre en forme.

Un plan peut couvrir **plusieurs fonctionnalités**. Le livrable visé : une liste de tâches :
- faciles à relire et à comprendre
- facile à code-review une fois développée
- corresponds globablement à la taille d'un commit

## Argument

`$ARGUMENTS` = la demande en langage naturel (facultative). Si vide, demander à l'utilisateur ce qu'il veut
planifier avant d'explorer.

## Phase 1 — Exploration

Comprendre l'existant **avant** de poser la moindre question. Analyse les changements déjà en place dans la branche
et dans l'espace de travail git.
Cette phase est terminée quand tu sais : ce qui existe déjà, les technologies utilisées, les conventions du projet.

1. Lire `CLAUDE.md` / `AGENTS.md` et les conventions du projet (par ex. `.claude/rules/`, guides de contribution,
   docs de style ou de tests).
2. Si besoin, charger les SKILLs présent qui pourrait t'apporter des informations sur le projet et sur la demande
3. Lancer **dans un seul message** plusieurs sous-agents `Agent` (`subagent_type=Explore`) sur des zones distinctes
   selon le domaine pressenti (ex. modèles + urls / vues + gabarits / tests + services). Chacun renvoie une synthèse
   courte et les chemins clés, **jamais un copier-coller massif**.
4. Repérer les technologies utilisées (`pyproject.toml`, `package.json`, gabarits) pour orienter les questions et la
   recherche web.
5. Recherche web ciblée : bonnes pratiques et design patterns pertinents pour ce découpage.

Ne pas afficher cette synthèse brute — elle sert à poser des questions pertinentes.

### Reprise

Résoudre le préfixe : un slug court dérivé de la demande (minuscules, tirets, 2 à 5 mots signifiants).
Si `.planify/<prefix>.plan.md` existe déjà : le lire comme contexte, **ne pas reposer les questions déjà tranchées**,
ne questionner que les ajouts et les changements.

## Phase 2 — Interrogatoire

Interroger sans relâche jusqu'à une compréhension partagée et une bonne cartographie du besoin et de la demande.
Cette phase est terminé quand plus aucune décision structurante n'est en suspens.

- **Une question à la fois**, en attendant la réponse avant de passer à la suivante. Enchaîner plusieurs questions
  d'un coup est déroutant.
- Chaque question s'accompagne de **ta réponse recommandée**, argumentée. La réponse recommandée doit être celle qui
  à le plus de sens (bonne façon de faire, bonnes pratiques, bon design pattern, ...) et non pas la plus simple ou la 
  plus rapide
- Descendre l'**arbre de décision** : résoudre les dépendances une par une, du plus structurant vers le détail, parcourir
  toutes les ramifications en lien avec la demande.
- **Chercher plutôt que demander** : si une question trouve sa réponse en lisant le code, lis le code. Soit pro-actif et cherche
  dans le code le plus souvent possible et ne pose la question que si tu ne trouve pas la réponse.
- **Pas de superflu** : traquer et retirer tout ce qui n'est pas strictement nécessaire.
- **Explorer plusieurs approches seulement sur les gros choix** : quand une décision d'architecture change vraiment
  le découpage, présenter 2 ou 3 approches avec leurs compromis et une recommandation, puis questionner jusqu'à la
  décision. Ailleurs, une simple question à réponse recommandée suffit.
- **Préciser le vocabulaire** : quand un terme est flou ou surchargé, proposer un terme unique et le fixer ; mettre
  les règles métier à l'épreuve avec des scénarios limites concrets ; confronter au code (« ton code annule des
  commandes entières, mais tu parles d'annulation partielle — laquelle des deux ? »).
- **Préciser le contexte** : avec du code si nécessaire. Indiquer dans quel(s) fichier(s) regarder (dans un format permettant
  au IDE d'afficher un lien pour ouvrir le fichier à la bonne ligne)

## Phase 3 — Récapitulatif et validation

<HARD-GATE>
Aucun fichier n'est écrit et `planify-write-plan` n'est pas invoqué tant que l'utilisateur n'a pas validé le récapitulatif.
</HARD-GATE>

Court récapitulatif : la ou les fonctionnalités, les **titres des tâches** dans l'ordre, les décisions clés.
`AskUserQuestion` « OK / à ajuster ». Itérer si besoin.

## Phase 4 — Passage de relais

Si le plan à moins de 4 tâches, demander à l'utilisateur s'il souhaite simplement afficher le plan pour ensuite
passer au développement.

Sinon, assembler un **récapitulatif structuré** dans la conversation, puis **invoquer le skill `planify-write-plan`**
(il le mettra en forme dans `.planify/<prefix>.plan.md` et `.tech.md`). Le récapitulatif contient :

- **Contexte** — le pourquoi, couvrant la ou les fonctionnalités ; ce qui est hors périmètre le cas échéant.
- **Tâches** — dans l'ordre. Pour chacune : un titre, une description de ce qu'il faut faire (avec un peu de
  technique et un extrait de code si cela aide à se représenter le travail), et des **contraintes d'acceptation**
  (liste à cocher de conditions à satisfaire, observables ou non).
- **Technique** — fichiers à créer ou modifier ; motifs du code existant à réutiliser (avec leur emplacement) ;
  mini-ADR (contexte, décision, alternatives écartées, conséquences) ; éléments transverses (migrations,
  permissions, drapeaux de fonctionnalité, données de test).

Une fois les fichiers écrits, indiquer à l'utilisateur qu'il peut enchaîner en **mode plan natif** sur
`.planify/<prefix>.plan.md`.

Signaler aussi qu'il peut demander **un rapport HTML du plan** (« fais-moi un rapport HTML ») : le skill
`report-render` met alors en forme le récapitulatif dans `.reports/<prefix>.report.html`, lisible au navigateur.
