# (ARTICLE) Computing many-to-many shortest paths using highway hierarchies

- **url** = [article](https://dl.acm.org/doi/10.5555/2791188.2791192) (md5sum=`da325cd2551d0b9e523812626d40d74c`, [copie locale](./LOCALCOPIES/2791188.2791192.pdf))
- **source** = [ALENEX 2007](https://archive.siam.org/meetings/alenex07/) ([proceedings](https://epubs.siam.org/doi/book/10.1137/1.9781611972870))
- **auteurs** = Sebastian KNOPP, Peter SANDERS (co-auteur du [papier original des CH](./2008-contraction-hierarchies.md), et du [survey de 2015](./2015-route-planning-in-transportation-networks.md)), Dominik SCHULTES, Frank SCHULZ and Dorothea WAGNER (co-autrice du [survey de 2015](./2015-route-planning-in-transportation-networks.md), de [ULTRA](./2019-ULTRA.md), ou encore de [CCH](./2014-customizable-contraction-hierarchies.md)).
- **date de publication** = 2007
- **date de rédaction initiale de ces notes** = octobre 2021

**TL;DR** :

- le contexte de ces notes est important :
    - je m'intéresse avant tout à la possibilité de faire des requêtes one-to-many avec les CH, j'aborde l'ensemble de l'article (qui concerne Highway Hierarchy, et non CH) avec ce besoin en tête.
    - plus précisément, le présent article est cité par [ULTRA](./2019-ULTRA.md) comme l'une des trois sources de leur implémentation de Bucket-CH, qui leur sert à faire du one-to-many efficace.
    - mieux : les deux autres articles cités par ULTRA comme sources de Bucket-CH sont [l'article original des CH](./2008-contraction-hierarchies.md), et [un article dérivé présentant les CH lui-aussi](./2012-exact-routing-in-large-road-networks-using-ch.md)... Or ces deux articles référencent _eux aussi_ le présent article comme source de leur implémentation many-to-many !
    - tout concourt donc à dire que l'implémentation par ULTRA de Bucket-CH pour faire du one-to-many efficace est décrite dans le présent article
- concernant l'algo many-to-many décrit ici, je l'ai déjà décrit dans mes notes sur [l'article original des CH](./2008-contraction-hierarchies.md) :
    - attribuer à chaque noeud du graphe un bucket qui stocke `|T|` distances depuis chaque target-node (initialisé à `+∞` qui veut dire "le noeud n'a pas été atteint par la backward-search de ce target-node")
    - faire `|T|` backward-searches depuis chacun des `|T|` target-nodes, ce qui remplit l'item approprié du bucket de chaque noeud reached
    - faire `|S|` forward-searches, et pour chacune, utiliser le bucket de chaque node reached 
- donc en résumé, l'idée est de "mutualiser" les backward-searches (et les mémorisant dans les buckets), pour qu'elles soient réutilisées par chaque forward-search
- il y a quelques différences entre l'implémentation CH et HH. Notamment, un point qui différencie CH et HH est le choix d'un paramètre `K` pour arrêter la backward-search (alors qu'avec CH, il faut explorer à fond la propagation pour garantir que les PCC pourront être trouvés).
- la section 6.2 donne une optimisation qui semble applicable à Bucket-CH également

----


* [(ARTICLE) Computing many-to-many shortest paths using highway hierarchies](#article-computing-many-to-many-shortest-paths-using-highway-hierarchies)
   * [Abstract](#abstract)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = Highway Hierarchies](#section-2--highway-hierarchies)
   * [Section 3 = the algorithm](#section-3--the-algorithm)
      * [Section 3.1 refinement = fewer bucket entries](#section-31-refinement--fewer-bucket-entries)
      * [Section 3.1 refinement = accurate backward search](#section-31-refinement--accurate-backward-search)
      * [Section 3.2 analysis](#section-32-analysis)
   * [Section 4 = implementation](#section-4--implementation)
   * [Section 5 = experiments](#section-5--experiments)
   * [Section 6 = generalizations](#section-6--generalizations)
      * [Section 6.1 = outputting paths](#section-61--outputting-paths)
      * [Section 6.2 = computing shortest connections incrementally](#section-62--computing-shortest-connections-incrementally)
      * [Section 6.3 = parallelization](#section-63--parallelization)
      * [Section 6.4 = precomputed cluster distances](#section-64--precomputed-cluster-distances)
   * [Section 7 = conclusions](#section-7--conclusions)

## Abstract

Objectif = many-to-many entre plusieurs sources `s ∈ S` et plusieurs targest `t ∈ T`.

S'appuie sur Highway Hierarchies (je ne connais pas, mais je sais que c'est le précurseur des CH, et que le même principe de base est utilisé = exploiter la hiérarchie du graphe).

À la date de rédaction de l'article, les HH étaient l'une des techniques les plus rapides.

À l'époque, calculer avec `|S| = |T| = 10.000` (soit 100 millions d'itinéraires) prend environ 1 minute.

Idée de base + quelques raffinements + implémentation minutieuse.

Pallalélisation possible.

Possibilité de se limiter aux `k` plus petits chemins.

## Section 1 = introduction

Taille typique des graphes (Wester Europe et America) qu'ils ont utilisés = 20M nodes.

Ce qui ne marche pas 1 = faire `|S|` propagations Dijkstra : dans la mesure où la version de base de Dijkstra calcule le PCC depuis une source `s` vers **tous** les nodes du graphe, pour calculer les `S x T` itinéraires, il "suffit" de faire `|S|` propagation Dijkstra, soit 10000 propagations Dijkstra. À l'époque, sur ce type de graphe, chaque Dijkstra prend environ 10s, ce qui fait donc plus de 27h pour calculer tous les `S x T` itis.

Ce qui ne marche pas 2 = faire `|S x T|` calculs d'itis utilisant Highway Hierarchies : chaque calcul d'iti est très rapide (1 ms), mais comme il faut en faire 100 millions, on se retrouve également avec un temps total de 27 heures...

A contrario, l'implémentation qui va être décrite dans le papier ne prend que 67 secondes.

Aha ! La section 2 va donner les outlines des Highway Hierarchies, ça m'intéresse :-)

Related work : comme le papier est assez ancien (2007) est que les CH n'existaient pas encore, il est probable que le related work ne soit pas très intéressant.

Un point intéressant = les goal directed techniques ne marchent pas bien avec du multi-target (vu qu'on n'a plus qu'une seule target vers laquelle se diriger, mais plusieurs).

À noter que le many-to-many peut être utile pour le preprocessing de transit-node routing (vu que TNR a besoin de calculer les PCC entre tous les transit-nodes)

## Section 2 = Highway Hierarchies

> The basic idea of the highway hierarchies approach is that outside some local areas around the source and the target node, only a subset of ‘important’ edges has to be considered in order to be able to find the shortest path.

C'est une autre formulation de la notion de hiérarchie : certains edges (les edges locaux) ne sont utiles que pour peu de PCC. D'autres edges (les "important edges") sont utiles pour plus de PCC.

À la différence des CH, où chaque noeud représente un level, ici, on a seulement deux levels = les edges locaux, et les edges importants. EDIT : ah non, on a plusieurs levels car on a plusieurs highway-networks en fait.

Concept de _local-area_ formalisé par le **neighborhood** d'un node `v` : `N(v)` ; chaque noeud `v` a son propre neighborhood.

À l'inverse, le **highway network** est un sous-graphe du graphe routier, tel que tous les PCC sont préservés entre tous les nodes de ce sous-graphe.

NdM : intuitivement : si je ne garde que les edges correspondants aux autoroutes de France, les PCC entre tous ces edges n'auront jamais besoin de passer par des petites routes résidentielles dans des villes. Du coup, les edges des autoroutes sont sur ce _highway network_ (d'où le nom), alors que les edges des routes résidentielles n'en font pas partie.

Plus formellement, un edge est sur le _highway network_ s'il est sur un PCC entre `u` et `v` mais qu'il n'appartient ni au _neighborhood_ de `u`, ni à celui de `v`.

Derrière, il y a une étape de contraction de nodes selon un _bypassability criterion_ (et si le critère est rempli, on remplace le triplet `<u, v, w>` par un shortcut `<u, w>`). Ce critère dépend du degré du node, et du nombre de shortcuts qui seraient créés si on "bypassait" le node. NdM : ça ressemble donc beaucoup à la contraction des CH.

On a plusieurs levels dans la highway-hierarchy : le level 0 contient tous les nodes : c'est le graphe `G` lui-même. Le level 1 est le highway-network du level 0 (pour rappel, chaque highway network correspond à un subset de noeuds dans lequel les PCC sont préservés). Le level 2 est le highway-network du level 1, etc.

Highway query algorithm = modified bidirectional search.

Notion de node particulier = _entrance point_ : grâce à ce node, on passe au level `l`. On dirait qu'on ne relaxe que les edges dans le current-level (et tout comme pour les CH, c'est ça qui fait qu'on accélère les queries : on relaxe beaucoup moins d'edges / on reache+settle beaucoup moins de nodes).

Tout comme CH, le stopping criterion du bidirectional dijkstra doit être modifié : il faut que CHAQUE sens de propagation soit devenu plus grand que le plus court chemin COMPLET pour arrêter le Dijkstra.

## Section 3 = the algorithm

Ils supposent qu'il y a plus de target que de source : `|T| ≥ |S|`, et l'algo est à "inverser" sinon.

Comme le one-to-many pourra être vu comme un cas particulier du many-to-many dont il est question dans l'article, c'est un point à garder en tête.

Ils partent de la solution naïve faisant `|S| x |T|` queries.

Ils limitent le backward search à un level K : quand la backward search atteint le level K, on l'arrête (et c'est la forward-search qui rejoindra les noeuds settled par la backward search).

De plus, pour chaque target, ils ne font qu'une seule backward-search, et stockent les search-spaces de chaque target (elles sont petites, vu qu'on a limité à K levels).

Derrière, pour chaque source, ils ne font qu'une seule forward-search : chaque forward-search utilise les infos des backward-searches.

NdM : ce que je trouve bizarre, c'est qu'ils commencent par faire les searches du côté le plus volumineux `|T|`, alors que j'aurais attendu le contraire. Je pense que j'attends le contraire car j'ai une vision `CH` où les deux propagations forward et backward ont le même coût. Or, là, avec les Highway Hierarchies, on dirait que c'est dissymétrique, et que la propagation backward est moins coûteuse que la forward. Dans ce cas, ça fait sens qu'on applique le moins coûteux au plus volumineux.

Comme attendu, chaque node "`v` du graphe est pourvu d'un bucket. Chaque propagation backward depuis le node `t` remplit le bucket des `v` rencontrés avec un doublet `{t, d}` (qui indique que pour rejoindre `v` depuis `t`, on a besoin d'une distance `t`).

Derrière, lorsqu'elles atteignent un node `v`, chaque propagation forward depuis `s` peut utiliser les différents `{t, d}` en `v` (qui est alors un meeting-node) pour calculer la tentative-distance du cemin de `s` à `t` passant par `v`.

Une citation importante issue de la section 6.2 qui aide à comprendre l'algo :

> the (small) backwards search is done for all t ∈ T until all entrance points to level K are encountered
>
> The (large) forward searches that require heavy scanning of buckets are only progressing incrementally after their search frontier is completely in the core of level K

### Section 3.1 refinement = fewer bucket entries

Le point important = les buckets qui nous intéressent sont ceux des nodes `v` qui sont 1. atteints par une backward propagation depuis `t` et 2. atteints par une forward propagation depuis `s`. Tout bucket additionnel est "du temps perdu". ... Bon, j'ai pas bien compris cette notion, qui semble très liée aux Highway Hierarchies (alors que j'aborde le papier du point de vue des CH, plutôt). On dirait qu'on n'ajoute pas de bucket si le noeud du bucket (`v`) est dans le même level que le chemin jusqu'à `t`. Côté équivalent CH, j'ai l'impression que c'est naturellement géré par le fait que la backward-search respecte la contrainte de propagation CH (à savoir grimper les ranks).

### Section 3.1 refinement = accurate backward search

Ici aussi je n'ai pas bien compris la notion, qui semble liée aux Highway Hierarchies (alors que j'aborde le papier du point de vue des CH, plutôt).

### Section 3.2 analysis

Analyse de la complexité asymptotique (je n'ai fait que survoler, car ça concerne surtout les HH, et il faudrait adapter en généralisant pour les CH).

Il y a un paragraphe dédié à comment choisir le paramètre `K` (qui me paraît inutile pour les CH, ici aussi).

## Section 4 = implementation

Courte section avec quelques détails d'implémentation.

## Section 5 = experiments

NdM : attention que le papier est ancien, et que les valeurs absolues des perfs ne sont plus représentatives de ce qui pourrait se faire aujourd'hui.

Western Europe = 18M nodes + 42M edges / North Ameria = 18M nodes +  47M edges.

Le preprocess (la construction de la highway hierarchy) sur Western Europe prend environ 15 minutes.

Je me contente de skimmer la section.

## Section 6 = generalizations

### Section 6.1 = outputting paths

Même principe que pour CH et sa phase de shortcut-expansion = à la base, on n'a que la plus courte _distance_, et il faut du travail supplémentaire (notamment une structure de données supplémentaire) pour avoir le plus court chemin.

### Section 6.2 = computing shortest connections incrementally

Cette sous-section est intéressante, et si je ne me trompe pas, elle est transposable aux CH.

L'idée est que chaque backward-search maintient un compteur des entrace-node de level K atteints (si un entrance-node de level K est atteint par la backward-search, on incrémente son compteur). Derrière, pour une forward-search donnée, lorsqu'elle atteint un entrance-node, elle décrémente son compteur. Une fois que le compteur atteinte 0, on est sûr que le trajet entre cette forward-search et la backward-search ayant atteint 0 a été trouvée, et il est inutile de poursuivre.

Transposé aux CH, ça revient à dire :

- lors de chaque backward-search on associe à un noeud target `t` son compteur de nodes atteints
- lors d'une forward-search depuis `s`, on décrémente le compteur pour `t` chaque fois qu'on atteint un node dont le bucket contient une entrée pour `t`
- lorsque le compteur est à zéro, on peut arrêter la forward-search depuis `s` pour le `t` particulier, puisqu'on est alors certain que tous les autres noeuds qui nous restent à settle ou à reach n'ont pas été atteints par la backward-search de `t`

NdM = mais comme on va vouloir poursuivre la forward-search pour les autres `t` (ceux dont le compteur n'est pas encore à 0), ça n'a possiblement que deux intérêts :

1. pouvoir arrêter complètement la forward-search depuis `s` un peu plus tôt (dès que TOUS les compteurs des différents `t` atteignent 0)
2. ne pas se casser la tête à continuer à updater des structures (genre des tentative-distances) pour les paires `<s,t>` dont le compteur pour `t` a déjà atteint 0

En gros, ce compteur est un moyen de dire "pour la paire `<s,t>`, j'ai terminé le boulot !".

### Section 6.3 = parallelization

On peut paralléliser les `|T|` backward-searches.

On peut également paralléliser les `|S|` forward-searches.

NdM : mais je pense que le fait d'enchaîner les backward-searches et les forward-searches reste séquentiel : il faut tout de même attendre que toutes les backward-searches soient terminées pour commencer les forward-searches.

### Section 6.4 = precomputed cluster distances

Je saute cette section, qui concerne le partitionnement du graphe en clusters.

## Section 7 = conclusions

**L'idée essentielle qui fait que ça marche bien** = stocker les backward-search-spaces dans des buckets.
