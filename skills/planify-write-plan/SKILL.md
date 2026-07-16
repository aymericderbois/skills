---
name: planify-write-plan
description: "Rédacteur interne du skill planify : met en forme le récapitulatif d'un plan (produit par l'interrogatoire de planify) en deux fichiers dans .planify/ — un plan.md lisible (tâches + contraintes d'acceptation) et un tech.md dense. Invoqué par planify via l'outil Skill ; ne se déclenche pas seul et n'a pas de slash command. Ne repose aucune question, ne prend aucune décision."
user-invocable: false
allowed-tools: Read, Write, Edit, Glob
---

# Rédaction du plan

Tu mets en forme, tu ne décides rien. Tu reçois le récapitulatif structuré assemblé par `planify` (dans la
conversation) et tu le coules dans deux fichiers. Tu ne reposes aucune question, ne prends aucune décision, et
n'inventes aucune tâche ni contrainte absente du récapitulatif.

## Règles

- **Tu dois** faire des phrases courtes et concretes.
- **Tu dois** utiliser des termes simples et précis.
- **Tu dois** éviter les mots-valise et les expressions trop générales.
- **Tu dois** faire des retours à la ligne pour que le fichier soit facile à lire.

## Résolution du préfixe

1. Préfixe = un slug court dérivé de la demande : en minuscules avec tirets, 2 à 5 mots signifiants.
2. Créer le dossier `.planify/` s'il n'existe pas.

**Reprise** : si `.planify/<prefix>.plan.md` existe déjà, le mettre à jour avec `Edit` (fusionner les changements
du récapitulatif, préserver le reste) plutôt que de l'écraser aveuglément.

## `<prefix>.plan.md` — lisible, facile à relire

```markdown
# Plan — [titre]

## Contexte

[le pourquoi, couvrant la ou les fonctionnalités]

> Hors périmètre : [ce qu'on exclut explicitement — retirer la ligne s'il n'y a rien]

## T1 — [titre court]

[description de ce qu'il faut faire — technique et court extrait de code si cela aide à visualiser]

Contraintes d'acceptation :
- [ ] [condition à satisfaire]
- [ ] [condition à satisfaire]

## T2 — …
```

Règles : les tâches sont ordonnées par leur position (pas de champ de dépendance). Une tâche correspond à un
changement de la taille d'un commit, facile à relire. Les contraintes forment une liste à cocher de conditions
vérifiables (observables ou non : performance, sécurité, invariant) — pas de format « lorsque/alors » imposé.

## `<prefix>.tech.md` — index technique dense pour l'agent

N'est pas relu par un humain. La densité et la précision priment.

```markdown
# Technique — [titre]

## Fichiers concernés
### À créer
- `apps/<app>/...`
### À modifier
- `apps/<app>/models.py` — [quoi]

## Motifs réutilisés
- [nom] — `apps/.../foo.py` — comment il s'applique ici.

## Décisions techniques (mini-ADR)
### D1 — [titre]
**Contexte** · **Décision** · **Alternatives écartées** (et pourquoi) · **Conséquences**

## Éléments transverses
- migrations · permissions et sécurité · drapeaux de fonctionnalité · données de test utiles
```

Retirer une section si le récapitulatif n'a rien à y mettre.

## Fin

Signaler les deux chemins écrits et rappeler que l'implémentation se fait en **mode plan natif** sur
`<prefix>.plan.md`.
