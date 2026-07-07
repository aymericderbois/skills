---
name: explain-your-changes
description: Explique, fichier par fichier, les changements de la session — ce qui a changé et, si nécessaire, pourquoi. Utiliser quand l'utilisateur demande un récapitulatif ou une explication des modifications.
---

Pour chaque fichier modifié dans la session, montre le diff puis explique **le changement, et si nécessaire pourquoi**. N'ajoute le *pourquoi* que lorsqu'il ne se déduit pas du diff — pas de justification pour un import ou un renommage évident.

Couvre tous les fichiers modifiés : aucun changement ne reste sans explication.

<output-template>
**`blog/constants.py:5`**
```diff
+WORDS_PER_MINUTE = 200
```

Vitesse de lecture moyenne servant à estimer le temps de lecture. Évite la valeur magique dans le calcul.

---

**`blog/models.py:24`**
```diff
+    def reading_time(self) -> int:
+        return max(1, round(len(self.body.split()) / WORDS_PER_MINUTE))
```

Nouvelle méthode sur le modèle. Estime le temps de lecture en minutes à partir du nombre de mots, avec un plancher à 1 minute pour les articles très courts.

---

**`templates/blog/detail.html:8`**
```diff
 <link rel="stylesheet" href="/static/css/article.css">
+<link rel="stylesheet" href="/static/css/meta.css">
```

Charge la feuille de style de la ligne de métadonnées (auteur, date, temps de lecture).

---

**`templates/blog/detail.html:21`**
```diff
-    <div class="meta">{{ post.published_at|date:"d/m/Y" }}</div>
+    <div class="meta">
+      {{ post.published_at|date:"d/m/Y" }} · {{ post.reading_time }} min de lecture
+    </div>
```

Ajoute le temps de lecture à côté de la date de publication. La valeur vient de la méthode `reading_time` du modèle.
</output-template>
