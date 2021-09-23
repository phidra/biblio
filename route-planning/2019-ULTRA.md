# (ARTICLE) UnLimited TRAnsfers for Multi-Modal Route Planning: An Efficient Solution

- **url** = [PDF](https://arxiv.org/pdf/1906.04832.pdf) (md5sum=`6c0762c8458206c9af5ac6e46ae951cf`), [copie locale](LOCALCOPIES/1906.04832.pdf)
- **source** = conf: [ESA 2019](https://drops.dagstuhl.de/opus/portals/lipics/index.php?semnr=16123)
- **auteurs** = Moritz BAUM, Valentin BUCHHOLD, Jonas SAUER, Dorothea WAGNER, Tobias ZÜNDORF, du KIT ([Karlsruhe Institute of Technology](https://www.kit.edu/english/))
- **date de publication** = 2019
- **date de rédaction initiale de ces notes** = 6 janvier 2021
- **code** = [ULTRA](https://github.com/kit-algo/ULTRA/), sur [le repo du KIT](https://github.com/kit-algo/)

**TL;DR** = still to do

* [(ARTICLE) UnLimited TRAnsfers for Multi-Modal Route Planning: An Efficient Solution](#article-unlimited-transfers-for-multi-modal-route-planning-an-efficient-solution)
   * [Section 1 : Introduction](#section-1--introduction)
* [Notes d'analyse du code](#notes-danalyse-du-code)
   * [Vrac](#vrac)
   * [Données binaires en entrée du pipeline](#données-binaires-en-entrée-du-pipeline)
      * [Fichier binaire RAPTOR](#fichier-binaire-raptor)
   * [Contraction](#contraction)
      * [StopCriterion](#stopcriterion)
   * [Graphe forward/backward / core + isCoreVertex](#graphe-forwardbackward--core--iscorevertex)
      * [Notes sur forward et backward](#notes-sur-forward-et-backward)
      * [Notes sur isCoreVertex / core](#notes-sur-iscorevertex--core)
   * [Comment est défini l'ordering ?](#comment-est-défini-lordering-)
   * [Serialization du binaire RAPTOR](#serialization-du-binaire-raptor)
   * [Format des departure_time/arrival_time](#format-des-departure_timearrival_time)
   * [Fonctionnement de l'algo RAPTOR utilisant les shortcuts ULTRA](#fonctionnement-de-lalgo-raptor-utilisant-les-shortcuts-ultra)
      * [arrivalByRoute / arrivalByTransfer](#arrivalbyroute--arrivalbytransfer)
      * [EarliestArrivalLabel](#earliestarrivallabel)
      * [Reconstruction du chemin complet](#reconstruction-du-chemin-complet)
      * [Données d'entrée](#données-dentrée)
      * [Données de sortie](#données-de-sortie)
      * [Quelles structures gèrent les transferts dans l'algo = baseQuery](#quelles-structures-gèrent-les-transferts-dans-lalgo--basequery)
         * [rôle de la baseQuery](#rôle-de-la-basequery)
      * [Bucket CH](#bucket-ch)
      * [Apport de bucket CH par rapport à CH classique](#apport-de-bucket-ch-par-rapport-à-ch-classique)
         * [buildBucketGraph](#buildbucketgraph)
         * [Point de détail = syntaxe que je ne connaissais pas](#point-de-détail--syntaxe-que-je-ne-connaissais-pas)
      * [Utilisation de initialTransfers par ULTRARAPTOR](#utilisation-de-initialtransfers-par-ultraraptor)
      * [In fine, comment reconstruire le chemin](#in-fine-comment-reconstruire-le-chemin)

## Section 1 : Introduction

FIXME : notes à prendre.


# Notes d'analyse du code

**DISCLAIMER** : ces notes sont des notes très très vrac, assez bordéliques, et à prendre avec des pincettes.

## Vrac

* _custom binary format for loading the public transit network as well as the transfer graph. As an example we provide the public transit network of Switzerland together with a transfer graph extracted from OpenStreetMap in the appropriate binary format_ ([lien pour DL les données](https://i11www.iti.kit.edu/PublicTransitData/Switzerland/binaryFiles/))
* cloc indique que le code d'ULTRA pèse ~13k lignes de C++
* QUESTION : où l'ordering a-t-il lieu ? (la data [a un membre order](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L152), mais je ne sais pas encore où il est calculé)

## Données binaires en entrée du pipeline

**TL;DR** : `BuildCoreCH` nécessite en entrée :
* `raptor.binary` = le fichier binaire stockant le même genre d'infos que le GTFS
* `raptor.binary.graph.XXX` = une série de fichiers stockant le graphe des transferts, sous forme de plusieurs tableaux, au format [CSR](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_row_(CSR,_CRS_or_Yale_format)), i.e. le même format que les AdjacencyArray
    + `raptor.binary.graph.beginOut` = tableau de Nv+1 éléments, où Nv est le nombre de vertex du graphe : chaque index est un numéro de vertex, chaque cellule du tableau contient l'indice du premier edge du vertex dans les autres tableaux (à propos du nom `beginOut` : la cellule du tableau contient l'indice du premier out-edge)
    + `raptor.binary.graph.coordinates` = (probablement) tableau de Nv éléments : chaque index est un numéro de vertex, chaque cellule du tableau contient les coordonnées du vertex
    + `raptor.binary.graph.toVertex` = tableau de Ne éléments, où Ne est le nombre d'edges du graphe : chaque index est un numéro d'edge, chaque cellule contient le vertex target de l'edge
    + `raptor.binary.graph.travelTime` = tableau de Ne éléments, chaque index est un numéro d'edge, chaque cellule contient le poids de l'edge

Le premier binaire à appeler est [BuildCoreCH](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L77), qui prend en entrée :
* le fameux fichier binaire (e.g. `DATA/complete/raptor.binary`)
* un paramètre `coreDegree` qui semble paramétrer l'arrêt de la contraction
* le répertorie de sortie de la contraction

```cpp
const std::string raptorFile = argv[1];  // DATA/complete/raptor.binary
RAPTOR::Data data = RAPTOR::Data::FromBinary(raptorFile);
```

* FromBinary [se contente de deserializer](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L66) à partir du chemin du fichier
* la deserialization [déserialize les deux structures](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L395) : les données PT depuis `raptor.binary` (à l'oeil, semblent équivalentes au GTFS), et le transferGraph depuis les `raptor.binary.graph.XXX`
* en remontant la chaîne, le transferGraph est un `StaticGraph` : [lien1](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L53) -> [lien2](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Graph.h#L37) -> [lien3](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/GraphInterface.h#L117) -> [lien4](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/StaticGraph.h#L45)
* la [déserialization d'un StaticGraph](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/StaticGraph.h#L501) déserialize le fichier `raptor.binary.graph.beginOut`, puis tous les fichiers `raptor.binary.graph.PROPERTY` correspondant aux properties des vertex et edges

### Fichier binaire RAPTOR

Rappel GTFS :
- stop = station
- route = enchaînement de stops particuliers
- trip = instance d'une route à un instant T donné (= véhicule qui fait un trajet)
- stop-event = arrêt d'un véhicule à un trip donné
    + les stop-events se partitionnent par véhicule en trips (les stop-events consécutifs d'un même véhicule forment un trip)
    + les trips se partitionnent par route (les trips partageant le même set de stops forment une route)

Analyse des tailles des données du binaire sur la Suisse [en téléchargement libre](https://i11www.iti.kit.edu/PublicTransitData/Switzerland/binaryFiles) :

```
stopData                     = 25426
---
firstRouteSegmentOfStop      = 25427
routeSegments                = 183113
---
routeData                    = 13934
---
firstStopIdOfRoute           = 13935
stopIds                      = 183113
---
firstStopEventOfRoute        = 13935
stopEvents                   = 4740929
---
implicitDepartureBufferTimes = false
implicitArrivalBufferTimes   = false
----------------------------------------
RAPTOR public transit data:
   Number of Stops:                25,426
   Number of Routes:               13,934
   Number of Trips:               369,534
   Number of Stop Events:       4,740,929
   Number of Connections:       4,371,395
   Number of Vertices:            604,167
   Number of Edges:             1,847,140
   First Day:                           0
   Last Day:                            2
   Bounding Box:             [(5.96054, 45.5748) | (10.667, 48.1195)]
```

Si je résume, le binaire raptor c'est :
- un set de stops
- un set de routes
- un set de trips

Ma compréhension :
- tout fonctionne avec la structure [CSR](https://en.wikipedia.org/wiki/Sparse_matrix#Compressed_sparse_row_(CSR,_CRS_or_Yale_format)), aussi appelée `AdjacencyArray` (d'où le fait que les tructures de type firstXXX aient un élément de plus que la liste des xxxData)
- chaque route comporte un set de stops (firstStopIdOfRoute + stopIds)
- chaque route comporte un set de stopEvents (firstStopEventOfRoute + stopEvents)
- comme les stopEvents ne sont que des couples departure_time+arrival_time, il faut bien être capable d'identifier les trips d'une route -> ?

Identifier les trips d'une route
- une possibilité = les stopEvents sont regroupés par chunk de N éléments (où N = le nombre de stops d'une route donnée)
- dit autrement = les trips sont les uns à la suite des autres
- pour le confirmer : le set de stopEvents d'une route donnée doit être divisible par N = le nombre de stops d'une route donnée
    + je confirme, et c'est même [la définition du nombre de trips dans le code](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L154)
    + le plus logique serait que les events d'un trip donné soient consécutifs
    + à la fois par le code et par les données, je confirme : les stopEvents sont groupés par paquets de N, et à la fin du paquet, on revient un peu en arrière, cf. cet exmple :

Exemple pour illustrer : la route n°42 a 15 stops, et tous les 15 stops, le départ du stop revient un peu en arrière (signe qu'on passe à autre trip).

```
La route n°42 est : Route{[18] T 18, 0}Elle contient #stops = 15
Elle contient #trips = 8
Elle contient #stopEvents = 120
        [0] 61620
        [1] 61680
        [2] 61740
        [3] 61860
        [4] 61920
        [5] 62040
        [6] 62160
        [7] 62220
        [8] 62280
        [9] 62400
        [10] 62520
        [11] 62640
        [12] 62700
        [13] 62820
        [14] 62941   <-- dernier event d'un trip
        [15] 62400   <-- premier départ du trip suivant
        [16] 62460
        [17] 62520
        [18] 62640
        [19] 62700
        [20] 62820
        [21] 62940
        [22] 63000
        [23] 63060
        [24] 63180
        [25] 63300
        [26] 63420
        [27] 63480
        [28] 63600
        [29] 63721  <-- dernier event d'un trip
        [30] 63120  <-- premier départ du trip suivant
        [31] 63180
        [32] 63240
        [...]
```

C'est quoi les `RouteSegments` ?
- D'après la donnée Data.h raptor, ça permet, à partir d'un stop donné, de retrouver efficacement les routes qui le contiennent un stop
- dans le code, c'est l'agrégat d'une RouteId et d'un StopIndex.

Concernant les `implicitDepartureBufferTimes`, d'après le code, on dirait que ça revient à intégrer les minTransferTime des stops dans le temps de trajet du stop.

## Contraction

```cpp
CoreCHData contractionData = buildCoreCH(data, coreDegree);
```

* c'est la fonction [buildCoreCH](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L44) qui fait la contraction, elle n'a que deux paramètres :
    + les données (j'imagine que seul le transferGraph est utilisé)
    + un paramètre `coreDegree` (qui semble servir à la terminaison de l'algo, mais c'est pas encore clair ce qu'il fait)
* la [donnée en sortie](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L39), qui est sérializée, est un agrégat de :
    + les [data RAPTOR](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L55) (qui semblent inchangées, vu que l'agrégat est initialisé sur une const-reference)
    + la donnée CH [est un agrégat de deux CHGraph](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/CH.h#L136) : forward+backward. On le voit bien dans les données de sorties de BuildCoreCH, qui sont deux jeux de fichiers :
        + `ch.forward.attributesSize`
        + `ch.forward.beginOut`
        + `ch.forward.statistics.txt`
        + `ch.forward.toVertex`
        + `ch.forward.viaVertex`
        + `ch.forward.weight`
* les [CHGraph sont des StaticGraph](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Graph.h#L42) où les vertex n'ont pas d'attributs, et où les edges, en plus d'avoir un weight de type `int`, ont un `ViaVertex`, [qui est un TaggedInteger](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Helpers/Types.h#L32) (NdM : ce ViaVertex est sans doute le noeud contracté pour les shortcuts).
* Du coup, ma compréhension des choses, c'est que l'output de la contraction est un graphe contracté.
* QUESTION : il semble y avoir [une notion de CoreGraph](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L53), avec des vertices, c'est pas encore clair ce qu'il y a derrière.
    + [on dirait](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L134) que le core, c'est juste le graphe (?)
    + en tout cas, le run [contracte les vertices](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L106).
    + AH, je pense avoir partiellement compris : comme on peut arrêter la contraction plus tôt, il peut rester des nodes dans la queue qui n'ont pas été contractés -> `copyCoreToCH` les ajoute aux graphes de sortie (avec leurs edges).

### StopCriterion

C'est [quand le StopCriterion renvoie true qu'on arrête la contraction](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L215).

Le StopCriterion [est initialisé à](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L51) : `StopCriterion(numberOfStops, coreDegree)`.

Ce critère [arrête l'algo quand](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/StopCriterion.h#L53) il reste moins de noeuds à contracter que le minCoreSize, ou quand le degré moyen des noeuds restants à contracter devient supérieur au maxCoreDegree (à cause des shortcuts ajoutés). Dans ce code, si je dis pas de bêtises, `data->coreSize()` est le nombre de noeuds pas encore contracté, et `data->core.numEdges` est le nombre d'edges des noeuds pas encore contractés.

À creuser = [ce qu'est coreSize](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHData.h#L69) = `return level.size() - order.size();`. `data.order` [semble être](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHData.h#L82) un tableau avec les nodes ordonnés (sans doute construit petit à petit), et c'est pas encore super clair ce qu'est `level`.

## Graphe forward/backward / core + isCoreVertex

### Notes sur forward et backward

* pour un vertex `u` donné, **forward** contient ses out-edges (i.e. les edges `u -> v`)
* pour un vertex `u` donné, **backward** contient ses in-edges (i.e. les edges `p -> u`)
* à confirmer : d'après ce que je comprends, les edges ne respectent pas forcément la contrainte `rank(v) > rank(u)` (comme la contraction peut être partielle, ça dépend de si les noeuds ont été contractés ou pas)
* question : c'est acquis que **backward** stocke les `in-edges` de `u`, mais dans quel "sens" sont-ils stockés ? Que renvoie `backward.edgesFrom(u)` ?
    + l'utilité de cette structure **backward** est de permettre le Dijkstra bidirectionnel ; le problème qu'on essaye de résoudre est : étant donné un vertex `u` (par exemple, la destination du trajet), trouver tous ses prédécesseurs (de rank supérieur)
    + il faut donc (à partir d'un vertex `u`) pouvoir itérer sur tous ses prédécesseurs, i.e. tous les vertex `p` pour lesquels il existe un edge `p -> u`
    + ma compréhension est donc : `backward.edgesFrom(u)` renvoie tous les `p`
    + la petite subtilité qui rend le tout contre-intuitif, c'est que le prédécesseur est désigné par `ToVertex` (alors que c'est un prédécesseur). Du coup, une bonne façon de voir les choses à mes yeux est plutôt de considérer le graphe backward comme un graphe direct dans le dual du graphe initial, permettant un cheminement à faire à rebours, "en sens interdit" (par exemple, depuis le vertex de destination du trajet souhaité par l'utilisateur). Vu comme ça, le fait de désigner un prédécessuer par `To` n'est pas problématique (c'est celui vers lequel on se dirige, quand on marche en sens interdit)
* concrètement, **forward** et **backward** étant des CHGraph (qui sont des StaticGraph), ils stockent des edges au format CSR, et `edgesFrom` permet donc de récupérer les out-edges d'un vertex donné : [lien](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/StaticGraph.h#L143), ce sont les `edges` qui partent `From` un vertex.

Un point très important : à cause du stopCriterion, [on peut arrêter la contraction alors qu'il reste des vertex à contracter](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L215). Du coup, les graphe forward et backward semblent contenir un état "partiellement" contracté.

**NOTE** : je pense que j'ai raté quelque chose, car pour moi, une contraction doit être complète pour permettre le calcul des PCC. Par exemple, sur le graphe suivant, on ne pourra pas faire le trajet `A -> B` si on ne contracte que les 4 premiers vertex :

```
# graphe initial :
 1     2     7     5     6     3     4
 A-----B-----C-----D-----E-----F-----G

# graphe après contraction des 4 premiers vertex :
 1           7     5     6           4
 A-----------C-----D-----E-----------G
```

En effet, le trajet forward s'arrêtera à C (sans pouvoir rejoindre D de rank inférieur) et le trajet backward s'arrêtera à E (sans pouvoir rejoindre D de rank inférieur).

Affaire à suivre...

**EDIT** : lorsqu'on arrête la contraction plus tôt, la contrainte "on n'a le droit de se diriger que vers les nodes de rank supérieur" n'est PAS appliquée aux nodes non-contractés. (en effet, on n'a pas encore remplacé leurs edges par des edges se dirigeant vers des nodes de rank supérieur, du coup, certains chemins seraient impossibles). Dans l'exemple ci-dessus, comme `E` n'a pas encore été contracté, on a le droit de relaxer l'edge `E → D`, et ce même si `D` est de rank inférieur à `E`.

À noter qu'au moins dans [l'implémentation des CH du RoutingKit](https://github.com/RoutingKit/RoutingKit), la contrainte de se diriger vers des nodes de rank supérieur n'est pas vérifiée dynamiquement, mais imposée par construction, puisque le query-graph ne contient QUE les edges vers des nodes de rank supérieur. Ce choix d'implémentation permet de facilement "laisser" des out-edges de noeuds non-contractés, vu qu'ils ne sont au final pas différemment des out-edges des noeuds contractés.

### Notes sur isCoreVertex / core

* Que fait `isCoreVertex` :
```cpp
inline bool isCoreVertex(const Vertex vertex) const noexcept {
    for (const Edge forwardEdge : forward.edgesFrom(vertex)) {
        for (const Edge backwardEdge : backward.edgesFrom(forward.get(ToVertex, forwardEdge))) {
            if (backward.get(ToVertex, backwardEdge) == vertex) return true;
        }
    }
    return false;
}
```

TL;DR : `isCoreVertex` indique si un vertex fait partie de ceux qui ont été contractés ou non, il pourrait être renommé en `isVertexContracted` :
* on itère sur tous les successeurs de `u`, qu'on va appeler `v`
* pour chaque successeur `v`, on itère sur ses prédécesseurs
* si on retrouve `u` dans les prédécesseurs de `v`, c'est que `v` est un core-vertex...

Ça reste à confirmer (car ça me paraît impossible d'avoir des contractions partielles), mais un core-vertex serait un vertex non-contracté :
* je présuppose que les graphes forward et backward contiennent à la fois des vertex contractés et des vertex non-contractés
* cas 1 = si le vertex a été contracté :
    + il n'a plus que des successeurs de rank supérieur (car pendant sa contraction, ses out-edges vers des successeurs de rank inférieur ont été remplacés par des shortcuts vers des vertex de rank supérieur)
    + réciproquement, il n'est le prédecesseur d'aucun de ses voisins (vu que sa contraction a remplacé l'edge par un shortcut)
* cas 2 = si le vertex n'a pas été contracté :
    + il se peut qu'il ait des successeurs de ranks inférieurs (si eux aussi n'ont pas encore été contractés)
    + s'il a des voisins de rank supérieur, il apparaîtra comme leur prédécesseur (car il n'a pas encore été contracté)
* du coup, si `v` est un successeur de `u` (peu importe le rank de `v`), `u` ne sera un prédecesseur de `v` que si `u` n'a pas encore été contracté.
* (note : possiblement, vu leur process de contraction qui semble mélanger contraction et ordering, parler de rank pour des vertex pas encore contractés n'a pas de sens)

## Comment est défini l'ordering ?

Les vertex sont [dequeués](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L216) (donc en fonction de leur poids dans la Q), puis [contractés](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L218). Et justement, c'est la contraction qui [sette l'ordering](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L223). Dit autrement, c'est le poids du vertex dans la queue qui définit l'ordre dans lequel il est contracté.

Est-ce ce k-heap qui est implémenté ? https://www.geeksforgeeks.org/k-ary-heap/

Au sein du heap, [ce qui permet d'ordonner les éléments](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Container/ExternalKHeap.h#L89) (donc leur poids dans la Q), ça semble être `hasSmallerKey`, qui utilise [la propriété `key` des vertex](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L64), lui-même utilisant le template-parameter [KEY_FUNCTION](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Preprocessing/CHBuilder.h#L47) du Builder.

Dans notre cas, il s'agit de [celle-ci](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L34), qui a l'air un peu complexe -> à étudier.

En résumé (même si j'ai pas fini d'étudier), l'ordering est défini par rapport à [cette KeyFunction](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L34).

## Serialization du binaire RAPTOR

La [lecture du binaire RAPTOR](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/BuildCoreCH.cpp#L80) est :

```cpp
RAPTOR::Data data = RAPTOR::Data::FromBinary(raptorFile);
```

Enchaînement : [fromBinaryh](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L64) -> [deserialize](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L395), qui déserialize les DEUX parties du binaire RAPTOR : [les données PT](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L396), et [le transferGraph](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L397).

La déserialization des données PT utilise [du code générique de déserialization](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Helpers/IO/Serialization.h#L181) :

```cpp
IO::deserialize(fileName, firstRouteSegmentOfStop, firstStopIdOfRoute, firstStopEventOfRoute, routeSegments, stopIds, stopEvents, stopData, routeData, implicitDepartureBufferTimes, implicitArrivalBufferTimes);
```

En deux mots, chaque argument est désérializé en fonction de son type, par une template-specialization de [operator()](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Helpers/IO/Serialization.h#L209) et de [deserialize](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Helpers/IO/Serialization.h#L244). Par exemple, `firstRouteSegmentOfStop` est un `std::vector<size_t>`, et sera déserializé en tant que tel.

Note : [byteSize](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L400) ne sert pas à la (dé)serialization, mais juste à indiquer la taille prise par la structure sur le disque, dans les statistiques.

## Format des departure_time/arrival_time

Dans le code, les `departure_time` sont des `int` (ex: [le paramètre d'entrée de RAPTOR](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L79)).

Le plus probable est que cet `int` code l'heure du jour en nombre de secondes. C'est plutôt confirmé par [ce code](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Runnables/RunRAPTORQueries.cpp#L138) qui construit des queries :

```cpp
const int departureTime = (
    (rand() % (16 * 60 * 60))
    + (5 * 60 * 60)
);
```

En gros, on prend une heure random entre 00h00 et 16h00 (par exemple, 03h19m57s), et on lui ajoute 5h. Du coup, ce code définit une heure random entre 05h00 et 21h00.

## Fonctionnement de l'algo RAPTOR utilisant les shortcuts ULTRA

(notes vrac)

### arrivalByRoute / arrivalByTransfer

Les méthodes [arrivalByRoute](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L279) et [arrivalByTransfer](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L290) sont des setters qui mettent à jour la meilleure heure d'arrivée à un stop donné (soit par un TC, soit par un transfert piéton). Du coup, elles gagneraient à être renommées `setArrivalByRoute` ou `setArrivalByTransfer` :

```cpp
inline bool arrivalByRoute(const StopId stop, const int time) noexcept { ...
inline bool arrivalByTransfer(const StopId stop, const int time) noexcept { ...
```

### EarliestArrivalLabel

Pour un stop donné, comment sont stockées les informations sur sa meilleure heure d'arrivée, et le moyen de le rejoindre ?

La struct [EarliestArrivalLabel](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L45) stocke ces infos :

```cpp
struct EarliestArrivalLabel {
    EarliestArrivalLabel() : arrivalTime(never), parentDepartureTime(never), parent(noVertex), usesRoute(false), routeId(noRouteId) {}
    int arrivalTime;
    int parentDepartureTime;
    Vertex parent;
    bool usesRoute;
    union {
        RouteId routeId;
        Edge transferId;
    };
};
```

Au sein de l'algo, pour un round donné, [un vector](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L56) permet de mapper un stopId donné à son EarliestArrivalLabel :

```cpp
using Round = std::vector<EarliestArrivalLabel>;
```

Et [chaque round a son propre vector](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L305) (ce qui permet à des stops d'être améliorés à chaque round) :

```cpp
std::vector<Round> rounds;

// [...]
inline Round& currentRound() noexcept {
    return rounds.back();
}

// [...]
inline Round& previousRound() noexcept {
    return rounds[rounds.size() - 2];
}
```

Je m'attendrais à ce que le currentRound contiennent les meilleurs temps de chaque stop (et donc que pour les stops non mis à jour, leurs EarliestArrivalLabel soit simplement copié de round en round), mais je ne sais pas encore si c'est le cas, il se peut que le round courant ne contienne que les infos des stops mis à jour. 

**EDIT** : je pense que c'est cette dernière phrase qui est correcte :

```
SOURCE = 435
TARGET = 120
DEPTIME= 36000

Le stop SOURCE 435 a pour data = Stop{Vernier, Croisette, (6.0927, 46.2191), 120}
Le stop TARGET 120 a pour data = Stop{Genève, Aéroport (TPG), (6.11013, 46.2309), 120}

Il y a eu : 4 rounds qui ont tourné
=== ROUND 0
         arrivalTime = 37626
         parentDepartureTime = 36000
         parent = 435
         usesRoute = false
         UNION> transferId = Invalid
=== ROUND 1
         arrivalTime = 36902
         parentDepartureTime = 36481
         parent = 114
         usesRoute = true
         UNION> routeId = 229
=== ROUND 2
         arrivalTime = 2147483647
         parentDepartureTime = 2147483647
         parent = Invalid
         usesRoute = false
         UNION> transferId = Invalid
=== ROUND 3
         arrivalTime = 2147483647
         parentDepartureTime = 2147483647
         parent = Invalid
         usesRoute = false
         UNION> transferId = Invalid
```

Mon interprétation de ceci :

- note : j'ai choisi les stops car ils sont sur la même route
- lors du premier round, on atteinte le stop cible `120` en marchant directement à pied, en 1626 secondes (à titre indicatif, c'est cohérent avec [le trajet proposé par Google Maps](https://www.google.fr/maps/dir/46.2191,6.0927/'46.2309,6.11013'/@46.2247454,6.0971196,16z/data=!3m1!4b1!4m7!4m6!1m0!1m3!2m2!1d6.11013!2d46.2309!3e2), qui propose 26 ou 27 minutes, soit `1560` ou `1620` secondes).
- lors du second round, on améliore l' `arrivalTime` qui descend à 36902 (soit `902` secondes = 15 minutes de trajet), en empruntant la route n°229.
- aucun des rounds suivants n'améliore l'EAT du stop, donc le stop "n'apparaît pas" dans les rounds suivants
- note : tant qu'il reste des stops mis à jour, les rounds continuent, ce qui explique qu'on a des rounds supplémentaires alors qu'on avait trouvé le meilleur EAT dès le deuxième round
 
### Reconstruction du chemin complet

Pour le moment, je sais définir à quelle heure et comment j'arrive à un stop donné (il suffit de retrouver son `EarliestArrivalLabel` dans le dernier `Round` où l'info est définie).

Si j'ai demandé à arriver pile sur un stop, [l'algo maintient](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L124) un `targetStop`, qui suffit à répondre au problème donné = reconstruire la meilleure route.

Ça reste vrai même si on part d'un vertex quelconque (qui n'est pas un stop), puisque si on arrive sur un stop, en reconstituant le chemin à rebours, on finira par arriver à une instance d' `EarliestArrivalLabel` qui utilise éventuellement un chemin piéton initial pour rejoindre le premier stop.

Dans le cas général où **j'arrive** sur un vertex quelconque, je ne sais pas encore où/comment retrouver l'info qui intègre le transfert à pied final.

### Données d'entrée

L'algo prend en entrée le binaire RAPTOR (constitué des données TC et d'un graphe de transferts), et des données bucket CH :

```
/path/to/ultra-server \
    WORKDIR/COMPUTE_SHORTCUTS_OUTPUT/ultra_shortcuts.binary \
    WORKDIR/BUILD_BUCKETCH_OUTPUT/bucketch.graph
```

Le binaire RAPTOR utilisé est bit-à-bit identique au binaire d'entrée du preprocess. En revanche, le graphe de transferts qui l'accompagne est considérablement raccourci, et ne contient plus que les shortcuts ULTRA :

```
md5sum WORKDIR/INPUT_DATA/raptor.binary WORKDIR/COMPUTE_SHORTCUTS_OUTPUT/ultra_shortcuts.binary
738c0a0a4c8e1d4dadc27bd349a3c3c2  WORKDIR/INPUT_DATA/raptor.binary
738c0a0a4c8e1d4dadc27bd349a3c3c2  WORKDIR/COMPUTE_SHORTCUTS_OUTPUT/ultra_shortcuts.binary

diff -y WORKDIR/INPUT_DATA/raptor.binary.graph.statistics.txt WORKDIR/COMPUTE_SHORTCUTS_OUTPUT/ultra_shortcuts.binary.graph.statistics.txt

StaticGraph with 604,167 vertices and 1,847,140 edges (24.44M | StaticGraph with 25,426 vertices and 111,210 edges (1.29MB on
[...]
                  #Vertices :            604,167  (100.00%)   |                   #Vertices :             25,426  (100.00%)
[...]
                     #Edges :          1,847,140  (100.00%)   |                      #Edges :            111,210  (100.00%)
[...]
              graphDistance :        295,739.999 km           |               graphDistance :      1,710,046.932 km
[...]
```

En plus des données, le run de l'algo utilise 3 inputs :
- id du stop de départ
- id du stop d'arrivée
- `departure_time`

```cpp
RAPTOR::Data data = RAPTOR::Data::FromBinary(raptorFile);
data.useImplicitDepartureBufferTimes();
CH::CH bucketCH(bucketChBasename);

using ShortcutRAPTOR = RAPTOR::ULTRARAPTOR<RAPTOR::NoDebugger>;
ShortcutRAPTOR algorithm(data, bucketCH);

int SOURCE = 435;
int TARGET = 120;
int DEPARTURE_TIME = 36000;
algorithm.run(Vertex(SOURCE), DEPARTURE_TIME, Vertex(TARGET));
```

### Données de sortie

L'algo ne renvoie rien, il s'arrête (comme tout RAPTOR) quand il n'y a plus de stops améliorés par le round précédent.

Du coup, il faut reconstruire l'itinéraire, j'ai décrit plus haut comment faire ça quand on faisait un trajet depuis un vertex quelconque vers un vertex qui est un stop.

À noter que les stops sont les premiers vertex du graphe, car [c'est à ça qu'on les reconnaît](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/RAPTOR/Data.h#L113) :

```cpp
inline bool isStop(const Vertex stop) const noexcept {return stop < numberOfStops();}
```

Reste à voir comment s'inscrivent les transferts bucket CH dans tout ça.

### Quelles structures gèrent les transferts dans l'algo = baseQuery

TL;DR : 
- `baseQuery` = c'est une query CH classique, avec le paramètre `CollectPOI` à `true`
- cette query calcule les distances entre un vertex source (resp. un vertex target) et tous les autres vertex du graphe
- avec `CollectPOI`, au cours du dijkstra, on mémorise en plus pour chaque vertex de type POI s'il a été atteint ou pas dans `reachedPOIs`
- dans notre contexte, les POI en question sont les stops TC (qui se trouvent être les tout premiers vertex du graphe — en indice)

L'algo [a un membre](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L303) `initialTransfers`, qui gère à la fois les transferts piétons en cours d'iti TC, et le bucketCH au départ/arrivée :

```cpp
BucketCHInitialTransfers initialTransfers;
```

Si on remonte un peu la chaîne ([lien1](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/InitialTransfers.h#L38) -> [lien2](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L66)), `initialTransfers` est une instance de `BucketQuery`.

Pas encore limpide = le rôle de `StallOnDemand`, passé à `true` dans la définition du type.

Cette instance de `BucketQuery` [semble s'appuyer sur une baseQuery CH "classique"](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L236), dont [le template-parameter](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L42) `COLLECT_POIS` est à `true` :

```cpp
using BaseQuery = Query<Graph, StallOnDemand, false, true>;
```

Le run de BucketQuery ne fait rien de plus que :
- [appeler run de la baseQuery](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L79)
- [collectPOIs forward, puis backward](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L81)

#### rôle de la baseQuery

C'est une instance de CH::Query, donc d'une query classique : dijkstra + contraction-hierarchie, l'élément "nouveau" (utilisé par BucketCH derrière) est la collecte des POIs.
En gros, si le template-parameter `CollectPOIs` est à `true` (ce qui est le cas), [le dijkstra mémorise pour chaque des N premiers vertex s'ils ont été atteints ou pas](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/CHQuery.h#L354), dans `reachedPOIs` :

```cpp
if constexpr (CollectPOIs) {
    if (u < endOfPOIs) {
        reachedPOIs[I].emplace_back(u);
    }
}
```

Le `N` en question est appelé `endOfPOIs`, et passé [à la construction de baseQuery](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/CHQuery.h#L56). Cette construction est [faite par BucketQuery](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L57), et ULTRARAPTOR passe le nombre de stops comme `endOfPOIs` : [lien](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L62).

À VÉRIFIER : normalement, sur un graphe piéton connexe, TOUS les stosp sont joignables à la fois en forward et en backward, et reachedPOI doit donc contenir autant d'éléments que de POIs/stops.

### Bucket CH

À ce stade, j'ai une queryCH classique, qui a calculé la distance entre le vertex source (resp. target) et chaque vertex du graphe. De plus, certains vertex spéciaux (les POIs = les stops) sont marqués comme étant ou non atteignables depuis la source (resp. la target).


### Apport de bucket CH par rapport à CH classique

Le run sur BucketQuery [est simplement](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L79) un run de la Query CH "classique" suivi de 2 x collectPOI :

```cpp
baseQuery.template run<TARGET_PRUNING>(from, to);
collectPOIs<FORWARD>();
collectPOIs<BACKWARD>();
```

Pas clair = stall on demand ?

L'apport de bucketCH semble être :
- d'une part le `collectPOI` [lien](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L217), 
- d'autre part le `bucketGraph`

Que fait buildBucketGraph : https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L186

Que fait collectPOI : https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L217

#### buildBucketGraph

C'est quoi le bucketGraph ?


Déjà, il est construit à partir d'un [CHConstructionGraph](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L188), qui est [est un EdgeList](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Graph.h#L40) sans VertexProperty, et pour lesquels les edges ont 4 propriétés :
- `weight`
- `via`
- `from` et `to` (via [EdgeListImplementation](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/GraphInterface.h#L123))

Et un `EdgeList`, c'est une simple [liste de vertex et d'attributes](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/EdgeList.h#L752). Au passge, il existe [une méthode pour construire un tel graphe depuis un dimacs](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/DataStructures/Graph/Classes/EdgeList.h#L510).

Les vertices dans le CHConstructionGraph [sont ceux](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L188) de `distance` -> il faut voir ce qu'il y a là-dedans (possiblement, c'est le résultat de la baseQuery classique ?)

#### Point de détail = syntaxe que je ne connaissais pas

Le code de BucketQuery [utilise une syntaxe](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/CH/Query/BucketQuery.h#L79) `baseQuery.template` que je ne connaissais pas :

```cpp
baseQuery.template run<TARGET_PRUNING>(from, to);
```

C'est documenté à la section _The template disambiguator for dependent names_ de [la doc sur les dependend_name](https://en.cppreference.com/w/cpp/language/dependent_name).

TL;DR : c'est juste une question de syntaxe C++ (pour désambiguer le parsing)

### Utilisation de initialTransfers par ULTRARAPTOR

À l'initialize, on calcule les transferts [en appelant](https://github.com/kit-algo/ULTRA/blob/0d8680f00c7ad0a74e02c60bc6be77cf93cc62bb/Algorithms/RAPTOR/ULTRARAPTOR.h#L197) le `run` de bucketCH sur le sourceVertex + targetVertex :

```cpp
initialTransfers.run(sourceVertex, targetVertex);
```

TODO : décrire comment ULTRARAPTOR utilise les initialTransfers

### In fine, comment reconstruire le chemin

Mon sentiment actuel = pour faire un trajet de vertex quelconque à vertex quelconque (plutôt que stop à stop) :
- une fois l'algo terminé, on connaît l'EAT de chaque stop
- on itère sur chaque stop, et on retient celui qui minimise **heure d'arrivée au STOP + distance entre le STOP et le TargetVertex**
- derrière, on reconstruit le trajet à rebours depuis ce stop, jusqu'à arriver sur le vertex de départ

Note : le fait que l'output de l'algo ne soit pas un chemin vers le vertex target, mais juste la mise à jour de l'EAT de tous les stops serait plutôt confirmé par le fait qu'en dehors 1. du calcul du bucketCH, et 2. du cas où le targetVertex est un stop, l'algo n'utilise pas le `targetVertex`...

Note : l'itération sur les stops peut sans-doute être optimisée (e.g. commencer par classer les stops par heure d'arrivée + arrêter l'itération quand l'EAT des stops dépasse le meilleur EAT+footpath , ou encore itérer sur les stops par proximité avec le vertex target).