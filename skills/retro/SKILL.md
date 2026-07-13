---
name: retro
description: À utiliser quand l'utilisateur demande une rétrospective de session, une « rétro », un « bilan de session », veut passer en revue les points de friction de la session en cours, ou invoque /retro. Analyse la conversation en cours pour repérer les frictions (corrections, redirections, hypothèses fausses, reprises manuelles), remonte chacune à sa cause racine, et propose des éditions de capitalisation concrètes routées vers .claude/rules/, .claude/skills/ ou CLAUDE.md.
disable-model-invocation: true
argument-hint: "[optionnel : aspect de la session à cibler]"
allowed-tools: Read, Write, Edit, Glob, Grep
---

# retro

## Overview

À la fin d'une session de travail, repérer les **points de friction** (ce qui a coincé entre l'utilisateur et l'agent),
remonter chacun à sa **cause racine**, et proposer des **éditions concrètes** pour que les sessions suivantes se passent
mieux — capitalisées dans le repo, donc versionnées et synchronisées sur toutes les machines.

Principe central : **une friction récurrente = une règle manquante**. La rétro convertit le feedback implicite de la
session en texte persisté et actionnable, exactement comme un post-mortem.

Cette skill **analyse la conversation en cours** (celle qu'elle a déjà dans son contexte). Elle ne parse pas de fichier
de transcript externe.

## When to Use

- L'utilisateur dit : « rétro », « bilan de session », « qu'est-ce qui a coincé ? », « retro », ou invoque `/retro`
- En fin de session, ou après une tâche qui a demandé beaucoup d'allers-retours
- Quand l'utilisateur veut transformer une frustration ponctuelle en amélioration durable

**Ne pas utiliser quand :**

- La session a été triviale (une seule question, aucune correction) → répondre qu'il n'y a rien de notable à capitaliser
- Le contexte a été lourdement compacté → le signaler : la rétro ne porte que sur ce qui reste en contexte (voir
  *Limites*)

## Limites

- La rétro ne voit que **la conversation présente en contexte**. Si une compaction a eu lieu, les frictions du début
  peuvent être perdues — le dire explicitement et ne pas inventer.
- Elle ne mesure pas (pas de métriques OpenTelemetry ici) : c'est une analyse **qualitative** des signaux de friction.

## Signaux de friction à repérer

Balayer la conversation et relever, avec pour chacun le **tour** concerné :

| Signal                     | À quoi le reconnaître                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------- |
| **Correction utilisateur** | « non », « plutôt », « en fait », « pas comme ça », reformulation d'une consigne      |
| **Redirection**            | l'utilisateur réoriente après que l'agent est parti dans une mauvaise direction       |
| **Hypothèse fausse**       | l'agent a supposé qqch d'inexact (chemin, convention, intention) et a dû se reprendre |
| **Clarification répétée**  | plusieurs allers-retours pour cerner un besoin qui aurait pu être posé d'emblée       |
| **Approche abandonnée**    | du travail jeté / refait après coup                                                   |
| **Reprise manuelle**       | l'utilisateur a dû corriger lui-même ce que l'agent a produit                         |
| **Permission refusée**     | une action bloquée faute d'autorisation prévue                                        |
| **Friction d'outillage**   | commande qui échoue par méconnaissance du repo (toolchain, chemins, conventions)      |

## Workflow

### 1. Analyser

Relire la conversation et lister les frictions repérées. Pour chacune : ce qui s'est passé, à quel moment, et l'impact
(temps perdu, retouche, frustration).

### 2. Remonter à la cause racine

Pour chaque friction, répondre à : *qu'est-ce qui, présent dès le départ, l'aurait évitée ?* Les causes typiques :

- une **règle/convention manquante** que l'agent aurait dû connaître → cible `rules/` ou `CLAUDE.md`
- un **savoir-faire procédural** absent (suite d'étapes réutilisable) → cible `skills/`
- une **instruction ambiguë** côté utilisateur → pas une édition de config, mais une note sur comment mieux formuler
- un **manque de contexte ponctuel** non récurrent → ne rien capitaliser (l'over-capitalisation pollue)

Ne pas transformer chaque friction en règle. **Une friction ponctuelle ne mérite pas une règle.** Privilégier ce qui a
des chances de **se reproduire**.

### 3. Router chaque leçon vers la bonne destination

| Nature de la leçon                                   | Destination                         | Pourquoi                                    |
| ---------------------------------------------------- | ----------------------------------- | ------------------------------------------- |
| Savoir-faire réutilisable (suite d'étapes, workflow) | **`.claude/skills/<nom>/SKILL.md`** | invocable, déclenché par sa `description`   |
| Règle / fait à retenir, ciblé sur un sujet           | **`.claude/rules/<sujet>.md`**      | auto-chargé chaque session, versionné       |
| Convention cœur, transversale et toujours active     | **`CLAUDE.md`**                     | toujours en contexte, vue de toute l'équipe |
| Fait propre à **une seule** machine                  | mémoire auto (`~/.claude/…`)        | rare ; à éviter car non synchronisé         |

Toutes les destinations « repo » (rules/skills/CLAUDE.md) sont versionnées Git → synchronisées entre machines et
partagées avec l'équipe. C'est le choix par défaut.

### 4. Proposer — et STOP

Présenter un tableau de propositions, **sans rien écrire encore**, puis attendre la validation explicite de
l'utilisateur. Format :

```
Frictions de la session :

1. <friction> (tour ~N)
   Cause racine : <cause>
   → Proposition : <action> dans <fichier cible>
     Texte exact à ajouter/modifier :
     ----
     <le contenu précis>
     ----

2. …

Aucune capitalisation pour : <frictions jugées ponctuelles, listées avec une phrase de justification>
```

Règles pour les propositions :

- Donner le **texte exact** à écrire, jamais un vague « améliorer X ».
- Indiquer le **fichier cible précis**.
- Si une règle existante est concernée, proposer une **modification** (fusion/mise à jour) plutôt qu'un doublon —
  appliquer l'esprit ADD / MERGE / DELETE pour éviter que `rules/` et `CLAUDE.md` enflent.
- Lister explicitement ce qu'on **ne** capitalise **pas**, avec la raison (transparence anti-bruit).

Après le tableau, demander : *« On applique tout / une partie / rien ? »* et attendre.

### 5. Appliquer

Sur validation, écrire uniquement les propositions retenues :

- nouveau fait/règle → créer/éditer `.claude/rules/<sujet>.md`
- nouveau savoir-faire → créer `.claude/skills/<nom>/SKILL.md` (frontmatter `name` + `description`, structure de cette
  skill comme modèle)
- convention transversale → éditer `CLAUDE.md`

### 6. Clôturer

- Récapituler ce qui a été écrit (liste des fichiers).
- Rappeler que ce sont des fichiers versionnés → **proposer** de committer (ne pas committer d'office ; suggérer
  `/commit`).

## Garde-fous

- **Ne jamais écrire avant validation** (étape 4 obligatoire) — comme la skill `commit`.
- **Ne pas sur-capitaliser** : une session saine peut ne produire aucune règle, et c'est un résultat valable.
- **Préférer modifier une règle existante** plutôt qu'en ajouter une qui se recoupe.
- **Ne pas committer sans demander.**
- Si la rétro ne porte que sur un contexte compacté/partiel, le **dire** au lieu de prétendre couvrir toute la session.
