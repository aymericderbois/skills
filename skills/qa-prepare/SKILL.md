---
name: qa-prepare
description: Prépare la QA d'un travail réalisé, avec les parcours à vérifier, les tests, les documents et les données utiles. À utiliser pour définir la QA d'une tâche ou compléter ses vérifications, même sans plan préalable.
argument-hint: "[tâche réalisée et QA souhaitée]"
---

# Préparer la QA

Partir du travail décrit par l'utilisateur. Lire le changement, le ticket, le plan et la QA existante
s'ils sont disponibles. Demander seulement ce qui manque pour savoir quel résultat vérifier.

Utiliser `.task/<KEY>/qa.md`, commun aux demandes du ticket, où `<KEY>` est son numéro ou son nom
validé. Réutiliser le dossier déjà choisi.

## Préparer ce qui aide à vérifier

Regrouper les cas dans une section par demande du ticket : `D1`, `D2`, etc., avec son titre.
Sans lien clair avec une demande numérotée, utiliser un titre qui décrit le travail à vérifier,
sans inventer de numéro.

Si `qa.md` existe, compléter ou mettre à jour la section de la demande concernée ; créer sa section
si elle manque. Conserver les autres sections et les résultats encore valables. Décocher les cas
que le changement remet en cause et les marquer « à vérifier », en gardant leur ancien résultat.

Pour chaque cas utile, préciser :

- la préparation : environnement, droits, données et fichiers nécessaires ;
- les actions à suivre, avec les mots de l'interface ;
- le résultat attendu ;
- une case de vérification et le résultat observé, initialement « à vérifier ».

Créer les supports utiles au cas : tests, parcours expliqués, PDF, fichiers à importer,
ou scripts pour peupler la base. Les ranger dans le même dossier, sous `assets/`, `tests/`
ou `scripts/` si besoin. Donner les liens et les commandes pour les utiliser. Le format dépend
du travail à vérifier ; ne pas produire un support qui n'aide à vérifier aucun cas.

## Vérifier si demandé

Une demande de préparation produit les supports. Si l'utilisateur demande aussi de passer la QA,
exécuter les cas accessibles et noter les résultats. Exécuter les scripts de peuplement seulement
sur une base de développement ou de test clairement désignée.

Cocher seulement les cas vérifiés avec succès. Garder les échecs et les cas non exécutés visibles.
Terminer avec les liens vers la QA et ses supports, puis les résultats s'il y en a.

## Format de `qa.md`

```markdown
# QA — <KEY>

Ticket : [<KEY>](ticket.md).

## D1 — <titre de la demande>

### <cas à vérifier>

- Préparation : <environnement, droits, données et liens vers les supports>.
- Actions : <parcours à suivre>.
- Attendu : <résultat attendu>.
- Résultat observé : à vérifier.
- [ ] Vérifié avec succès.

## D2 — <titre de la demande suivante>

<Cas de cette demande, au même format.>
```
