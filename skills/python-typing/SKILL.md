---
name: python-typing
description: >
  Bonnes pratiques de typing Python : annoter les signatures publiques, syntaxe moderne (`X | None`, generics
  PEP 695, alias `type`), rendre l'absence explicite et restreindre les types, préférer `Protocol` au duck typing non
  vérifié, fuir `Any`, et faire valider par un type-checker strict (pyrefly, ty, `mypy --strict` ou pyright). À consulter
  dès qu'on ajoute ou modifie des annotations de type en Python, qu'on conçoit une API/bibliothèque typée, qu'on
  définit des generics/protocols/TypedDict, qu'on configure pyrefly/ty/mypy/pyright, ou qu'on corrige des erreurs de
  type-checker — même si l'utilisateur ne dit pas explicitement « typing ».
argument-hint: "[fichier ou extrait à typer]"
allowed-tools: Read, Edit, Bash(pyrefly *), Bash(uvx pyrefly *), Bash(ty *), Bash(uvx ty *), Bash(mypy *), Bash(pyright *), Bash(uvx pyright *)
---

# python-typing

Le `typage` de variables, paramètres et valeurs de retour est essentiel. Il permet :
- de détecter des bugs avant l'exécution
- de documenter le code
- de faciliter la compréhension du code pour les LLM
- d'avoir une meilleure auto-complétion dans les IDE

Cible : Python **3.12+**

Type-checkers (choisir en fonction de la configuration du projet, et des outils installés) :

| Outil                  | +                                     | −                          |
|------------------------|---------------------------------------|----------------------------|
| `pyrefly` (Meta, Rust) | très rapide, moderne                  | jeune, écosystème en cours |
| `ty` (Astral, Rust)    | très rapide, intégré à `uv`/`ruff`    | encore en preview          |
| `mypy --strict`        | mature, très répandu, la référence    | plus lent                  |
| `pyright` (Microsoft)  | rapide, excellent narrowing (Pylance) | dépend de Node             |

## Lexique

Les termes techniques employés plus bas, une fois pour toutes :

- **Annotation de type** — le type écrit dans le code (`age: int`, `-> User`) - terme officiel (PEP 526).
- **Type-checker** — l'outil qui contrôle la cohérence des annotations (pyrefly, mypy, ...).
- **Inférence** — le type que le type-checker déduit seul, sans annotation explicite.
- **Generic** — un paramètre de type (`T`) qui transporte le type d'entrée jusqu'à la sortie.
- **`Protocol`** — un type défini par sa forme (ses méthodes), sans héritage : du *duck typing* vérifié.
- **Duck typing** — accepter un objet pour ce qu'il sait faire, pas pour sa classe.
- **`Any`** — le type qui désactive la vérification là où il apparaît.
- **`cast`** — permet d'indiquer le type au `type-checker`.

## Règles

### 1. Typer toutes les signatures publiques

- **Il faut** typer toutes les fonctions, méthodes et attributs de classes : les paramètres, la valeur de retour et
les attributs.
- **Si cela est nécessaire**, il faut typer le code à l'intérieur de fonctions ou méthodes lorsque cela apporte de la
lisibilité (par exemple pour documenter un retour de fonction pas très explicite).

```python
def get_user(user_id: str) -> User | None:
    """Renvoie l'utilisateur s'il existe, None sinon."""
    ...

def do_something(v: str) -> None:
    """
    Le type de retour de process_value() n'est pas explicite, le nom de la fonction ne donne pas
    d'information claire, on peut donc typer le résultat pour améliorer la lisibilité.
    """
    result: int = process_value(v)
    ...

class UserRepository:
    user: User | None

    def __init__(self, db: Database) -> None:  # -> None explicite sur __init__
        self._db = db

    async def save(self, user: User) -> User: ...
```


### 2. Syntaxe moderne plutôt que les anciens alias `typing`

**Il faut** préférer la syntaxe moderne plutôt que les anciens alias `typing` (`Optional`, `Union`, `List`…)
L'ancienne syntaxe ne doit être utilisée que pour la rétrocompatibilité avec d'anciennes versions de Python.

| À éviter                        | Écrire                      |
|---------------------------------|-----------------------------|
| `Optional[X]`                   | `X \| None`                 |
| `Union[A, B]`                   | `A \| B`                    |
| `List[X]`, `Dict[K, V]`         | `list[X]`, `dict[K, V]`     |
| `Tuple[int, ...]`, `Set[X]`     | `tuple[int, ...]`, `set[X]` |
| `from typing import List, Dict` | (aucun import)              |

