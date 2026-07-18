---
name: python-upgrade-package
description: >
  Monte de version un ou plusieurs paquets Python d'un projet géré avec uv, en évaluant l'impact des changements
  cassants avant d'appliquer la montée. À utiliser quand l'utilisateur veut monter une dépendance Python, corriger
  une CVE remontée par uv audit, rafraîchir le lockfile, ou invoque /python-upgrade-package.
argument-hint: "[paquet(s) et version(s) cible, ex. « gunicorn 23 » ou « requests django »]"
allowed-tools: Read, Grep, Glob, Edit, Agent, Bash(uv *), Bash(uvx *), Bash(git diff *), Bash(git grep *), Bash(git log *), WebSearch, WebFetch, AskUserQuestion
---

# python-upgrade-package

Monter de version un ou plusieurs paquets Python **avec uv**, en évaluant l'impact réel de la montée sur le projet
avant de l'appliquer, puis en la validant par les tests. `$ARGUMENTS` = les paquets à monter (et, si précisé, la
version cible de chacun).

Le risque, c'est le **changement cassant** qu'on ne voit pas. Ce skill force donc l'ordre :
- *comprendre ce qui change*
- *vérifier si le projet l'utilise*
- *appliquer*
- *prouver que ça tient*.

## Workflow

Copier cette checklist dans la réponse et la cocher au fil de l'eau :

```
- [ ] 1. Cadrer     : lister les paquets, version courante → cible, pin exact vs plage
- [ ] 2. Changelogs : prévisualiser les co-montées, puis lire les entrées entre courant et cible, extraire les cassures
- [ ] 3. Impact     : confronter chaque changement cassant à l'usage réel dans le code
- [ ] 4. Décision   : impact → corriger dans la foulée ou s'arrêter ; sinon continuer
- [ ] 5. Appliquer  : la montée via uv
- [ ] 6. Lock       : git diff uv.lock — vérifier ce qui a bougé
- [ ] 7. Tests      : suite de tests + test de démarrage si binaire de service
- [ ] 8. Audit      : uv audit --frozen (si la montée visait une CVE)
- [ ] 9. Rapport    : synthèse + indexer pyproject.toml et uv.lock (ne pas commiter)
```

Les étapes 2 à 4 se répètent **par paquet** ; l'étape 5 les applique ensemble.

**Délégation.** Quand plusieurs paquets sont à analyser (demandés + co-montées sensibles relevées par le dry-run),
lancer **dans un seul message** un subagent `Agent` (`subagent_type=general-purpose`) par paquet, chacun avec le
prompt [references/analyse-paquet.md](references/analyse-paquet.md) rempli (paquet, versions, racine du projet,
rôle). Chaque agent rend le tableau compact défini dans le prompt — jamais de copier-coller massif. Un seul paquet
sans co-montée → faire les étapes 2–3 inline, sans subagent. Dans tous les cas, les étapes 4 et suivantes restent
dans la conversation principale.

### 1. Cadrer

Rassembler d'abord les informations qui serviront ensuite.

- **Version courante** : lire `pyproject.toml` (contrainte) et `uv.lock` (version verrouillée). En cas de doute :
  `uv tree --package <pkg>`.
- **Version cible** : depuis `$ARGUMENTS`. Non précisée ou « dernière » → dernière stable.
- **Forme de la contrainte** : pin exact (`pkg==X`) ou plage (`pkg>=X`, `>=X,<Y`) — ça change la mécanique (étape 5).

### 2. Changelogs

**D'abord, lister les co-montées.** Une montée entraîne souvent d'autres paquets (bornes minimales relevées,
dépendances partagées). Prévisualiser la résolution **avant** toute lecture de changelog :

- pin exact : `uv add --dry-run 'pkg==Y'` (avec les extras s'il y en a) ;
- plage : `uv lock --dry-run --upgrade-package pkg`.

Relever **tous** les paquets qui bougeraient. Chaque co-montée d'un paquet sensible (serveur, worker, ORM, crypto,
sérialisation, tout paquet importé par le code) suit les étapes 2 à 4 au même titre que les paquets demandés — pas
seulement un contrôle après coup.

