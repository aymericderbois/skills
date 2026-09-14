---
name: ticket-prepare
description: Importe un ticket Jira et ses pièces jointes, ou définit un ticket avec l'utilisateur par une discussion centrée sur le produit. À utiliser pour préparer un travail dans un dossier local de ticket.
argument-hint: "[ticket Jira ou besoin à définir]"
---

# Préparer un ticket

Utiliser `.task/<KEY>/` à la racine du projet : `<KEY>` est le numéro du ticket.
Sans numéro de ticket, proposer deux ou trois noms courts issus du besoin et attendre que
l'utilisateur valide le nom avant de créer le dossier. En reprise, garder le nom déjà validé.
Réutiliser le dossier choisi dans la session et lire le ticket existant avant de le compléter.
Si plusieurs dossiers conviennent, demander lequel utiliser.

## Définir le besoin

- **Depuis Jira** : récupérer le ticket avec l'outil disponible. Conserver sa référence, son titre
  et sa description dans `ticket.md`. Lire les pièces jointes utiles, les ranger dans `assets/`
  et les lier depuis le ticket. Signaler ce qui reste inaccessible.
- **Depuis une discussion** : préciser avec l'utilisateur le but, les personnes concernées,
  les parcours, les règles et les résultats attendus. Poser les questions produit encore utiles,
  sans choix techniques. Rédiger `ticket.md` quand le besoin est assez clair ; garder les points
  ouverts comme tels, sans décider à la place de l'utilisateur.

Dans les deux cas, identifier et numéroter les demandes dans `ticket.md` (`D1`, `D2`, etc.)
pour que les plans puissent les citer. Garder les numéros existants en reprise et attribuer
les suivants aux nouvelles demandes.

Pour un ticket importé, conserver les formulations, l'ordre, les listes, les liens et les détails.
Ajouter les repères `D1`, `D2`… aux demandes sans réécrire leur contenu. Garder le contexte et les
notes à leur place, sans les transformer en demandes. Ne pas inventer de critères d'acceptation
absents du ticket. Distinguer les précisions ajoutées ensuite.

Réserver le titre de niveau `#` au ticket et utiliser `## [ ] D1 — …` pour les demandes.
Cocher une demande seulement quand elle a été vérifiée, pas dès que son développement est terminé.

<EXAMPLE-OUTPUT>
## [ ] D1 — Mise en place OAuth2

Mettre en place l'authentification OAuth2 pour l'intégration externe

## [ ] D2 — Champs de recherche avancé

Ajouter un champ de recherche avancé pour les clients

- Permettre de filtrer par nom, email et statut
- Ajouter des opérateurs booléens (AND, OR, NOT)
- Supporter les recherches partielles (wildcards)
- Ajouter des filtres par date de création et date de modification

## [ ] D3 — Notifications email/webhook

Gérer les notifications par email et webhook
</EXAMPLE-OUTPUT>

Le ticket décrit le besoin et le comportement attendu. Inclure les critères d'acceptation lorsqu'ils
figurent dans le ticket ou ont été définis avec l'utilisateur. Ne pas créer ni modifier de ticket
dans Jira sans demande.

## Ranger le travail

Chaque tâche du ticket a son sous-dossier, avec un nom court en minuscules et tirets :

```text
.task/PROJ-123/
  ticket.md
  qa.md
  assets/
  recherche-client/
    plan.md
    tech.md
    review.md
```

Créer les dossiers et fichiers au fil des besoins. Terminer avec le lien vers `ticket.md`
et les éventuels points à préciser.