### 3. Éviter `Any` et lui préférer un type précis

`Any` désactive la vérification là où il apparaît.
Lorsque c'est possible, remplacer `Any` par un type plus précis :

- **Donnée vraiment inconnue** → `object` : légal, mais force à restreindre le type avant usage.
- **Conteneur ou fonction réutilisable** → un generic (règle 5), qui préserve le type.
- **Sortie d'une lib non typée** → `cast(TypeVoulu, valeur)` au point d'entrée, puis un type propre en aval.
- **« Quelque chose qui a telle méthode »** → un `Protocol` (règle 6).

`cast()` affirme un type sans le contrôler : le réserver à une valeur réellement connue, jamais pour faire
taire une erreur légitime — dans ce cas, corriger le type réel.

Si tu dois supprimer une erreur ponctuelle, cibler le code : `# type: ignore[return-value]`, jamais un
`# type: ignore` nu qui masque aussi les futures erreurs de la ligne.

### 4. Rendre l'absence explicite

L'absence fait partie du type : l'annoter avec `X | None` plutôt qu'une sentinelle implicite ou un `None` non
typé. Le type-checker impose alors de **restreindre** (`is None`, `assert`) avant d'utiliser la valeur.

```python
def find(user_id: str) -> User | None:  # l'absence est dans la signature
    ...


u = find("42")
u.name                    # rejeté : u peut être None
if u is not None:
    u.name                # ok : restreint à User
```

### 5. Generics : préserver l'information de type

Un generic permet d'avoir un typage générique, fonctionnant avec plusieurs types différents. Syntaxe PEP 695 (3.12+).
C'est généralement pour faire suivre le/les types d'entrée d'une fonction/méthode jusqu'au type de retour.

```python
def first[T](items: list[T]) -> T:
    # first([1, 2]) -> int
    # first(["a"]) -> str
    return items[0]


class Box[T]:
    def __init__(self, value: T) -> None:
        self._value = value

    def get(self) -> T:
        return self._value
```


### 6. `Protocol` : typage structurel plutôt qu'héritage

Un `Protocol` décrit la forme attendue (ses méthodes) sans imposer d'héritage.
À préférer quand tu ne possèdes pas la classe concrète, ou que seule une méthode t'intéresse.

```python
from typing import Protocol


class Readable(Protocol):
    def read(self) -> bytes: ...


def consume(source: Readable) -> bytes:  # accepte fichier, socket, BytesIO… sans lien d'héritage
    return source.read()
```

Réserve l'**ABC** (héritage explicite) au cas où tu possèdes la hiérarchie et veux partager de l'implémentation.

### 7. La bonne structure de données

Choisis le conteneur selon la nature de la donnée plutôt que d'empiler des `dict[str, Any]` :

- **`TypedDict`** — un dict à clés fixes connues (payload JSON, réponse d'API).
- **`dataclass`** — un objet avec valeurs par défaut, comportement ou invariants.
- **`NamedTuple`** — un tuple nommé, immuable et léger.
- **`Literal` / `Enum`** — un ensemble **fermé** de valeurs (`Literal["asc", "desc"]`) : le type-checker rejette tout
  ce qui sort de l'ensemble, ce qu'un `str` ne fait pas.

### 8. Nommer et verrouiller

- **Alias** pour un type complexe et récurrent :
  `type Json = dict[str, "Json"] | list["Json"] | str | int | float | bool | None`
  (PEP 695, 3.12+). Un nom parlant vaut mieux qu'une union répétée partout.
- **`Final`** pour une constante non réassignable
- **`ClassVar`** pour un attribut de classe (vs d'instance).
- **`Self`** (3.11+) pour une méthode qui renvoie son propre type (`-> Self`), y compris sur les sous-classes.
- **`@override`** (3.12+, ou `typing_extensions`) sur une méthode censée en redéfinir une autre : la CI casse si le
  parent renomme ou supprime la méthode.

## Signaux d'alerte — STOP

- Sur le point d'écrire `Any` → est-ce `object`, un generic, un `Protocol` ou un `cast` ciblé ?
- `# type: ignore` sans code → ajouter le `[code]`, sinon on masque les futures erreurs.
- `Optional` / `List` / `Dict` importés de `typing` → moderniser (`X | None`, `list`, `dict`).
- `cast()` pour éteindre une erreur qui a raison → corriger le type, pas le type-checker.
- Annotations soignées mais aucun type-checker strict en CI → le typing n'engage rien.
