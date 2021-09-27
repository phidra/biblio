# (ARTICLE) Fast Routing in Very Large Public Transportation Networks using Transfer Patterns

- **url** = [PDF](https://ad-publications.cs.uni-freiburg.de/ESA_transferpatterns_BCEGHRV_2010.pdf) (md5sum=`32dfa773409953ebfdaa8f373e43c28c`), [copie locale](LOCALCOPIES/ESA_transferpatterns_BCEGHRV_2010.pdf)
- **source** = [ESA 2010](http://algo2010.csc.liv.ac.uk/index.php?page=esa). ESA = [European Symposia on Algorithms](http://esa-symposium.org/), conférence générale sur algorithmes, data structures et maths appliquées. [Publication Google](https://research.google/pubs/pub36672/).
- **auteurs** = Hannah Bast, Erik Carlsson, Arno Eigenwillig, Robert Geisberger, Chris Harrelson, Veselin Raychev, and Fabien Viger
- **date de publication** = 2010
- **date de rédaction initiale de ces notes** = 23 octobre 2020

**TL;DR** :
- principe de base = lourde phase de preprocessing pour calculer les transfer patterns de chaque station, utilisés à la query pour accélérer la réponse
- transfer-pattern d'une station `A` = liste des stations où on doit changer pour faire un trajet de façon optimale. Ils sont calculés pour chaque station au preprocessing via un dijkstra multi-critère sur le graphe représentant la timetable.
- à la query, on construit un graphe ad-hoc (au final assez petit) à partir des TP de la station source, puis on dijkstra dessus
- la phase de preprocessing est trop lourde (bien qu'elle se parallélise parfaitement), du coup, on ne traite exhaustivement que quelques stations dites "hub" (un trajet `A->B` est découpé en `A->hub->B`). EDIT : corrigé dans des améliorations ultérieures de la techniques, notamment [frequency-based](2014-frequency-based-search.md)
- lors du preprocessing, il faut faire beaucoup de profile-queries. L'article ne mentionne qu'un dijkstra sur un graphe time-expanded, mais il vaut mieux utiliser rRAPTOR ou CSA, inventés postérieurement à l'article.
- les chemins sont approximés : en effet, les hub stations n'allègent pas suffisamment -> le papier propose une heuristique, qui consiste à limiter à 2 au max le nombre de changements. EDIT : corrigé dans des améliorations ultérieures de la techniques, notamment [frequency-based](2014-frequency-based-search.md)
- Malgré le lourd preprocessing, les TP calculés n'empêchent pas (trop) la prise en compte des mises à jour en temps-réel, cf. [l'article dédié](2013-delay-robustness-in-transfer-patterns.md)


* [(ARTICLE) Fast Routing in Very Large Public Transportation Networks using Transfer Patterns](#article-fast-routing-in-very-large-public-transportation-networks-using-transfer-patterns)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = related work](#section-2--related-work)
   * [Section 3 = problem formalisation](#section-3--problem-formalisation)
      * [cas de base](#cas-de-base)
      * [transferts](#transferts)
      * [autres points](#autres-points)
   * [section 4 = basic algorithm](#section-4--basic-algorithm)
      * [direct-connection queries](#direct-connection-queries)
      * [transfer patterns precomputation](#transfer-patterns-precomputation)
      * [query](#query)
   * [section 5 = hub stations](#section-5--hub-stations)
   * [section 6 = further refinements](#section-6--further-refinements)
   * [section 7 = heuristic optimizations](#section-7--heuristic-optimizations)
   * [section 8 = experiments](#section-8--experiments)

**NOTE** : d'après [ce post de blog](2016-03-02-post-google-transfer-patterns.md), c'est ce qui est utilisé par Google pour leurs calculs d'itinéraires PT depuis 2010. À partir de 2016, ils utilisent une version améliorée de Transfer Patterns, leur permettant de scaler à l'échelle d'un pays.

## Section 1 = introduction

**TL;DR** : au preprocess, on précalcule un subset de transfer patterns (TP) optimaux. À la query, on combine ce subset de TP pour fabriquer un *query-graph* adapté à cette query, qui nous permet de trouver les chemins optimaux. 

Les algos (tricks) pour calcul d'itinéraire sur road networks fonctionnent très mal sur public-transportation networks.

Query = `A@t → B` (on veut partir d'une station A à t et arriver à une station B).

Principe général = précalculer que tous les chemins optimaux allant de Freiburg à Zürich sont :

- soit directs Freiburg→Zürich
- soit ont un changement à Basel : Freiburg→Basel→Zürich

Freiburg→Zürich et Freiburg→Basel→Zürich sont les **transfer patterns** de la paire *{source,target}* particulière qui est *{Freiburg,Zürich}*.

En pratique, on ne peut pas précalculer les transfer patterns optimaux de TOUTES les paires possibles -> on en précalcule un subset, qui permet de reconstituer les transfer patterns de toutes les autres paires.

Côté chiffres (à la date de publication de l'article = 2010) :
- temps de preprocess de 20 à 40 core-hours par million de paires
- espace de stockage de 10 à 50 Mio par millier de stations
- structure de données pour répondre aux *direct-connection queries* nécesite 3 à 10 Mio par millier de stations, et répond en 2 à 10 µs

À noter que la combinaison du subset de transfer patterns pourra conduire également à utiliser des transfer patterns non-optimaux, mais que c'est pas très grave (on va les éliminer car elles ne seront pas sur le front de Pareto, ça conduira juste à un peu plus de travail à la query).


À partir de la query A@t→B, à l'aide du subset précalculé, on calcule tous les transfer-patternss allant de A à B (donc qui incluent des TP non-optimaux). On merge tous ces TP, ça forme un graphe ad-hoc (dit *query grah*) spécialement forgé pour la query considérée.

Sur ce graphe, chaque arc correspond à une *direct-connection query*. Pour l'exemple donné plus haut, le graphe sera :

```
Freiburg────>────Zürich
    │               │
    └──>──Basel──>──┘
```

En pratique, comme le query graph combine les divers TP précalculés (ceux qui font partie du subset), un query-graph typique aura quelques centaines d'arcs.

## Section 2 = related work

Quelques articles généraux qui ont l'air intéressants :

- [Car or Public Transport—Two Worlds](2009-car-or-public-transport-two-worlds.md) = article de 2009 expliquant pourquoi le calcul d'itinéraire sur un réseau routier et sur un réseau de transport public sont deux problèmes très différents.
- [Engineering Route Planning Algorithms](https://i11www.iti.kit.edu/extra/publications/dssw-erpa-09.pdf) = état des lieux de 2009 sur le calcul d'itinéraire sur road network, et les limites. Beaucoup (76) de références. Attention : la lecture de [ce survey plus récent](2015-route-planning-in-transportation-networks.md) est sans doute plus pertinente.
- [Timetable Information: Models and Algorithms](https://i11www.iti.kit.edu/extra/publications/mswz-tima-06.pdf) = état des lieux de 2006 des modèles et algos résolvant l'EAP (Earliest Arrival Problem) à partir de timetable information. Attention : la lecture de [ce survey plus récent](2015-route-planning-in-transportation-networks.md) est sans doute plus pertinente.

## Section 3 = problem formalisation

### cas de base

Modèle de données proche de celui présenté dans le papier sur RAPTOR, et proche du format GTFS (assez logique, vu que ce format a sans doute été créé par Google en même temps que leur implémentation des transfer-patterns) :
- **stations** = bus stops, train stations, ports
- **trip** = une séquence particulière de stations, avec l'heure d'arrivée et de départ à chaque station
- **line** (introduit dans la section suivante) = groupes de trips partageant exactement la même séquence de **stations**, sans dépassement
- **minimum transfer duration** = en une station `B` donnée, `ΔtB` indique le temps nécessaire pour descendre d'une ligne et remonter dans une autre ligne, à la même station `B`
- **elementary connection** = c'est simplement le fait d'avancer d'une station (d'aller de `A` à `B`) en restant dans son métro, sans faire de transfert

Les stations sont représentées dans un graphe, où une même station A (qui offre un trajet vers une station B) va avoir :
- `Ad@t₁` = un noeud représentant le fait qu'on part ( **d**eparture) de `A` à `t₁`
- `Ba@t₂` = un noeud représentant le fait qu'on arrive ( **a**rrival) à `B` à `t₂`
- `Ad@t₁ → Ba@t₂` = un arc indiquant on part de `A` à `t₁` et qu'on arrive à `B` à `t₂`
- `Ba@t₂ → Bd@t₃` = un arc indiquant qu'en restant dans le transport, on peut "poursuivre" le trajet, et repartir de B à `t₃` (possiblement, `t₂` et `t₃` sont égaux, ça ne pose pas problème)

Illustration du cas de base = une ligne de transport qui passe par A, B et C, sans transfert :

```
┄───Ad@t₁────>────Ba@t₂────>────Bd@t₃────>────Ca@t₄───┄
```

### transferts

Côté transfers (p.ex. en `B`, on change de ligne), on ajoute un node représentant le transfert :
- `Bt@t₅` = un noeud représentant le fait qu'on peut faire un **t**ransfert (changement de ligne) en B, et arriver à la nouvelle ligne en `t₅`
- chaque transfer node est relié au transfer suivant : `Bt@t₅────>────Bt@t₆` en formant une **waiting chain** (je suppose pour représenter le fait qu'on peut si on le souhaite "laisser passer un train" ? ou bien pour indiquer qu'on ne prend pas nécessairement le premier transfer disponible ?)
- pour représenter la possibilité de faire un transfert, on met un arc de transfert : `Ba@t₂────>────Bt@t₅`, vers le premier noeud `Bt@tx` respectant la contrainte `tx >= t₂ + ΔtB`


Illustration = une ligne de transport qui passe par A, B et C, avec un transfert possible en `B` pour rejoindre une ligne dont la prochaine station est `D` :

```
┄───Ad@t₁────>────Ba@t₂────>────Bd@t₃────>────Ca@t₄───┄
                    │
                  Bt@t₅────>────Dd@t₇──┄
                    │
                  Bt@t₆
                    │
                    ┆
```

Conséquence à garder en tête : dans ce graphe, une même station A peut avoir 3 types de noeuds :
- `Aa@t` = noeud d'arrivée à `A`
- `Ad@t` = noeud de départ à `A` (en restant sur la même ligne)
- `At@t` = noeud de transfert depuis `A` vers une autre ligne

### autres points

- l'origine des temps (temps `0`) est le premier jour à minuit
- pour exploiter la périodicité = se limitent à 24h + un bitmask pour indiquer les jours où un trip est actif ou non
- autorisent des transferts à pieds entre stations proches
- chaque arc est associé à une **DURÉE** (différence des `t` entre ses nodes) et un **COST** arbitraire
- la notion de cost arbitraire permet d'implémenter beaucoup de choses, l'article s'en sert 1. pour pénaliser les transferts et ainsi privilégier des résultats minimisant les transferts et 2. pour également pénaliser les **elementary connections**, et ainsi privilégier des résultats avec moins de stations
- mon interprétation : une façon simple de minimiser le nombre de transfert et d'attribuer un cost de 1 à chaque arc de transfert, et un cost de 0 à tous les autres -> on retombera sur des résultats équivalents au RAPTOR simple
- la **waiting chain** permet d'assurer que si on arrive à la station `X` en `tx`, alors on sera capable de prendre **n'importe quel** transport public qui quitte `X` **après** `tx`

**feasible connection** = tous les chemins permettant d'aller de `A` à `B`, même s'ils sont non-optimaux
**optimal connection** (resp. optimal cost) = le chemin (resp. coût) permettant d'aller de `A` à `B`, tel qu'il ne soit pas dominé par un aure chemin.

## section 4 = basic algorithm

(l'algo présenté dans cette section n'est pas l'algo définitif, car il a une complexité au preprocess qui est quadratique)

### direct-connection queries

- **direct connection** = pas de transfert.
- permet de répondre rapidement (10 µs) aux queries `A@t → B` qui ne nécessitent pas de changement
- introduit la notion de **line**, à précalculer en amont
- une **line** est un tableau 2D, où chaque colonne est une **station**, chaque ligne est un **trip**, et chaque cellule du tableau est un 2-uple contenant l'heure d'arrivée à, puis de départ de la station considérée, pour le trip considéré :

| line L17 | S154 | S097 | S987 | S111 |
|----------|------|------|------|------|
| **trip 1** | 8:15 | 8:22 8:23 | 8:27 8:29 | 8:38 8:39 |
| **trip 2** | 9:14 | 9:21 9:22 | 9:28 9:28 | 9:37 9:38 |
| **trip X** |  …   | … …       | … …       | … …       |

On construit une deuxième structure de donnée ( **incidence list**) pour chaque station indiquant les lignes qui la parcourent (ordonnées par id de **line**), en notant la position à laquelle la station apparaît dans la ligne considérée.

Par exemple, la station `S097` a l' **incidence list** suivante :

```
L8(4)  /  L17(2)  /  L34(5)  /  L87(17)
```

Ce qui signifie que 4 lignes différentes passent par `S097`, et qu'elle apparaît en deuxième position de la ligne `L17`.

Derrière, pour répondre à une query `A@t → B`, et retrouver la **direct connection** résolvant l'EAP :
- on récupère l' **incidence list** de `A`, puis celle de `B`, et on regarde les lignes qui apparaîssent dans les deux lists : ce sont les lignes qui passent à la fois par `A` et par `B`
- parmi toutes les lignes candidates, on ne garde que celles où la station `A` apparaît avant la station `B`
- pour chaque ligne, on itère sur tous trips qui partent de `A` pour retrouver le premier trip qui part de `A` après `t`
- pour chaque ligne, on regarde à quelle heure ce premier trip arrive en `B`, et on garde le meilleur (ou les meilleurs en cas d'égalité)

### transfer patterns precomputation

**transfer pattern** d'un chemin donné (pas forcément optimal) = première station (+ toutes les stations où on a un transfert) + dernière station

**Optimal set of transfer patterns for a pair of stations (A,B)** = un set `S` de TP tel que quel que soit le `t` auquel on part de `A` :
- il existe un chemin optimal de `A` à `B` dont le TP est dans `S`
- tout TP dans `S` est le TP d'une **optimal connection** de `A` à `B` à au moins un temps `T`
- NdM : la définition n'implique pas de façon limpide que tous les chemins optimaux ont leur TP dans `S`... Dit autrement, il manque à cette définition une contrainte du genre : _tout chemin optimal de `A` à `B` il existe un chemin optimal dont le TP est dans `S`_ Pourtant, plus tard dans l'article, on laisse à entendre que c'est le cas.

Structure pour représenter les TP d'une station :
- à partir d'une station `A` on calcule l' **optimal set of TP** vers toutes les stations `B` accessibles à `A`.
- à partir de ce set `S`, on construit un DAG (pour la station `A`), qui sera la structure représentant les TP de `A`.
- ce DAG a trois noeuds :
    - un seul **root node** = la source-station `A`
    - autant de **leaf nodes** que de target-stations `B`, `C`, etc.
    - un **middle node** (dénomination perso) par préfixe de TP
- plus de précisions sur le **middle node** : s'il existe les TP optimaux suivants de `A` vers `E` :
    - `AE`
    - `ABE`
    - `ABCDE`
    - `ABDE`
- on leur attribue un label correspondant à leur dernière station (et on retrouve le préfixe en "remontant" le DAG du middle-node vers la source). Les **middle nodes** de l'exemple plus haut auront les labels suivants, avec leur préfixe associé :
    - `B` : AB
    - `C` : ABCD
    - `D` : ABCD **et** ABD (il y a deux noeuds avec le même label)
    - `AD`
- (dans le papier, on gagnerait en clarté à réécrire chaque middle-node comme son préfixe plutôt que son label : AB, ABC, ABCD, ABD)

NdM : trouver les chemins optimaux de `A` vers toutes les stations quel que soit `t` nécessite des calculs énormes ?! Comment éviter d'avoir à faire ça pour chaque minute (voire seconde ?!) de la timetable ? EDIT : justement, le papier indique que c'est effectivement énorme, et propose une solution pour le diminuer.

Algo pour calculer les TP de `A`, en gros :
- l'objectif est de calculer le DAG des TP de `A`
- Dijkstra multi-critère (**EDIT** : les profile-queries peuvent être implémentées plus efficacement qu'avec un dijkstra multi-critère, en utilisant CSA ou RAPTOR, inventés après la rédaction de cet article) qui démarre sur tous les transfer nodes de la station `A`, qui va associer à chaque target-station `B` un set de labels (u, v) indiquant l'heure d'arrivée + le coût.
- (À CLARIFIER) pour une target-station `B` donnée, parmi tous les labels, on choisit un subset de labels (donc de chemins) en appliquant **arrival chain algorithm**
- (À CLARIFIER) pour chaque temps `ti` d'arrivée possible (pas forcément optimal) dans les labels, on retient 1. les chemins dont le label est settled en `ti` (donc les chemins qui ne peuvent plus améliorer leur temps d'arrivée en `B`) et 2. les chemins dont le label était sélectionné pour `ti-1`, avec leur durée augmentée de `ti - ti-1`
- pour tous les labels retenus, on reconstruit le chemin, leur transfer pattern, et on construit au final le DAG de ces transfer patterns

Ce preprocess se parallélise très bien (on peut préprocesser les patterns de toutes les stations sources en parallèle), mais ça ne suffit pas à rendre l'algo faisable sur des données réelles, même non-nationales.

### query

À partir d'une query `A@t → B`, on utilise le DAG de `A` pour construire un graphe ad-hoc (contenant toutes les stations apparaissant dans les TP permettant d'aller de `A` à `B`). Note : comme tous les transferts sont des noeuds sur ce graphe, aller d'un node à un autre revient à faire un trajet sans transfert, résolvable avec une **direct connection query** décrite plus haut.

Derrière, trouver les chemins optimaux revient à faire un time-dependent multi-criteria dijkstra sur ce graphe ad-hoc.

## section 5 = hub stations

Même s'il est parallélisable, le preprocessing est trop lourd, et conduit à des preprocessed data trop volumineuses.

**Principe de base pour limiter ce preprocessing** = on présélectionne un set de **hub-stations**, et on fait un calcul exhaustif de TP (dit _global_ dans l'article) uniquement sur ces hub stations. Pour toutes les autres stations, on ne calcule que les TP (dit _local_ dans l'article) de chemins qui ne passent pas par des hubs, soit des TP vers les hub stations sans aller plus loin que la première hub station, qu'on appelle **access station**.

**Comment identifier les hub-stations** : on s'affranchit du côté time-dependent des donées, on calcule beaucoup de chemins random (en utilisant le cost comme poids des edges, donc en minimisant le nombre de stations parcourues / de transferts effectués), et les hubs sont les stations qui apparaissent le plus dans les chemins.

**Ce qui change au preprocess des TP locaux** : dans le MC-dijkstra, on arrête de relaxer les noeuds s'ils appartiennent à une hub-station.

**Ce qui change à la query** : en plus de construire le graphe ad-hoc à partir du DAG de `A` pour aller à `B`, on le construit pour aller de `A` à toutes ses **access-stations**, et de chaque access stations vers `B`.

## section 6 = further refinements

Cette présentation de l'algo est en fait "simplifiée" : leur vrai algo a pas mal de tweaks :
- **location to location** : le graphe ad-hoc contient des arcs reliant la location réelle à quelques stations les plus proches, avec comme poids la durée de marche à pieds
- **walking transfers** : le graphe sur lequel sont calculés les TP se voit augmentés de nodes et arcs supplémentaires représentant les transferts à pied
- **more compact graph model** : mineur, cf. papier
- **query graph search** : mineur, cf. papier

## section 7 = heuristic optimizations

S'arrêter aux hub stations ne suffisent pas (car même en s'arrêtant à la première hub-station rencontrée, les local search ont tout de même beaucoup trop de chemins à explorer, notamment qui contournent ou évitent des hub-stations).

Dans le papier, ils n'ont pas de solution à la fois rapide et exacte -> ils utilisent une heuristique.

Leur heuristique = dans le calcul des TP, limiter les local search à 2 transferts.

En pratique, sur 10k chemins en Suisse, seuls 3 résultats étaient non-optimaux (et encore, pas trop mauvais), et la mauvaise qualité de la donnée était souvent plus embêtante que l'approximation dûe à leur heuristique.

Le papier liste quelques autres heuristiques de moindre importance.

## section 8 = experiments

Ils ont conduit leurs expériences sur 3 réseaux :
- petit = 20k stations = Suisse
- moyen = 30k stations = NY
- grand = 338k stations = North-America
