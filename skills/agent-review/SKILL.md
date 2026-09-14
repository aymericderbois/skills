---
name: agent-review
description: >-
  Appelle Codex ou Cursor Agent dans une session neuve pour relire le travail produit par un agent
  ou un LLM : code, plan ou document. À utiliser pour obtenir une revue courte des défauts concrets
  du travail demandé.
argument-hint: "[travail à relire, référence Git ou fichiers, Codex ou Cursor]"
---

# Faire relire le travail de l'agent

Délimiter le travail à relire d'après la demande et la session : fichiers, plan ou changements
depuis une référence Git. Inclure les changements non commités et le contenu des nouveaux fichiers
concernés. Ne pas prendre toute la branche si la demande porte sur une seule tâche.

Prévoir le rapport dans `.task/<KEY>/<prefix>/review.md`, à côté du plan ou de la QA de la tâche.
Pour une revue du ticket entier, utiliser `.task/<KEY>/review.md`. Réutiliser le dossier choisi ;
sans ticket, prendre un nom court issu du travail. Créer le dossier avant l'appel.
Si un rapport existe, choisir un suffixe libre.

## Appeler le relecteur

Utiliser Codex ou Cursor selon le choix de l'utilisateur, sinon un des deux outils disponibles.
Lancer une seule revue, dans une session neuve et en lecture seule.

Préparer une consigne dans un fichier temporaire avec le besoin, les chemins du travail à relire,
la référence Git utile et les conventions du projet. Demander au relecteur de comparer le travail
au besoin et de rendre un rapport en français :

> Signale seulement les défauts concrets, par ordre de gravité. Pour chacun, donne l'emplacement,
> le cas qui pose problème et sa conséquence, en quelques phrases. Évite les conseils de style,
> les risques supposés, les longs extraits de code et le résumé des changements. Si tu ne trouves
> aucun défaut, dis-le en une phrase. Précise les limites qui empêchent une vérification.
> Vise 300 mots ; dépasse seulement si un défaut important l'exige.
> Relis toi-même, sans autre agent, et ne modifie aucun fichier.

Renseigner `review_prompt_file` avec le chemin de cette consigne et `review_report_file` avec celui
du rapport. Depuis la racine du projet, lancer une des commandes suivantes :

```sh
codex exec --sandbox read-only --model gpt-3.6 \
  -c 'model_reasoning_effort="medium"' \
  --output-last-message "$review_report_file" - < "$review_prompt_file"
```

```sh
cursor-agent --print --mode ask --model auto --output-format text \
  "$(cat "$review_prompt_file")" > "$review_report_file"
```

Si l'outil ou le modèle est indisponible, ou si la commande échoue, signaler la revue non réalisée.
Une sortie vide ne vaut pas une revue sans défaut. Ne pas changer le modèle demandé en silence.

## Rendre le rapport

Indiquer l'outil, le modèle demandé et le périmètre relu, puis les défauts et limites relevés.
Donner le lien vers le rapport et un bref bilan. Une demande de revue seule n'autorise pas
la correction des fichiers.
