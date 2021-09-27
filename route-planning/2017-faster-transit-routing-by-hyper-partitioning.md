# (ARTICLE) Faster Transit Routing by Hyper Partitioning

- **url** = [PDF](https://drops.dagstuhl.de/opus/volltexte/2017/7896/pdf/OASIcs-ATMOS-2017-8.pdf) (md5sum=`b9fd9973a942e4ce5c81057b9af2605e`), [copie locale](LOCALCOPIES/OASIcs-ATMOS-2017-8.pdf)
- **source** = [ATMOS 2017](https://algo2017.ac.tuwien.ac.at/atmos/)
- **auteurs** = Daniel Delling, Julian Dibbelt, Thomas Pajor, and Tobias Zündorf
- **date de publication** = 2017
- **date de rédaction initiale de ces notes** = 2 novembre 2020

**TL;DR** :
- HypRAPTOR = technique pour accélérer RAPTOR
- représentation du PTN sous forme d'hypergraphe (chaque vertex est une route, chaque hyperedge représente un stop, et regroupe les routes qui contiennent le stop)
- partitionnement de l'hypergraphe en k cellules à peu près équilibrées, en essayant de minimiser le nombre de cut-edges
- calcul d'un fill-in contenant toutes les routes et stops nécessaires pour faire un chemin optimal, quelles que soient la cellule source et target
- speedup car query modifiée : au lieu de scanner toutes les routes/stops, on se limite à :
    - la cellule source
    - la cellule target
    - la cellule fill-in
- le speedup est modeste (~2), mais est appliqué sur des PTN assez grands (netherlands / suisse)
- par rapport à RAPTOR, on trade l'ajout de preprocessing (entre 30m et 3h30) contre une petite acélération -> pas sûr que le jeu en vaille la chandelle, surtout face aux concurrents.
- attention : l'article suppose qu'on part/arrive sur un stop source/target -> pas sûr que ça se marie bien avec l'unrestricted walking


* [(ARTICLE) Faster Transit Routing by Hyper Partitioning](#article-faster-transit-routing-by-hyper-partitioning)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = Preliminaries](#section-2--preliminaries)
   * [Section 3 = Partitioning Public Transit Networks](#section-3--partitioning-public-transit-networks)
   * [Section 4 = Main Algorithm](#section-4--main-algorithm)
      * [4.1.2 = partitioning](#412--partitioning)
      * [4.2 = fill-in computation](#42--fill-in-computation)
      * [4.3 = queries](#43--queries)
   * [Section 5 = experiments](#section-5--experiments)

## Section 1 = introduction

État de l'art du route planning sur PTN = 5 techniques :
- CSA (peu de preprocess)
- RAPTOR (peu de preprocess, orienté multi-critères)
- Trip-Based Public Transit Routing (10x plus rapide que CSA/RAPTOR pour un préprocessing léger)
- Transfer Patterns (très très rapide, mais nécessite un énorme préprocessing + manque de robustesse aux retards)
- Labelling techniques (le plus rapide, mais préprocessing prohibitif)

## Section 2 = Preliminaries

RAS

## Section 3 = Partitioning Public Transit Networks

Approche classique = partitionner les stops en k cellules (si possible, équilibrées), en minimisant le nombre de cut-edges, mais ça marche pas bien pour RAPTOR, car ça splitte les routes  (qui sont les éléments de base de RAPTOR).

Approche proposée = partitionner les routes plutôt que les stops.

Au préalable, on construit un hypergraphe du PTN : chaque vertex de l'hypergraphe est une route (ou un footpath), chaque hyperedge (i.e. groupe de routes) est un stop partagé par les différentes routes/footpaths que sont les vertices de l'hyperedge.

Puis, on partitionne cet hypergraphe en cellules ( _cells_), les cuts de cet hypergraphe sont des hyperedges = des cut-stops.

## Section 4 = Main Algorithm

### 4.1.2 = partitioning

Deux algos de partioning proposés : greedy et utilisation de [h-metis](http://glaros.dtc.umn.edu/gkhome/metis/hmetis/overview) en blackbox.

Objectif = trouver des cellules (=regroupements) de vertex (i.e. de routes) et hyperedges (i.e. stops), en essayant de minimiser les cut-edges.

Attribuer des poids aux aux vertices (donc aux routes) ou aux hyperedges (donc aux stops) permet d'influer sur le partitioning. 

### 4.2 = fill-in computation

On ne peut pas limiter RAPTOR aux cellules auxquelles appartiennent les stops source/target, car en pratique, il se peut qu'un chemin optimal emprunte une route qui soit en dehors de ces cellules -> on calcule un set de routes **fill-in**, tel que tout chemin optimal n'empruntera que :
- la cellule source
- la cellule target
- la cellule fill-in

Calcul naïf du fill-in = faire une range-query pour chaque cut-stop, mais trop lourd.

Simplification = range-query pour chaque cut-stop, _limité_ aux cellules voisines du stop (i.e. aux routes à laquelle appartient le stop). En pratique, ça ne calcule pas forcément la route optimale, du coup ces routes non-optimales peuvent se retrouver dans le fill-in -> n'empêche pas l'algo d'être exact, mais dégrade un peu ses perfs.

Papier à analyser plus en profondeur, mais au final, on calclue le fill-in, permettant de limiter RAPTOR à la cellule source, la cellule target, et le fill-in.

### 4.3 = queries

Variante de RAPTOR n'utilisant que les routes de :
- la cellule source
- la cellule target
- la cellule fill-in

Note : chaque cell contient à la fois des vertices (=routes) et des hyperedges (=stops).

Plusieurs implémentations :
- on flagge (bool) chaque stop/route s'il est sur le fill-in -> à la query, on peut ignorer les routes/events/stops qui ne sont ni sur la même cellule que source ou target, ni flaggées. Pas hyper-efficace, car il faut tout de même scanner les routes/events/stops pour savoir s'ils sont flaggés
- représenter les routes/stops/events comme une skip-list permettant de ne pas scanner ce qui est à ignorer : plus rapide, mais pas très cache-friendly
- (le plus rapide, mais "gâche" de la mémoire) copier dans une structure à part les trips contenant au moins un stop flaggé, pour itérer dessus de façon efficace.

Cas particulier à gérer si le stop source/target est sur un cut-edge.

Ils proposent une règle pour contourner le besoin de transitive closure sur les footpaths.

## Section 5 = experiments

L'article donne deux sources de données, qui mixent tous deux des données locales (métros) et longue-distance (trains) :

- NT = [Timetables for transit in the Netherlands](https://old.datahub.io/dataset/gtfs-nl) : 54k stops, 618k trips, 13k routes, 13M events.
- CH = [Public Transporation Feed for Switzerland](https://gtfs.geops.ch/) : 25k stops, 1076k trips, 16k routes, 12M events.

Temps de preprocess (partitioninig + calcul fill-in) = ~3h30m sur NT, 36m sur CH, très parallélisable.

Trade-off à trouver : si on découpe en trop de cellules, le fill-in devient trop gros. 8 cellules semble un bon compromis pour leur cas, mais **la valeur pourra être différente pour d'autres topologies**.

Calcul du fill-in = on lance un rRaptor par cut-stops.

Au final, le speedup obtenu est pas faramineux : query environ divisée par deux (entre 1.7 et 2.1) sur la variante la plus rapide (=cache-friendly) d'HypRAPTOR.

Note : borne supérieure du speedup attendu en utilisant k-cells est ~k/2
