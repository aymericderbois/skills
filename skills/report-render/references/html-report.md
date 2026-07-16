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
    <!-- Couche d'annotation : coller le bloc « Couche d'annotation (retours) » avant </body>. Présent sur CHAQUE rapport. -->
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

`data-annote` sert d'**ancre** aux retours : pour un plan, c'est l'ID de tâche (`T3`) ; sinon un slug du titre.
La couche d'annotation s'en sert pour nommer chaque retour (`## Retour N — T3`).

```html
<section class="space-y-4" id="T3" data-annote="T3">
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

### Couche d'annotation (retours)

**Présente sur chaque rapport.** À coller telle quelle avant `</body>`. Elle laisse le lecteur **sélectionner un
passage et y attacher un commentaire**, puis **copier tous ses retours** dans le presse-papier pour les coller dans
`planify`. Elle est **inerte** : elle ne modifie jamais le contenu rendu (surlignage via l'API Custom Highlight, sans
toucher au DOM), persiste en `localStorage`, et se dégrade proprement (si le presse-papier est bloqué en `file://`,
une modale présente le texte présélectionné ; si l'API de surlignage manque, le retour reste listé dans le panneau).

Format produit par le bouton « Copier les retours » — **c'est le contrat lu par `planify`** :

```markdown
# Retours — <titre du rapport>
_source : <slug>.report.html_

## Retour 1 — T3
> passage cité exact
Commentaire : découper en deux, la migration mérite sa propre tâche.
```

