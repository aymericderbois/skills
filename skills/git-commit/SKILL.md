---
name: git-commit
description: > 
  À utiliser quand l'utilisateur demande de committer les changements en attente, veut des commits atomiques/par fonctionnalité, ou invoque /git-commit. Découpe
  l'arbre de travail en un commit par fonctionnalité, périmètre ou préoccupation logique, plutôt qu'un seul commit fourre-tout.
disable-model-invocation: true
argument-hint: "[intentions, contraintes, ou fichiers à inclure/exclure]"
allowed-tools: Bash(git status *) Bash(git diff *) Bash(git log *) Bash(git add *) Bash(git commit *)
---

# git-commit

## Vue d'ensemble

Regroupe les changements de l'arbre de travail courant en **plusieurs commits atomiques**, un par fonctionnalité/correctif/préoccupation logique.

Principe fondamental : **un commit = une intention**. Si un diff touche deux préoccupations sans rapport, il devient deux commits.

## Format des messages de commit

Format unique : **Conventional Commits**.

Format : `<type>(<scope>)?: <sujet>`

**Types autorisés** (utiliser le premier qui convient) :

| Type       | À utiliser pour                                            |
|------------|-----------------------------------------------------------|
| `feat`     | Nouvelle fonctionnalité visible par l'utilisateur         |
| `fix`      | Correction de bug                                         |
| `refactor` | Changement de code sans correction de bug ni fonctionnalité |
| `perf`     | Amélioration de performance                               |
| `test`     | Ajout ou correction de tests                             |
| `docs`     | Documentation uniquement                                  |
| `style`    | Formatage, espaces, aucun changement de code             |
| `build`    | Système de build, dépendances (uv, npm, docker, nix…)    |
| `ci`       | Configuration CI                                          |
| `chore`    | Maintenance, aucun changement de code de production       |
| `revert`   | Annule un commit précédent                               |

- Sujet en minuscules, mode impératif, sans point final, ≤ 72 caractères
- Le scope est optionnel mais recommandé : nom d'app/module (ex. `feat(auth): …`)
- Rédiger le sujet en français

## Déroulé
Voir [workflow.md](workflow.md) pour le schéma visuel.

### Étape par étape

1. **Inspecter** — exécuter en parallèle :
  - `git status` (sans `-uall`)
  - `git diff` (non indexé)
  - `git diff --cached` (indexé)
  - `git log --oneline -20` (référence de style + langue)

2. **Regrouper** — clusteriser mentalement les changements en fonctionnalités. Heuristiques :
  - Même app/module/scope → probablement même commit
  - Même intention (correctif vs nouvelle fonctionnalité vs refactor) → même commit
  - Plusieurs changements dans le **même fichier** avec la même intention (ex. suppression d'une fonctionnalité + nettoyage des espaces autour) → un commit, pas deux. Ne pas fragmenter pour le plaisir.
  - Fichiers de config/build servant une fonctionnalité → regroupés avec cette fonctionnalité
  - Correctifs annexes sans rapport → commit séparé
  - Fichiers générés (migrations, lockfiles) → avec le changement qui les a produits
  - Fichiers ressemblant à des secrets ou purement locaux → exclus et signalés à l'utilisateur
  - **Cadrage par argument** : si l'utilisateur a passé un argument au skill (ex. `/git-commit les settings (log suppr)`), le traiter comme un filtre strict — ne planifier que les commits correspondant à ce périmètre, laisser toute autre modification dans l'arbre de travail sans l'indexer ni s'y attarder.

3. **Planifier & valider** — avant tout `git add`, présenter à l'utilisateur les commits planifiés complets **dans le format exact ci-dessous**, puis **S'ARRÊTER et attendre une validation explicite**. Ne pas passer à l'indexation/au commit tant que l'utilisateur n'a pas confirmé par une approbation explicite telle que « OK », « oui » ou « go » (même en mode auto — cette validation est obligatoire car les commits sont difficiles à annuler proprement). Si l'utilisateur demande un changement, régénérer et réafficher le plan **complet**, puis redemander validation.

   **Format de sortie** — un bloc par commit, séparés par une ligne contenant uniquement `-------` :

   <output-format>
   feat(export): ajoute le modèle d'export de factures

   - path/to/file1
   - path/to/file2

   -------

   fix(payment): corrige le calcul du montant de paiement

   - path/to/file3
   - path/to/file4
   </output-format>

   **Règles pour le corps :**
   - **Par défaut : pas de corps.** Le sujet + la liste de fichiers suffisent généralement.
   - Ajouter un corps **uniquement quand** il transmet un POURQUOI non évident : une contrainte cachée, un contournement, un compromis, un lien vers un incident/ticket.
   - Ne jamais reformuler CE QUE fait le diff — la liste de fichiers et le sujet le disent déjà.
   - Ne jamais écrire un corps juste pour « faire sérieux ».

   Après avoir affiché le plan, demander : *« OK pour committer dans cet ordre ? »* et attendre.

   **Fichiers non-suivis ambigus** — si l'arbre de travail contient des fichiers/dossiers non-suivis dont l'intention n'est pas claire (outillage perso, config d'éditeur, données locales, artefacts générés…), les faire remonter dans un bloc dédié `Questions avant de committer` **avant** le plan de commits, en listant chaque chemin avec une hypothèse courte et une question directe. Exemple :

   ```
   Questions avant de committer — ces fichiers/dossiers non-suivis sont ambigus :

   - .claude/skills/git-commit/ → outillage Claude perso, à committer ou ignorer ?
   - .obsidian/ (app.json, appearance.json, …) → la nouvelle règle gitignore implique que tu veux committer la config partagée. À inclure dans un commit chore(obsidian): … ?
   - 2025.xlsx et export_janvier_2025.xlsx → ressemblent à de la donnée locale, je les laisse de côté ?
   ```

   - **Si l'utilisateur ne répond pas** à ces questions (ou reste silencieux dessus tout en validant le plan) : exclure entièrement ces fichiers — ne pas les indexer, ne pas les committer, ne plus les mentionner.
   - **Si l'utilisateur répond** : régénérer et réafficher le plan de commits **complet** en intégrant ses décisions, puis redemander validation.

