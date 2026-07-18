# Prompt subagent : analyse changelog + impact d'un paquet

Prompt à remplir puis à passer tel quel à un subagent `Agent` (`subagent_type=general-purpose`).
Remplacer les placeholders `{…}` avant l'envoi.

---

Tu analyses la montée du paquet Python `{paquet}` de `{version courante}` vers `{version cible}` dans le projet
situé à `{racine du projet}`. Rôle de ce paquet dans le lot : {demandé | co-montée entraînée par {paquet parent}}.

**Lecture seule** : tu n'édites aucun fichier, tu ne lances ni `uv`, ni tests, ni aucune commande qui modifie l'état.

## 1. Changelogs

Lis **uniquement les entrées strictement entre la version courante (exclue) et la cible (incluse)**. Pour un saut
de plusieurs majeures, cumule les cassures de chaque palier.

- Trouver la source : page PyPI `https://pypi.org/project/{paquet}/` → liens projet → GitHub *Releases*,
  `CHANGELOG.md` / `CHANGES.rst`. `WebFetch` sur ces URLs ; `WebSearch "{paquet} {version cible} changelog breaking
  changes"` en complément.
- Repérer et noter : **Breaking / Removed / Deprecated / Backwards incompatible**, valeurs par défaut changées,
  signatures modifiées, montée du Python minimum requis.
- Ignorer le reste (fonctionnalités, performances, corrections internes) : seul ce qui peut casser le projet compte.

## 2. Impact sur le projet

Pour chaque cassure repérée, vérifie si le projet **utilise réellement** l'API touchée — **ne présume pas**.

- `git grep -n "<symbole|import|api>"` sur le code (imports, appels, sous-classes).
- Ne pas oublier : fichiers de conf, options CLI, clés de configuration, points d'entrée / `[project.scripts]`,
  Dockerfile / Procfile / CI.
- Classer chaque cassure : **touchée** ou **non touchée**.

## 3. Sortie attendue

Rends **exactement** ceci, rien de plus — pas de copier-coller de changelog ni de dump de fichiers :

```
Paquet : {paquet} {version courante} → {version cible}
Verdict : aucun impact | impact

| Cassure | Touché ? | Fichiers concernés (chemin:ligne) |
|---------|----------|-----------------------------------|

Python minimum requis par la cible : <version>
Sources : <URLs consultées>
```
