# SKILLs

# SKILL
## Planify

Il faut revoir ce SKILL pour le simplifier.
Dans l'idée, il faut un seul SKILL qui permet d'aller de bout en bout dans le cycle de développement.
- analyser le ticket (ou la question/sujet/idée/...) et poser des questions pour clarifier le besoin
- vérifier les cas d'utilisation, les effets de bors possibles, les contraintes, ... pour faire évoluer le besoin
- préparer des specs avec ces informations (courtes, lisbible, peu verbeuse, pas de mot-valise, ...)
- préparer des tâches techniques qui doivent être courte à réaliser pour permettre une review simple par le développeur
    - mettre des exemples de code dans les tâches
    - doit déjà être précis sur ce qui est à faire ...
- préparer un fichier de contexte avec toutes les informations récupérées pour permettre à l'agent qui fera le développement d'aller plus vite
- générer un fichier spec + tasks et un fichier contexte

Ensuite le développeur pourra faire : /plan les développements de @cheminfichier.md

Il est important que le fichier spec + tasks soit :
- lisible aisément par un humain
- linké avec le fichier contexte
- donne des instructions pour la planification (ne pas être trop verbeux, mettre des exemples de code, ...)

Il est important que pour la création des tâches (qui sont techniques), le SKILL fasse appel aux SKILLs de `ipt-doc:*` qui contiennent
des informations techniques sur le codebase ainsi que les bonnes pratiques.

Je souhaiterais que le SKILL soit "simple", comme c'est le cas pour `grilling` et `grill-me`
J'aime bien aussi le côté "brainstorming" de superpowers

Par ailleurs, je souhaite un seul skill invocable via `/planify` **mais** il peut y avoir plusieurs skill model-invocables qui pourrait
etre utilisée par `/planify`. A voir ce qui pourrait être utile.

### Format 

Pour la partie tâches 

```Markdown

# Nom de la feature

> Un bloc = une sous-tâche/story à créer sous **PROJ-1234**, dans l'ordre ci-dessous (dépendances). Remplacer les
> « Tranche N » de la section « Bloqué par » par les vraies clés Jira une fois créées.
> Contexte complet : [PRD](PROJ-1234-prd.md) · [cahier des charges](PROJ-1234-cahier-des-charges.md) ·
> [ADR 0001](adr/0001-import-direct-document-formate.md) · vocabulaire : [`CONTEXT.md`](../CONTEXT.md).
> Note transverse : toute la fonctionnalité est derrière le feature flag (Tranche 5), **créé désactivé** ; il reste
> off jusqu'à la recette, donc aucun document n'est envoyé en prod pendant l'intégration (les tranches 2 et 3
> peuvent atterrir après le squelette sans régression réelle).

---

## [ ] Tâche 1 — Import via l'API (endpoint complet, walking skeleton)

### Ce qu'il faut construire

Un endpoint API authentifié qui accepte **un seul** fichier de document déjà formaté (PDF ou XML structuré)
et l'**importe de bout en bout via le service externe**. Le parcours : extraire les métadonnées, les convertir
en JSON normalisé via le service externe (ce qui détecte aussi le format et la version), le valider, l'enregistrer,
puis récupérer le visuel PDF généré par le service. En cas de succès, créer un `Document` **non brouillon de type
import**, peuplé depuis le JSON normalisé (montants, dates, tiers, référence d'origine), stocker l'identifiant
côté service externe et le visuel, puis synchroniser vers une **entrée en attente au statut « À traiter »**.
Renvoyer l'identifiant du document. En cas d'échec de validation, renvoyer la liste des erreurs sans rien créer.
Si l'organisation n'a pas le service actif, renvoyer un refus.

Introduire un **service d'import dédié** qui encapsule la séquence du SDK (convert → validate → create → pdf)
et traduit les erreurs du SDK (400 / 404 / 422 / 5xx) en une liste plate de messages. Étendre le modèle de données
avec l'identifiant externe et le stockage du visuel PDF (migration).

### Critères d'acceptation

- [ ] POST avec un fichier valide → `201` avec `{ document_id }`.
- [ ] Un `Document` non brouillon de type `import` est créé, peuplé depuis le JSON normalisé (montants, dates, tiers,
      référence d'origine).
- [ ] L'identifiant externe et le visuel PDF sont stockés sur le document.
- [ ] Une entrée en attente est créée au statut « À traiter », statut import initialisé, portant le visuel.
- [ ] Erreurs de validation → `400` avec `{ errors: [...] }`, et **rien n'est persisté**.
- [ ] Organisation sans service actif → `403`.
- [ ] L'endpoint accepte exactement un fichier (contrat multi-fichiers supprimé).
- [ ] Test d'intégration au niveau de l'endpoint, **service externe mocké** (seam unique), couvrant succès, erreur de
      validation et éligibilité.

### Bloqué par

Aucun — démarrable immédiatement.

---

## [ ] Tâche 2 — Protection de la numérotation (parades A + B2)

### Ce qu'il faut construire

Garantir que l'import d'un document déjà formaté **n'impacte jamais la numérotation interne**. Deux mesures :
**(A)** le document importé porte sa référence d'origine, de sorte que la validation ne génère aucun numéro ;
**(B2)** les documents importés sont exclus de **toutes** les requêtes de comptage des compteurs — la génération
du prochain numéro en mode « ignorer le type de document », le calcul de la date de première utilisation (ancrage
de séquence), et le comptage de verrouillage du compteur.

Point d'attention : un compteur est **auto-assigné** à tout document à la création ; la parade B2 rend cette
assignation inoffensive côté numérotation.

### Critères d'acceptation

- [ ] Importer un document ne consomme aucun numéro de séquence interne.
- [ ] Après import d'un document sur un compteur « ignorer le type de document », le document suivant
      conserve son numéro attendu (pas de décalage +1).
- [ ] La date d'ancrage de séquence n'est pas affectée par un document importé.
- [ ] Un compteur n'est pas considéré « verrouillé » du seul fait qu'un document importé y est rattaché.
- [ ] Les tests couvrent un compteur « ignorer le type » et un compteur à séquence globale.

### Bloqué par

- Tranche 1

```

## artifact-design

Matt Pocock a un SKILL qui s'appel `artifact-design` qui permet de générer des rapports au format HTML disponible en tant qu'artefact sur
claude code.

Je trouve ça pas mal, à voir comment on pourrait en faire un similaire et l'utiliser dans nos propres SKILLs.


# TODO Dev rules

Ce document est une base pour la mise en place futur de règles des développements pour les projets Python et Django.

## Python

### Imports

- Les imports doivent être triés via la configuration de Ruff.
- Les imports doivent être positionnés en haut du fichier **sauf** en cas d'erreurs (circular imports, performance, etc.).

## Django