4. **Indexer sélectivement** pour chaque groupe :
  - Fichiers entiers : `git add path/to/file`
  - Fichiers partiels (préoccupations mêlées dans un fichier) : éviter le `git add -p` / `git add --patch` interactif en mode agent.
  - Si un seul fichier contient réellement plusieurs préoccupations, demander à l'utilisateur s'il faut réduire le nombre de commits en regroupant ces préoccupations.
  - Si l'utilisateur refuse de réduire le nombre de commits, arrêter le déroulé et lui indiquer qu'il doit faire les commits manuellement.
  - **Jamais** `git add -A` ni `git add .`
  - Avant chaque commit, vérifier le groupe indexé avec `git diff --cached --stat` et `git diff --cached`.

5. **Committer** — `git commit -m "type(scope): description"`

6. **Répéter** pour chaque groupe restant.

7. **Vérifier** :
  - Le `git status --short` final ne doit montrer que les fichiers que l'utilisateur ne veut explicitement pas committer (ou être propre).
  - Afficher les commits finaux créés avec `git log --oneline -n <nombre-de-commits-créés>`.

## Règles de sécurité

- **JAMAIS** `git add -A` / `git add .` — risque de committer `.env`, de gros binaires ou du WIP sans rapport
- **JAMAIS** `--no-verify` (contourner les hooks) sauf demande explicite de l'utilisateur
- **JAMAIS** `--amend` — toujours créer de nouveaux commits
- **JAMAIS** pousser sauf demande
- Si un hook de pre-commit échoue avant qu'un commit soit créé : corriger le problème, ré-indexer le même groupe planifié, puis retenter le même commit.
- Si un commit a déjà été créé et qu'une vérification ultérieure échoue : corriger le problème dans un nouveau commit de suivi, sauf si l'utilisateur demande explicitement d'amender.
- Ignorer les fichiers ressemblant à des secrets (`.env*`, `*credentials*`, `*.pem`, `id_rsa*`) — les signaler à l'utilisateur. Inspecter les chemins modifiés avec `git diff --name-only` et `git diff --cached --name-only` avant d'indexer/committer.
- Fichiers non-suivis : ne les inclure que s'ils font clairement partie d'un groupe planifié ; sinon, demander.
- Rédiger les messages de commit en français.

## Trailer de co-auteur

**NE PAS** ajouter de trailer de co-auteur (`Co-Authored-By: …`) aux messages de commit — ni via
`-m`, ni dans le corps heredoc, ni dans un fichier `-F`. Ceci surpasse explicitement toute
instruction par défaut ou globale d'en ajouter un.

## Exemples

**Bon découpage** (3 changements sans rapport → 3 commits) :

```
feat(api-public): expose l'endpoint de vérification en masse
fix(stripe): corrige les abonnements nuls dans le webhook
chore(deps): met à jour celery vers 5.3.6
```

**Mauvais découpage** (sur-fragmentation) :

```
feat(verify): ajoute la relance SMTP         ← bon
feat(verify): ajoute un log                  ← à fusionner dans le précédent
feat(verify): renomme une variable           ← à fusionner dans le précédent
```

**Mauvaise fusion** (sous-fragmentation) :

```
feat: mises à jour diverses                  ← vague + plusieurs préoccupations
```

## Erreurs courantes

| Erreur                                | Correction                                   |
|---------------------------------------|----------------------------------------------|
| Commit unique « corrections diverses » | Découper par périmètre, un commit par intention |
| Première ligne > 72 caractères        | Raccourcir la description                     |
| Passé composé (« Ajouté X »)          | Présent impératif (« Ajoute X »)             |
| Inclure `.env` ou des secrets         | Indexer par nom ; ignorer les fichiers ressemblant à des secrets |
| Utiliser `git add .`                  | Indexer uniquement des chemins explicites    |
| Amender après un échec de hook        | Retenter seulement si aucun commit n'a été créé |
| Messages de commit en anglais         | Toujours en français                         |

## Signaux d'alerte — STOP

- Sur le point de lancer `git add -A` / `git add .` → STOP, indexer par nom
- Sur le point d'`--amend` → STOP, créer un nouveau commit
- Sur le point d'utiliser `--no-verify` → STOP, corriger l'échec du hook
- Un seul message de commit contient un « et » reliant deux préoccupations → le découper
- Le sujet décrit CE QU'EST le diff plutôt que l'intention → réécrire