Pour chaque paquet, lire **uniquement les entrées strictement entre la version courante (exclue) et la cible
(incluse)**. Pour un saut de plusieurs majeures, cumuler les cassures de chaque palier.

- Trouver la source : page PyPI `https://pypi.org/project/<pkg>/` → liens projet → GitHub *Releases*,
  `CHANGELOG.md` / `CHANGES.rst`. `WebFetch` sur ces URLs ; `WebSearch "<pkg> <cible> changelog breaking changes"`
  en complément.
- Repérer et noter : **Breaking / Removed / Deprecated / Backwards incompatible**, valeurs par défaut changées,
  signatures modifiées, montée du Python minimum requis.
- Ignorer le reste (fonctionnalités, performances, corrections internes) : seul ce qui peut casser le projet compte ici.

### 3. Impact sur le projet

Pour chaque changement cassant repéré, vérifier si le projet **utilise réellement** l'API touchée — **ne pas
présumer**.

- `git grep -n "<symbole|import|api>"` sur le code (imports, appels, sous-classes).
- Ne pas oublier : fichiers de conf, options CLI, clés de configuration, points d'entrée / `[project.scripts]`.
- Classer chaque cassure : **touché** (à traiter) ou **non touché** (sans effet ici).

### 4. Décision

- **Aucun impact** → continuer.
- **Impact** → ne **jamais** monter la version en cassant silencieusement. Deux options, selon l'ampleur :
  - petit et directement lié → corriger le code dans la même tâche ;
  - large ou incertain → s'arrêter et demander la marche à suivre (`AskUserQuestion`).

### 5. Appliquer (mécanique uv)

- **Pin exact `pkg==X`** — recommandé : `uv add 'pkg==Y'` (met à jour la contrainte dans `pyproject.toml`,
  reverrouille et synchronise en une commande). Conserver les extras : `uv add 'pkg[extra]==Y'`. Plusieurs d'un coup :
  `uv add 'a==1' 'b==2'`.
- **Plage `pkg>=X`** — monter dans la plage sans toucher la contrainte : `uv lock --upgrade-package pkg` puis
  `uv sync`. Pour dépasser la borne haute : éditer la contrainte (ou `uv add`) puis reverrouiller.
- **Ne pas** lancer `uv lock --upgrade` (sans `--package`) sauf demande explicite : il monte **tout** le graphe.

### 6. Vérifier le lock

`git diff uv.lock` : confirmer que le diff correspond aux co-montées prévues à l'étape 2 — rien de plus. Tout paquet
qui bouge sans avoir été prévu retourne à l'étape 2.

### 7. Tests + smoke-test

- Lancer la suite de tests **du projet** en suivant les bonnes pratiques définies dans le projet.
- Cibler d'abord les modules qui utilisent le paquet, puis élargir.
- Si le paquet fournit un **binaire de service** (serveur WSGI/ASGI, worker, CLI), ajouter un smoke-test,
  un import réussi ne prouve pas que le service démarre.

### 8. Audit (si CVE)

Quand la montée visait une vulnérabilité : `uv audit --frozen` et confirmer que le paquet a **quitté** le rapport.

### 9. Rapport

Synthèse compacte, un tableau par lot :

| Paquet | Courant → Cible | Changements cassants | Impact projet | Tests |
|--------|-----------------|----------------------|---------------|-------|

Puis **indexer** `pyproject.toml` et `uv.lock` (`git add`). **Ne pas commiter** : c'est la décision du mainteneur ;
une montée (ou un lot cohérent) = un commit.

## Garde-fous

- L'ordre *changelog → impact → application* n'est pas négociable : appliquer d'abord, c'est découvrir la cassure
  en production.
- Une montée silencieuse qui casse une API utilisée est un échec, pas un gain de temps.
- Ne jamais monter tout le graphe (`uv lock --upgrade`) par commodité.
