# Format du rapport HTML

Le rapport est **un seul fichier `.html` autonome**. Tailwind (CDN) porte la mise en page ; une petite couche
`<style>` définit les **variables CSS** de couleur, la typo et les composants sur mesure (diff, callout, badge, cases à
cocher). Mermaid (CDN, en module ESM) rend les rares diagrammes. Aucune autre ressource, **aucune police via
CDN** — on reste sur des polices système pour que le fichier s'affiche correctement même hors-ligne.

## Scaffold

À copier tel quel, puis remplir `<main>` avec les blocs ci-dessous.

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>[titre du rapport]</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      const dark = matchMedia("(prefers-color-scheme: dark)").matches;
      mermaid.initialize({ startOnLoad: true, theme: dark ? "dark" : "neutral", securityLevel: "loose" });
    </script>
    <style>
      :root {
        --bg:#fbfbfa; --surface:#ffffff; --border:#e7e5e4; --ink:#1c1917; --muted:#78716c;
        --accent:#4f46e5;                    /* ← retune selon le sujet (voir « Palette ») */
        --good:#059669; --warn:#d97706; --bad:#dc2626;
        --code-bg:#f5f5f4;
        --font-sans:ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
        --font-serif:ui-serif,Georgia,Cambria,"Times New Roman",serif;
        --font-mono:ui-monospace,"SF Mono","JetBrains Mono",Menlo,Consolas,monospace;
      }
      @media (prefers-color-scheme: dark) {
        :root {
          --bg:#0c0a09; --surface:#1c1917; --border:#292524; --ink:#f5f5f4; --muted:#a8a29e;
          --accent:#818cf8; --code-bg:#1c1917;
        }
      }
      body { background:var(--bg); color:var(--ink); font-family:var(--font-sans);
             -webkit-font-smoothing:antialiased; }
      h1,h2,h3 { font-family:var(--font-serif); text-wrap:balance; letter-spacing:-.01em; }
      .eyebrow { font-size:.72rem; text-transform:uppercase; letter-spacing:.12em; color:var(--muted); }
      .card { background:var(--surface); border:1px solid var(--border); border-radius:.75rem; }
      .rule { height:1px; background:var(--border); }
      /* code + diff */
      pre.code, pre.diff { font-family:var(--font-mono); font-size:.82rem; line-height:1.6;
             background:var(--code-bg); border:1px solid var(--border); border-radius:.5rem;
             padding:.9rem 1rem; overflow-x:auto; }
      pre.diff .add { display:block; background:color-mix(in srgb,var(--good) 15%,transparent);
             color:var(--good); }
      pre.diff .del { display:block; background:color-mix(in srgb,var(--bad) 15%,transparent);
             color:var(--bad); }
      pre.diff .ctx { display:block; color:var(--muted); }
      /* callout */
      .callout { border-left:3px solid var(--warn); border-radius:.4rem;
             background:color-mix(in srgb,var(--warn) 8%,transparent); padding:.75rem 1rem; }
      .callout.info { border-left-color:var(--accent);
             background:color-mix(in srgb,var(--accent) 8%,transparent); }
      /* badge */
      .badge { display:inline-flex; align-items:center; gap:.35rem; font-size:.72rem; font-weight:600;
             padding:.15rem .55rem; border-radius:999px; border:1px solid var(--border); color:var(--muted); }
      .badge.ok  { color:var(--good); border-color:color-mix(in srgb,var(--good) 40%,var(--border)); }
      .badge.acc { color:var(--accent); border-color:color-mix(in srgb,var(--accent) 40%,var(--border)); }
      /* checklist */
      ul.check { list-style:none; padding:0; margin:0; display:flex; flex-direction:column; gap:.4rem; }
      ul.check li { display:flex; gap:.6rem; align-items:baseline; }
      ul.check li::before { content:"☐"; color:var(--muted); }
      ul.check li.done::before { content:"☑"; color:var(--good); }
      /* mermaid : neutraliser le fond blanc de la lib */
      .mermaid { background:var(--surface); border:1px solid var(--border); border-radius:.5rem; padding:1rem; }
    </style>
  </head>
  <body>
    <main class="max-w-4xl mx-auto px-6 py-12 space-y-10">
      <!-- header, sections… -->
    </main>
  </body>
