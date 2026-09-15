# Braconnage — mod Cobblemon pour le serveur Arverni

Un PNJ commanditaire donne un contrat sur une espèce précise. Le joueur pose une cage, l'appâte avec
la bonne baie, attend que le Pokémon y entre, et rapporte la cage pleine pour se faire payer.

---

## 1. Installation

**Ce mod va sur le serveur ET chez tous les joueurs.**

Il ajoute des objets, des blocs et des entités. Un joueur qui ne l'a pas sera éjecté à la connexion
avec une erreur d'entrée de registre inconnue. Il faut donc :

1. Déposer `braconnage-1.0.0.jar` dans le dossier `mods` du serveur.
2. L'ajouter au modpack pour que les joueurs l'aient aussi.
3. Redémarrer le serveur.

Au premier démarrage, le mod crée `config/braconnage/commanditaires.json`. Le fichier fourni dans
`config-exemple/` peut être copié à cet emplacement tel quel.

### Dépendances

Toutes déjà présentes sur Arverni.

| Dépendance | Version |
| --- | --- |
| Minecraft | 1.21.1 |
| Fabric Loader | 0.17.2 ou plus |
| Fabric API | — |
| Fabric Language Kotlin | 1.13.0 ou plus |
| Cobblemon | 1.7.0 ou plus |

---

## 2. Poser un commanditaire

```
/braconnage pnj le_borgne
```

Le PNJ apparaît à vos pieds, garde sa position et son orientation. Les joueurs lui parlent d'un clic
droit. `le_borgne` est l'identifiant du profil dans la configuration ; l'autocomplétion propose ceux
qui existent.

---

## 3. Commandes

| Commande | Qui | Effet |
| --- | --- | --- |
| `/braconnage moi` | tous | Sa réputation et son contrat en cours |
| `/braconnage pnj <profil>` | op | Pose un commanditaire |
| `/braconnage reputation <joueur> [valeur]` | op | Lit ou fixe la réputation |
| `/braconnage annuler <joueur>` | op | Efface le contrat en cours |
| `/braconnage recharger` | op | Relit la configuration sans redémarrer |

---

## 4. Configuration

Tout se passe dans `config/braconnage/commanditaires.json`. Après modification,
`/braconnage recharger` suffit, pas besoin de redémarrage.

### 4.1 L'argent

Par défaut, le mod verse les Pokédollars directement via le composant `POKEDOLLARS` du mod Arverni.
L'appel se fait par réflexion : aucune dépendance de compilation entre les deux mods, et le
braconnage fonctionne même sans Arverni.

Pour passer par une commande à la place, par exemple pour un autre système d'économie :

```json
"commande_argent": "eco give %joueur% %montant%"
```

`%joueur%` et `%montant%` sont remplacés à l'exécution. Laisser vide pour utiliser le composant.

### 4.2 La réputation — trois modes

**Mode 1 : compteur interne** (par défaut)

```json
"reputation_arverni": "",
"reputation_faction_actuelle": false
```

Le mod tient son propre compteur dans la sauvegarde du monde, indépendant de tout le reste.

**Mode 2 : une faction dédiée au braconnage**

```json
"reputation_arverni": "braconnage",
"reputation_faction_actuelle": false
```

Le mod appelle `ReputationData` du mod Arverni avec ce nom. **Il faut créer ce type de réputation sur
le serveur au préalable.** C'est le mode recommandé : la progression du métier reste séparée des
factions existantes.

**Mode 3 : les points vont dans la faction du joueur**

```json
"reputation_faction_actuelle": true,
"reputation_factions": ["faction1", "faction2", "faction3", "faction4"]
```

Remplacer par les quatre vrais identifiants de factions. La faction retenue est celle où le joueur a
le plus de points parmi la liste. Si la liste est vide, le mod demande à Arverni quelle est la
faction du joueur via `getBestReputationName()`.

> **À savoir avant de choisir ce mode.** La réputation sert aussi à débloquer les contrats : un
> joueur déjà bien noté dans sa faction se verra proposer les espèces rares sans avoir jamais
> braconné. Si vous voulez une vraie progression du métier, préférez le mode 2.

Dans tous les cas, si la classe d'Arverni est absente ou le nom de réputation inconnu, le mod le
signale dans les logs et retombe sur son compteur interne sans rien perdre.

### 4.3 Les commanditaires et leurs contrats

```json
"commanditaires": {
  "le_borgne": {
    "nom": "Le Borgne",
    "skin": "le_borgne",
    "bras_fins": false,
    "rang_min": 0,
    "contrats": [
      {
        "espece": "cobblemon:dratini",
        "rang_min": 3,
        "argent": 4000,
        "recompenses": [],
        "commandes": [],
        "reputation": 3
      }
    ]
  }
}
```

**Sur le profil**

| Champ | Effet |
| --- | --- |
| `nom` | Nom affiché dans le chat et le menu |
| `skin` | Fichier dans `assets/braconnage/textures/entity/commanditaire/`, sans l'extension |
| `bras_fins` | `true` pour un skin à bras fins |
| `rang_min` | En dessous de cette réputation, le PNJ refuse de parler au joueur |

**Sur un contrat**

| Champ | Effet |
| --- | --- |
| `espece` | Identifiant Cobblemon, par exemple `cobblemon:pikachu` |
| `rang_min` | Réputation exigée pour que ce contrat soit proposé |
| `argent` | Somme versée à la livraison |
| `reputation` | Points gagnés à la livraison |
| `recompenses` | Objets donnés en plus, `[{ "item": "minecraft:emerald", "nombre": 4 }]` |
| `commandes` | Commandes serveur lancées à la livraison, avec `%joueur%` |

Un contrat est tiré au hasard parmi ceux que la réputation du joueur autorise. La même offre revient
tant qu'il ne l'a pas refusée.

---

## 5. Comment ça marche en jeu

- **Une cage appartient à celui qui l'a lancée.** Lui seul la ramasse, l'appâte et en sort le
  Pokémon. Les autres ne peuvent rien en faire.
- **N'importe qui peut la couper à la pince** et libérer le Pokémon. Le braconnier perd alors
  2 points de réputation, et son contrat si c'était sa proie.
- **Chaque espèce aime dix baies**, toujours les mêmes sur tout le serveur, et chaque Pokémon sauvage
  en préfère une seule parmi ces dix. Le Carnet Pokémon liste les dix par espèce ; trouver la bonne
  est le travail du joueur.
- **Une cage sans la bonne baie fait fuir le gibier à dix blocs.** Se tromper d'appât vide le
  secteur : le placement et le choix de la baie sont le cœur du métier.
- **Trois tailles de cage**, choisies automatiquement selon la taille de l'espèce demandée. La petite
  se porte d'une main, les deux autres à deux mains, et le Pokémon reste visible dans la cage.

---

## 6. Où sont les données

Réputations et contrats en cours sont enregistrés dans la sauvegarde du monde, sous le nom
`braconnage`. Rien n'est écrit en dehors du dossier du monde et de `config/braconnage/`.