```html
<style>
  /* Couche d'annotation (retours) */
  ::highlight(annote-hl) { background-color: color-mix(in srgb, var(--accent) 22%, transparent); }
  .annote-ui { font-family: var(--font-sans); }
  .annote-selbtn { position: absolute; z-index: 50; font-size: .8rem; font-weight: 600;
    padding: .3rem .6rem; background: var(--accent); color: #fff; border: none; border-radius: .4rem;
    box-shadow: 0 2px 8px rgba(0,0,0,.18); cursor: pointer; }
  .annote-pop, .annote-panel, .annote-modal { position: fixed; z-index: 60; background: var(--surface);
    color: var(--ink); border: 1px solid var(--border); border-radius: .6rem; box-shadow: 0 8px 30px rgba(0,0,0,.22); }
  .annote-pop { width: 18rem; padding: .75rem; }
  .annote-pop textarea, .annote-modal textarea { width: 100%; box-sizing: border-box; resize: vertical;
    min-height: 4rem; font: inherit; color: var(--ink); background: var(--bg); border: 1px solid var(--border);
    border-radius: .4rem; padding: .5rem; }
  .annote-row { display: flex; gap: .5rem; justify-content: flex-end; margin-top: .5rem; }
  .annote-btn { font-size: .8rem; font-weight: 600; padding: .35rem .7rem; border-radius: .4rem;
    border: 1px solid var(--border); background: var(--bg); color: var(--ink); cursor: pointer; }
  .annote-btn.primary { background: var(--accent); color: #fff; border-color: transparent; }
  .annote-btn:disabled { opacity: .5; cursor: default; }
  .annote-toggle { position: fixed; right: 1rem; bottom: 1rem; z-index: 55; font-size: .82rem; font-weight: 600;
    padding: .5rem .8rem; border-radius: 999px; background: var(--accent); color: #fff; border: none;
    cursor: pointer; box-shadow: 0 4px 14px rgba(0,0,0,.2); }
  .annote-panel { right: 1rem; bottom: 3.6rem; width: 20rem; max-width: calc(100vw - 2rem); max-height: 70vh;
    display: none; flex-direction: column; }
  .annote-panel.open { display: flex; }
  .annote-panel header { padding: .7rem .8rem; border-bottom: 1px solid var(--border); font-weight: 600; }
  .annote-list { overflow-y: auto; padding: .5rem; display: flex; flex-direction: column; gap: .5rem; }
  .annote-item { border: 1px solid var(--border); border-radius: .5rem; padding: .5rem .6rem; cursor: pointer; }
  .annote-item blockquote { margin: 0 0 .35rem; padding-left: .5rem; border-left: 2px solid var(--accent);
    color: var(--muted); font-size: .78rem; font-style: italic; }
  .annote-item .cmt { font-size: .85rem; }
  .annote-item .meta { display: flex; justify-content: space-between; align-items: center; margin-top: .35rem; }
  .annote-item .tag { font-size: .7rem; color: var(--muted); }
  .annote-item .actions { display: flex; gap: .6rem; }
  .annote-item .copy { font-size: .72rem; color: var(--accent); background: none; border: none; cursor: pointer; }
  .annote-item .del { font-size: .72rem; color: var(--bad); background: none; border: none; cursor: pointer; }
  .annote-panel footer { padding: .6rem .8rem; border-top: 1px solid var(--border); }
  .annote-hint { font-size: .72rem; color: var(--muted); margin: .4rem 0 0; }
  .annote-modal { top: 50%; left: 50%; transform: translate(-50%,-50%); width: 32rem;
    max-width: calc(100vw - 2rem); padding: 1rem; }
  .annote-backdrop { position: fixed; inset: 0; z-index: 59; background: rgba(0,0,0,.4); }
  .annote-toast { position: fixed; left: 50%; bottom: 1.5rem; transform: translateX(-50%); z-index: 70;
    background: var(--ink); color: var(--bg); font-size: .8rem; padding: .5rem .9rem; border-radius: .5rem;
    box-shadow: 0 4px 14px rgba(0,0,0,.25); }
  @media print { .annote-ui { display: none !important; } }
</style>
<script>
(function () {
  const main = document.querySelector('main');
  if (!main) return;

  const STORE_KEY = 'annote:' + location.pathname + ':' + (document.title || '');
  let notes = [];
  try { notes = JSON.parse(localStorage.getItem(STORE_KEY) || '[]'); } catch (e) {}

  const canHL = 'highlights' in CSS && typeof Highlight !== 'undefined';
  const hl = canHL ? new Highlight() : null;
  if (hl) CSS.highlights.set('annote-hl', hl);

  const save = () => { try { localStorage.setItem(STORE_KEY, JSON.stringify(notes)); } catch (e) {} };
  const nextId = () => 'a' + (notes.reduce((m, n) => Math.max(m, parseInt(n.id.slice(1), 10) || 0), 0) + 1);

  function anchorOf(node) {
    const el = node && (node.nodeType === 1 ? node : node.parentElement);
    const a = el && el.closest('[data-annote]');
    if (a) return a.getAttribute('data-annote');
    const sec = el && el.closest('section');
    const h = sec && sec.querySelector('h1,h2,h3');
    return h ? h.textContent.trim() : '';
  }

  function rootForAnchor(anchor) {
    if (!anchor) return main;
    try {
      const byData = main.querySelector('[data-annote="' + CSS.escape(anchor) + '"]');
      if (byData) return byData;
    } catch (e) {}
    for (const s of main.querySelectorAll('section')) {
      const h = s.querySelector('h1,h2,h3');
      if (h && h.textContent.trim() === anchor) return s;
    }
    return main;
  }

  function findRange(root, quote) {
    if (!root || !quote) return null;
    const w = document.createTreeWalker(root, NodeFilter.SHOW_TEXT);
    let text = '', segs = [];
    while (w.nextNode()) { segs.push({ n: w.currentNode, s: text.length }); text += w.currentNode.nodeValue; }
    const idx = text.indexOf(quote);
    if (idx < 0) return null;
    const end = idx + quote.length, r = document.createRange();
    let started = false;
    for (const seg of segs) {
      const segEnd = seg.s + seg.n.nodeValue.length;
      if (!started && idx < segEnd) { r.setStart(seg.n, idx - seg.s); started = true; }
      if (started && end <= segEnd) { r.setEnd(seg.n, end - seg.s); return r; }
    }
    return null;
  }

  function rebuild() {
    if (hl) hl.clear();
    for (const n of notes) {
      const r = findRange(rootForAnchor(n.anchor), n.quote);
      n._found = !!r;
      if (r && hl) hl.add(r);
    }
    renderPanel();
  }

  const ui = document.createElement('div');
  ui.className = 'annote-ui';
  document.body.appendChild(ui);

  const selBtn = document.createElement('button');
  selBtn.className = 'annote-selbtn';
  selBtn.textContent = '💬 commenter';
  selBtn.hidden = true;
  ui.appendChild(selBtn);
  selBtn.addEventListener('mousedown', (e) => e.preventDefault());

  let pending = null;
  const hideSel = () => { selBtn.hidden = true; pending = null; };

  function onMouseUp() {
    const sel = document.getSelection();
    if (!sel || sel.isCollapsed || !sel.rangeCount) return hideSel();
    const range = sel.getRangeAt(0), quote = sel.toString().trim();
    if (!quote || !main.contains(range.commonAncestorContainer)) return hideSel();
    const rect = range.getBoundingClientRect();
    pending = { quote, anchor: anchorOf(range.startContainer), rect: { bottom: rect.bottom, left: rect.left } };
    selBtn.style.top = (window.scrollY + rect.bottom + 6) + 'px';
    selBtn.style.left = (window.scrollX + rect.left) + 'px';
    selBtn.hidden = false;
  }
  document.addEventListener('mouseup', () => setTimeout(onMouseUp, 0));
  document.addEventListener('mousedown', (e) => { if (e.target !== selBtn) hideSel(); });
  selBtn.addEventListener('click', () => { if (pending) openEditor(pending, null); hideSel(); });

  function openEditor(data, existing) {
    closeEditor();
    const pop = document.createElement('div');
    pop.className = 'annote-pop';
    const ta = document.createElement('textarea');
    ta.placeholder = 'Ton commentaire…';
    ta.value = existing ? existing.comment : '';
    const row = document.createElement('div'); row.className = 'annote-row';
    const cancel = document.createElement('button'); cancel.className = 'annote-btn'; cancel.textContent = 'Annuler';
    const ok = document.createElement('button'); ok.className = 'annote-btn primary'; ok.textContent = 'Enregistrer';
    row.append(cancel, ok);
    pop.append(ta, row);
    ui.appendChild(pop);
    if (data && data.rect) {
      pop.style.top = Math.min(data.rect.bottom + 8, window.innerHeight - 200) + 'px';
      pop.style.left = Math.min(data.rect.left, window.innerWidth - 300) + 'px';
    } else {
      pop.style.top = '50%'; pop.style.left = '50%'; pop.style.transform = 'translate(-50%,-50%)';
    }
    ta.focus();
    cancel.addEventListener('click', closeEditor);
    ok.addEventListener('click', () => {
      const c = ta.value.trim();
      if (!c) return closeEditor();
      if (existing) existing.comment = c;
      else notes.push({ id: nextId(), quote: data.quote, anchor: data.anchor || '', comment: c });
      save(); rebuild(); closeEditor(); panel.classList.add('open');
    });
  }
  function closeEditor() { const p = ui.querySelector('.annote-pop'); if (p) p.remove(); }

  const toggle = document.createElement('button');
  toggle.className = 'annote-toggle';
  ui.appendChild(toggle);

  const panel = document.createElement('div');
  panel.className = 'annote-panel';
  panel.innerHTML =
    '<header>Retours</header><div class="annote-list"></div>' +
    '<footer><button class="annote-btn primary" style="width:100%">Copier les retours</button>' +
    '<p class="annote-hint">Colle le bloc copié dans planify pour appliquer tes retours.</p></footer>';
  ui.appendChild(panel);
  const listEl = panel.querySelector('.annote-list');
  const copyBtn = panel.querySelector('footer button');
  toggle.addEventListener('click', () => panel.classList.toggle('open'));
  copyBtn.addEventListener('click', copyFeedback);

  function renderPanel() {
    toggle.textContent = '💬 Retours (' + notes.length + ')';
    listEl.textContent = '';
    if (!notes.length) {
      const p = document.createElement('p'); p.className = 'annote-hint';
      p.textContent = 'Sélectionne du texte dans le rapport pour ajouter un retour.';
      listEl.appendChild(p);
    }
    notes.forEach((n, i) => {
      const item = document.createElement('div'); item.className = 'annote-item';
      const bq = document.createElement('blockquote'); bq.textContent = n.quote.replace(/\s+/g, ' ').trim();
      const cmt = document.createElement('div'); cmt.className = 'cmt'; cmt.textContent = n.comment;
      const meta = document.createElement('div'); meta.className = 'meta';
      const tag = document.createElement('span'); tag.className = 'tag';
      tag.textContent = 'Retour ' + (i + 1) + (n.anchor ? ' · ' + n.anchor : '') +
        (n._found === false ? ' · passage introuvable' : '');
      const actions = document.createElement('span'); actions.className = 'actions';
      const copy = document.createElement('button'); copy.className = 'copy'; copy.textContent = 'Copier';
      copy.addEventListener('click', (e) => { e.stopPropagation(); copyText(docMd([entryMd(n, i + 1)]), 'Retour copié'); });
      const del = document.createElement('button'); del.className = 'del'; del.textContent = 'Supprimer';
      del.addEventListener('click', (e) => { e.stopPropagation(); notes = notes.filter((x) => x !== n); save(); rebuild(); });
      actions.append(copy, del);
      meta.append(tag, actions);
      item.append(bq, cmt, meta);
      item.addEventListener('click', () => openEditor(null, n));
      listEl.appendChild(item);
    });
    copyBtn.disabled = !notes.length;
  }

  function entryMd(n, num) {
    return '## Retour ' + num + (n.anchor ? ' — ' + n.anchor : '') + '\n' +
      '> ' + n.quote.replace(/\s+/g, ' ').trim() + '\n' +
      'Commentaire : ' + n.comment.trim() + '\n';
  }
  function docMd(entries) {
    const h1 = main.querySelector('h1');
    const title = (h1 ? h1.textContent : (document.title || 'Rapport')).trim();
    const src = location.pathname.split('/').pop() || 'rapport.html';
    return '# Retours — ' + title + '\n_source : ' + src + '_\n\n' + entries.join('\n') + '\n';
  }
  async function copyText(md, okMsg) {
    try { await navigator.clipboard.writeText(md); toast(okMsg); }
    catch (e) { fallbackCopy(md); }
  }
  function copyFeedback() {
    if (!notes.length) return;
    copyText(docMd(notes.map((n, i) => entryMd(n, i + 1))), 'Retours copiés — colle-les dans planify');
  }

  function fallbackCopy(md) {
    const backdrop = document.createElement('div'); backdrop.className = 'annote-backdrop';
    const modal = document.createElement('div'); modal.className = 'annote-modal';
    const p = document.createElement('p'); p.style.margin = '0 0 .5rem'; p.style.fontWeight = '600';
    p.textContent = 'Copie ces retours (Ctrl/Cmd + C) puis colle-les dans planify :';
    const ta = document.createElement('textarea'); ta.readOnly = true; ta.style.minHeight = '12rem'; ta.value = md;
    const row = document.createElement('div'); row.className = 'annote-row';
    const close = document.createElement('button'); close.className = 'annote-btn'; close.textContent = 'Fermer';
    row.appendChild(close);
    modal.append(p, ta, row);
    ui.append(backdrop, modal);
    ta.focus(); ta.select();
    const dismiss = () => { backdrop.remove(); modal.remove(); };
    close.addEventListener('click', dismiss);
    backdrop.addEventListener('click', dismiss);
  }

  let toastT = null;
  function toast(msg) {
    let t = ui.querySelector('.annote-toast');
    if (!t) { t = document.createElement('div'); t.className = 'annote-toast'; ui.appendChild(t); }
    t.textContent = msg;
    clearTimeout(toastT);
    toastT = setTimeout(() => { if (t) t.remove(); }, 2600);
  }

  rebuild();
})();
</script>
```

## Rappels

- **Contenu réel** : aucun texte de remplissage, aucune section vide.
- **Débordement horizontal** : tout contenu large (code, diff, table, diagramme) scrolle dans son propre
  conteneur (`overflow-x:auto` est déjà sur `pre`) — le corps de page ne scrolle jamais latéralement.
- **Échapper** `<`, `>`, `&` dans les blocs de code et de diff.