</html>
```

## Palette et typo — éviter le rendu « généré par IA »

Le rendu Tailwind-CDN par défaut trahit vite le « design IA ». Faire des choix, pas des réflexes.

- **Neutre choisi, pas subi** : le neutre par défaut ci-dessus (`stone`) est légèrement chaud. Garde un neutre
  cohérent ; ne mélange pas trois familles de gris.
- **Un seul accent, adapté au sujet** : retune `--accent` (et sa version sombre) en fonction du contenu — un
  rapport de sécurité n'a pas le même accent qu'un plan produit. Un seul accent ; `--good`/`--warn`/`--bad`
  restent réservés au sens (statut, diff), ce ne sont **pas** des accents.
- **Typo à deux rôles** : serif système pour les titres (donne un ton éditorial), sans-serif système pour le
  corps, mono pour le code. Texte courant lisible (~65-75 caractères de large, d'où `max-w-4xl`).
- **Thème clair ET sombre** : les deux passent par les *tokens* ; vérifier que l'accent et les contrastes tiennent
  dans les deux. Mermaid bascule via le paramètre `theme` du scaffold.

**À éviter** (marqueurs de design générique) : le combo cream `#F4F1EA` + serif + terracotta ; le dégradé
violet→bleu en bandeau ; les marqueurs numérotés `01 / 02 / 03` **décoratifs** (n'en mettre que si le contenu est
réellement une séquence) ; tout centrer ; `rounded-lg` partout ; emoji en puce de section.

## Blocs (gabarit unique)

Un seul gabarit souple : pioche les blocs utiles selon le contenu. Prose sobre, pas de remplissage.

### Header

Titre, date du jour, et une ligne de contexte. Pas de paragraphe d'introduction cérémonieux.

```html
<header class="space-y-2">
  <p class="eyebrow">[type de rapport — ex. Plan · Changements · Audit]</p>
  <h1 class="text-3xl font-bold">[titre]</h1>
  <p class="text-sm" style="color:var(--muted)">[date] · [contexte en une ligne]</p>
</header>
```

### Section

```html
<section class="space-y-4">
  <h2 class="text-xl font-semibold">[titre de section]</h2>
  <p class="leading-relaxed">[texte — reprend les mots du contenu source]</p>
</section>
```

### Liste à cocher (contraintes d'acceptation, critères)

Ajouter `class="done"` sur les `<li>` déjà satisfaits.

```html
<ul class="check">
  <li>[condition à satisfaire]</li>
  <li class="done">[condition déjà satisfaite]</li>
</ul>
```

### Bloc de code

```html
<pre class="code"><code>[code — échapper &lt; &gt; &amp;]</code></pre>
```

### Bloc de diff (pour un rapport de changements)

Une ligne = un `<span>` de classe `add` / `del` / `ctx`. Échapper `<`, `>`, `&`.

```html
<pre class="diff"><span class="ctx"> contexte inchangé</span><span class="del">-ancienne ligne</span><span class="add">+nouvelle ligne</span></pre>
```

### Callout

```html
<div class="callout">[avertissement / point d'attention]</div>
<div class="callout info">[information / note]</div>
```

### Badge / pill (statut)

```html
<span class="badge ok">Fait</span>
<span class="badge acc">En cours</span>
<span class="badge">À faire</span>
```

### Diagramme Mermaid (optionnel, rare)

Uniquement quand un schéma aide vraiment (dépendances, flux) — sinon s'en passer.

```html
<pre class="mermaid">
flowchart LR
  A[planify] --> B[report-render]
  B --> C[.reports/*.report.html]
</pre>
```

## Rappels

- **Contenu réel** : aucun texte de remplissage, aucune section vide.
- **Débordement horizontal** : tout contenu large (code, diff, table, diagramme) scrolle dans son propre
  conteneur (`overflow-x:auto` est déjà sur `pre`) — le corps de page ne scrolle jamais latéralement.
- **Échapper** `<`, `>`, `&` dans les blocs de code et de diff.
