# Protocole robot ↔ interface gouverneur (ColonyAFIS)

C'est le **contrat** entre le programme du robot et l'interface
`gouverneur.html`. Tant que ces messages restent identiques, on peut
modifier tout le reste du programme (réglages, déplacements, stratégie)
sans toucher à l'interface. Dans le `.ino`, c'est la **PARTIE B**.

Liaison : module Bluetooth de la MegaPi sur `Serial3` (115200 bauds).
Les mêmes messages sortent aussi sur le câble USB (115200 bauds).

## Interface → robot

**Configuration**, avant la manche, une ligne par paramètre :

| Ligne | Sens |
|---|---|
| `#D=8` | durée de la manche en minutes (6 à 12) |
| `#S=80` / `#S=?` | stabilité initiale / inconnue |
| `#G=WE?` | type des gisements gauche, milieu, droit (W eau, E énergie, M matériaux, ? inconnu) |
| `#O=3` | dégradation par l'adversaire, en points par minute |
| `#C=1` | aller lire la carte d'état (1) ou non (0) |
| `#L=8` | port du suiveur de ligne (0 = désactivé, 5 ou 8) |

Le robot répond `CFG;D=8` à chaque ligne.

**Commandes** (une lettre) :

| Lettre | Pendant une manche | Hors manche |
|---|---|---|
| `q` | arrêt d'urgence → pause | arrêt |
| `r` | reprise sur place | – |
| `h` / `j` | replacé au départ sans / avec cube | – |
| `f` | arrêter la manche | – |
| `i` | état (`STATUT`) | état + position |
| `s` / `t` | – | lancer la manche / l'auto-test |
| `z u d x` | – | bras : origine, monter, descendre, stop |
| `o c` | – | pince : ouvrir, fermer |
| `w b 3 4 0` | – | ±20 cm, ±90°, position = départ |
| `1 2` | – | calibration 50 cm, calibration 360° |
| `g v n k` | – | sonder gisements, sonar, couleur, suiveur de ligne |
| `p e ?` | – | test saisie, test dépose, aide |

## Robot → interface

Format : `TAG;champ;champ…`, une ligne par message.

| Message | Quand |
|---|---|
| `MODE;PRET` / `MODE;MANCHE STANDARD` / `MODE;AUTO-TEST` | démarrage |
| `MODE;PAUSE;raison` / `MODE;REPRISE` / `MODE;REPRISE APRES REPLACEMENT` | arrêt d'urgence |
| `MODE;FIN MANCHE` / `MODE;FIN AUTO-TEST` / `MODE;MANCHE INTERROMPUE` / `MODE;ARRET` | fin |
| `TEL;t=;x=;y=;th=;S=;etat=;cube=;score=` | chaque seconde pendant la manche |
| `CIBLE;ZONE;gisement=G1;type=EAU` | zone d'infrastructure visée |
| `RES;EAU` puis `PICK;EAU;G1` | ressource identifiée et récupérée |
| `DEPOT;EAU;HABITAT` | dépôt |
| `INFRA;HABITAT;total=1` | infrastructure terminée |
| `INV;HABITAT=1/0/0 construits:0;FERME=…` | ressources posées par zone (eau/énergie/matériaux) |
| `ETAT;ALERTE;S~62;source=carte` | état du territoire (estimé ou lu sur la carte) |
| `SCORE;estime=…;etat=…`, `FIN;…` | fin |
| `ALERTE;texte` | problème (bras bloqué, sortie de terrain…) |
| `STATUT;mode=0/1;pause=0/1` | réponse à `i` |
| `MSG;texte` | information libre (affichée dans le journal) |

Ressources : `EAU`, `ENERGIE`, `MATERIAU`, `INCONNU`, `AUCUN`.
Zones : `HABITAT`, `FERME`, `INDUSTRIE`, `RECYCLAGE`.
États : `STABLE`, `ALERTE`, `CRISE`, `CRITIQUE`, `EFFONDREMENT`.

**Règle :** pour ajouter une information, utiliser `MSG;…` ou un
nouveau tag. Ne jamais renommer ni réordonner les champs existants.
