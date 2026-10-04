# Braconnage — installation serveur

Addon Cobblemon : cages, appâts et commanditaires. Un joueur accepte un contrat auprès d'un PNJ,
piège le Pokémon demandé dans une cage appâtée, le rapporte vivant et se fait payer.

## Important : ce mod va sur le serveur **et** chez tous les joueurs

Il ajoute des objets, des blocs et des entités, donc il doit être présent des deux côtés. Un joueur
qui ne l'a pas sera éjecté à la connexion avec une erreur de registre.

Ce n'est pas comme un mod purement client : il faut le distribuer à tout le monde, ou l'ajouter au
modpack.

## Installation

1. Déposer `braconnage-1.0.0.jar` dans le dossier `mods` du serveur.
2. Le distribuer aux joueurs, ou l'ajouter au modpack.
3. Redémarrer le serveur. Le fichier `config/braconnage/commanditaires.json` se crée tout seul au
   premier démarrage.

### Dépendances

Toutes déjà présentes sur le serveur Arverni.

| Dépendance | Version |
| --- | --- |
| Minecraft | 1.21.1 |
| Fabric Loader | 0.17.2 ou plus |
| Fabric API | — |
| Fabric Language Kotlin | 1.13.0 ou plus |
| Cobblemon | 1.7.0 ou plus |

## Mise en place

Poser un commanditaire à l'endroit voulu :

```
/braconnage pnj le_borgne
```

Il reste où il est posé et garde son orientation. Les joueurs lui parlent d'un clic droit.

## Commandes

| Commande | Qui | Effet |
| --- | --- | --- |
| `/braconnage moi` | tous | Sa réputation et son contrat en cours |
| `/braconnage pnj <profil>` | op | Pose un commanditaire |
| `/braconnage reputation <joueur> [valeur]` | op | Lit ou fixe la réputation |
| `/braconnage annuler <joueur>` | op | Efface le contrat en cours |
| `/braconnage recharger` | op | Relit la configuration sans redémarrer |

## Configuration

`config/braconnage/commanditaires.json`

### L'argent

Par défaut, le mod verse les Pokédollars directement via le composant `POKEDOLLARS` du mod Arverni,
par réflexion — aucune dépendance de compilation entre les deux mods.

Pour passer par une commande à la place, par exemple pour un autre système d'économie :

```json
"commande_argent": "eco give %joueur% %montant%"
```

### La réputation

Trois modes au choix.

**Rangée dans le monde** (par défaut) : le mod tient son propre compteur, indépendant de tout.

**Une faction dédiée au braconnage** :

```json
"reputation_arverni": "braconnage"
```

Le mod appelle alors `ReputationData` du mod Arverni avec ce nom. Il faut créer ce type de
réputation sur le serveur au préalable.

**Les points vont dans la faction du joueur** :

```json
"reputation_faction_actuelle": true,
"reputation_factions": ["faction1", "faction2", "faction3", "faction4"]
```

La faction retenue est celle où le joueur a le plus de points parmi la liste. Liste vide, le mod
demande à Arverni quelle est la faction du joueur via `getBestReputationName()`.

À savoir : dans ce mode, ce sont les points de faction qui débloquent les contrats rares. Un joueur
déjà bien noté se verra proposer les espèces rares sans avoir jamais braconné.

Si la classe d'Arverni est absente ou le nom inconnu, le mod le signale dans les logs et retombe sur
son compteur local sans rien perdre.

### Gagner et perdre selon les factions

Chaque action peut faire monter certaines factions et baisser d'autres. `nom` = le nom exact de la
réputation sur le serveur Arverni ; `points` positif = gain, négatif = perte.

```json
"reputations": {
  "braconniers": { "libelle": "Braconniers", "min": 0,    "max": 100 },
  "rangers":     { "libelle": "Rangers",     "min": -100, "max": 100 }
},
"reputation_sabotage":   [ { "nom": "braconniers", "points": -2 } ],
"reputation_liberation": [ { "nom": "rangers", "points": 3 }, { "nom": "braconniers", "points": -1 } ]
```

- `reputations` : déclare les factions (libellé affiché, bornes). Facultatif, mais sans ça une réputation ne descend pas sous 0.
- `reputation_sabotage` : ce que perd le braconnier dont la cage est coupée.
- `reputation_liberation` : ce que gagne (ou perd) le joueur qui coupe la cage et libère le Pokémon.
- Dans chaque commanditaire et chaque contrat, `"reputations": [...]` : ce que rapporte ou coûte une livraison.

Le fichier `commanditaires.json` est réécrit avec toutes les clés au démarrage et à chaque
`/braconnage recharger`, et `config/braconnage/exemple_reputations.json` montre un réglage complet à recopier.

### Les contrats

Chaque commanditaire a un profil, et chaque profil une liste de contrats :

```json
{
  "espece": "cobblemon:dratini",
  "rang_min": 3,
  "argent": 4000,
  "reputation": 3
}
```

`rang_min` est la réputation exigée pour que le contrat soit proposé. `reputation` est ce que la
livraison rapporte. On peut aussi donner des objets (`recompenses`) ou lancer des commandes
(`commandes`, avec `%joueur%`).

Le profil lui-même a un `rang_min` : en dessous, le commanditaire refuse de parler au joueur.

Après modification, `/braconnage recharger` suffit.

## Ce que les joueurs peuvent faire

- Une cage posée n'appartient qu'à celui qui l'a lancée : lui seul la ramasse, l'appâte et en sort
  le Pokémon.
- N'importe qui peut couper une cage trouvée avec une pince, et libérer le Pokémon. Le braconnier
  perd alors de la réputation (`reputation_sabotage`, 2 points par défaut) et son contrat si c'était
  sa proie ; celui qui a libéré le Pokémon gagne ce que donne `reputation_liberation`.
- Chaque espèce aime dix baies, toujours les mêmes sur le serveur, et chaque Pokémon sauvage en
  préfère une parmi ces dix. Le Carnet Pokémon liste les dix par espèce.
- Une cage sans la bonne baie fait fuir le gibier à dix blocs : se tromper d'appât vide le secteur.

## Données

Réputations et contrats sont enregistrés dans la sauvegarde du monde, sous `braconnage`. Rien n'est
écrit ailleurs.
