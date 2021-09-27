# (ARTICLE) Connection Scan Algorithm

- **url** = [PDF](https://arxiv.org/pdf/1703.05997.pdf) (md5sum=`be27c82b32bd880b5835e1949421e7e1`), [copie locale](LOCALCOPIES/1703.05997.pdf)
- **source** = Le papier est le merge de 3 articles : [l'article original sur CSA](https://link.springer.com/chapter/10.1007/978-3-642-38527-8_6) a été publié à [SEA 2013](http://sea2013.dis.uniroma1.it/), [l'article original sur CSAccel](https://epubs.siam.org/doi/abs/10.1137/1.9781611973198.12) a été publié à [ALENEX 2014](https://epubs.siam.org/doi/book/10.1137/1.9781611973198), [l'article original sur MEAT](https://drops.dagstuhl.de/opus/frontdoor.php?source_opus=4748) a été publié à [ATMOS 2014](https://drops.dagstuhl.de/portals/oasics/index.php?semnr=14010).
- **auteurs** = Julian Dibbelt, Thomas Pajor, Ben Strasser, and Dorothea Wagner
- **date de publication** = le papier date de 2017, mais c'est un merge de 3 articles : CSA qui date de 2013, CSAccel et MEAT qui datent de 2014. 
- **date de rédaction initiale de ces notes** = 3 novembre 2020

**TL;DR** :
- Le papier est un merge de 3 articles, je m'intéresse surtout à CSA.
- Les données sont représentées comme un grand tableau de connections élémentaires, triées par departure_time (le tout saupoudré d'un peu de footpaths).
- Derrière, l'algo est rapide car 1. il suffit d'itérer une fois sur le tableau, et 2. des optimisations permettent de limiter la quantité de connections scannées.
- Comme il n'y a presque pas de preprocessing (il suffit de trier les connections), on peut prendre en compte les délais si besoin.
- En terme de perfs, il est annoncé comme pas mal meilleur que RAPTOR.
- En revanche, dans sa version de base, il ne semble pas permettre le multi-critère, et notamment ne semble pas optimiser le nombre de transfert.
- Détail intéressant : il illustre une catégorie de problèmes liée à un chemin optimal mais absurde.
- **CSAccel** : je n'ai fait que survoler ; gain de x10 sur les range-queries, mais x10 aussi sur le preprocsesing + algo plus compliqué que CSA, et nécessite de partitionner le graphe. D'autres références ultérieures avaient l'air de dire que CSAccel se concentrait sur l'EAT en négligeant l'optimisation du nombre de transferts.
- **MEAT** : je n'ai pas regardé MEAT en détail, mais ça a l'air intéressant : ça permet de renvoyer des morceaux d'itinéraires "backup" (ainsi, on sait quoi faire au cas où on loupe un changement)


* [(ARTICLE) Connection Scan Algorithm](#article-connection-scan-algorithm)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = preliminaries](#section-2--preliminaries)
   * [Section 3 = Earliest Arrival Conection Scan](#section-3--earliest-arrival-conection-scan)
   * [Section 4 = profile connection scan](#section-4--profile-connection-scan)
   * [Section 5 = connection scan accelerated](#section-5--connection-scan-accelerated)
   * [Section 6 = Minimum Expected Arrival Time](#section-6--minimum-expected-arrival-time)

## Section 1 = introduction

Se passer de la priority-queue du Dijkstra (c'est là où Dijkstra passe le plus clair de son temps), et utiliser un array à la place.

## Section 2 = preliminaries

Une connection = tuple de 5 éléments :
- departure_stop + departure_time
- arrival_stop + arrival_time
- trip

Dans ce modèle, Chaque connection relie deux stops **consécutifs**.

Footpath = triplet :
- departure_stop
- arrival_stop
- duration

Dans ce modèle, le graphe des footpaths doit être transitively closed (i.e. si AB et BC sont dans le graphe, alors on a obligatoirement AC dans le graphe) et respectant l'inégalité triangulaire (si AB, BC et AC sont dans le graphe, on a AC ≤ AB+BC).

Inconvénient de la transitive closure = chaque composante connexe doit former une clique, ce qui est très lourd.

Modèle alternatif = limiter les footpaths à une certaine durée de marche. En pratique ce modèle n'est pas utilisé car il pose des problèmes concrets.

Journey = alternance de footpaths et de connections.

À noter que chaque stop a au moins un footpath = le footpath "loop" (qui relie la connection actuelle à la suivante du même trip, et indique qu'on reste dans le même mode de transport).

Ils soulèvent une problématique intéressante = un chemin peut-être "optimal tout en étant absurde, leur exemple à la section 2.3 est très illustratif.

Autre problème (indiquant un iti alternatif) = range-problem = renvoyer l'iti optimal + tous ceux qui ont un temps de trajet moins de deux fois celui de l'iti optimal.

## Section 3 = Earliest Arrival Conection Scan

L'algo est agréablement simple et très bien décrit par les pseudo-codes des figures 3 et 4.

On représente tout le GTFS en entrée par un grand array des connections (5-uple décrit plus haut), classés par ordre de departure_time croissantes.

De la même façon qu'au cours de l'algo, Dijkstra fait évoluer un tentative_cost pour chaque vertex, au cours de l'algo, CSA fait évoluer un tentative_arrival_time pour chaque stop.

CSA = scanner une seule fois le grand array, et mettre à jour à chaque tour de boucle les arrival_time des stops qu'on peut rejoindre depuis l'arrival_stop de la connection en cours d'itération.

T est un array qui associe à chaque trip un flag disant si oui ou non on a atteint le trip.

S est un array qui associe à chaque stop l'actuel tentative_arrival_time.

Une connection est accessible si :
- son trip a été flaggé comme accessible
- on est capable d'arriver au stop de départ avant l'heure à laquelle la connection part (i.e. on peut "attrapper" la connection)

Dans ce cas, on flagge le trip de la connection comme accessible (si pas encore le cas), et on met à jour la tentative_arrival_time de tous les stops qu'on peut rejoindre à pieds depuis le stop d'arrivée (ce qui inclut le stop d'arrivée de la connection courante).

Optimisations : le papier propose 3 optimisations, qui semblent essentielles pour les bonnes performances de CSA :
- **starting criterion** = on peut commencer l'algo non pas sur la première connection du tableau, mais sur la première connection qui part après l'heure de départ requis par l'utilisateur (en la trouvant par binary search)
- **stopping criterion** = on peut arrêter l'algo (i.e. arrêter d'itérer sur les connections `c`) lorsque la connection courante `c` part après la tentative_arrival_time du stop target `S[t]` : en effet, quoi qu'il arrive, cette connexion (et a fortiori les suivantes) arriveront en `t` **après** `S[t]`, et n'amélioreront donc plus l'EAT en t.
- **limited walking** = critère un peu similaire au stopping criterion, mais sur l'heure d'arrivée : si la tentative_arrival_time de `S[t]` est déjà meilleure que l'heure d'arrivée à l'arrival_stop de la connection, ça veut dire que la connection arrive **après** notre meilleur score pour `t`, et il est donc inutile d'itérer sur les footpaths sortants de `c` car ils ne peuvent pas améliorer `S[t]`. Ça ne veut pas dire qu'il faut arrêter l'algo, though, car une connection partant ultérieuremen peut encore permettre d'arriver plus tôt en `t`.

Note importante : le **limited walking** n'est une optimisation valide (i.e. qui ne dégrade pas l'exactitude des résutlats) que si le walking graph vérifie la propriété de transitive closure (si à la place on limite la durée de la marche à pieds, cette optimisation peut conduire à louper des itinéraires optimaux).

La reconstruction du chemin (une fois qu'on a trouvé l'EAT) fait l'objet d'une section à part entière, que j'ai sautée.

L'algo est naturellement robuste aux delays (il n'y a presque pas de preprocessing : il s'agit juste de convertir le GTFS en tableau trié, donc dès qu'on a connaissance des délais, on peut se mettre à jour. En pratique, un délai sur un train X doit être propagé au reste des données, et ça prend sans doute plus de temps que la construction d'un tableau trié.

Côté perfs, CSA est annoncé comme 5 à 10 fois plus rapide que RAPTOR (hum...).

## Section 4 = profile connection scan

Juste survolé cette section.

Elle semble mentionner l'optimisation du nombre de transfert (laissé de côté par CSA de base), et pourrait donc renvoyer un pareto-set.

## Section 5 = connection scan accelerated

CSAccel = Connection Scan Accelerated

Je n'ai fait que survoler, mais principe de base = multilevel overlay = partitionner les stops en cellules (même principe qu'avec HypRAPTOR, mais ça reste différent car HypRAPTOR partitionnes les routes et non les stops).

En terme de perf, on peut espérer un gain de x10 (range-queries en 25ms au lieu de 250ms), au prix d'un temps de preprocessing beaucoup plus long (2min au lieu de 10s, soit 10 à 15 fois plus long, même parallélisé sur 16 cores).

Je suis bien d'accord avec la phrase suivante :-)

> Arguably the most important selling point of CSA is its simplicity

Un autre point à retenir : la performance  de CSA ne dépend pas du tout de la structure (topologie) du réseau, uniquement de la quantité de connections ; à l'inverse, la performance de CSAccel est dépendante de la topologie du réseau (vu qu'on le partitionne).

## Section 6 = Minimum Expected Arrival Time

MEAT = Minimum Expected Arrival Time, basé sur CSA.

Je n'ai pas regardé comment ça fonctionnait, mais ça a l'air super intéressant, et ce qu'on obtient en sortie est sympa : au lieu de renvoyer une liste de transfers, renvoie un arbre de décision avec des itinéraires "de secours" si on rate une connection :
- l'iti le plus rapide est A->B->C
- si tu n'arrives pas à temps pour chopper B, il y a un iti alternatif en B2->C
- si tu le loupes aussi, tu peux prendre B3->D->C

Le nom de MEAT vient sans doute de "à quelle heure au minimum du peux arriver si tu loupes telle connexion"

Average query running times = 100 ms.

Il y a une démo sur http://meatdemo.iti.kit.edu
