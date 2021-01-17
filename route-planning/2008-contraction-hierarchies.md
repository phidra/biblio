# (ARTICLE) Contraction Hierarchies: Faster and Simpler Hierarchical Routing in Road Networks

- **url** = [article simplifié](http://algo2.iti.kit.edu/schultes/hwy/contract.pdf) (md5sum=`9dfe56278407459bb0e2d9a781bcf8f4`, [copie locale](./LOCALCOPIES/contract.pdf)) / [thèse](https://algo2.iti.kit.edu/documents/routeplanning/geisberger_dipl.pdf) (md5sum=`b31efbba33a3bc527523eed6b8722a2f`, [copie locale](./LOCALCOPIES/geisberger_dipl.pdf))
- **source** = [WEA 2008](https://dl.acm.org/doi/proceedings/10.5555/1788888)
- **auteurs** = Robert GEISBERGER (dont c'est la thèse), Peter SANDERS, Dominik SCHULTES, and Daniel DELLING / KIT
- **date de publication** = 2008
- **date de rédaction initiale de ces notes** = juin 2020


## Notes synthétique

Le travail de synthèse reste à faire : je me suis contenté de déplacer mes notes vrac de l'éqpoque à cet emplacement.

## Notes vrac

### NdM

NdM : quand on cherche à contracter un noeud v
- on va supprimer le noeud v du graphe
- du coup, il faut ajouter des raccourcis partout où v est sur un plus court chemin (PCC)
- si le plus court chemin entre le noeud u et le noeud w passe par v, il faut rajouter un raccourci de (u;w)
- moi : inutile de regarder pour tous les noeuds du graphe : il suffit de regarder s'il y a des plus courts chemins entre deux noeuds VOISINS de v !
- (en effet, si un PCC de u à w passe par v, alors il passera forcément par deux voisins de v)
- le hic : comme à ce stade on travaille sur l'overlay graph (auquel il manque des noeuds, mais qui contient des edges supplémentaires) alors les "voisins" de v peuvent être très TRÈS nombreux, et très éloigné les uns des autres !
- il peut donc être compliqué de rechercher les plus courts chemins entre tous les voisins de v :-(

NdM = pourquoi CH accélère le routing ?
- parce qu'on ne peut aller QUE vers des noeuds d'ordre supérieur
- à chaque nouveau noeud relaxé dans un dijkstra unidirectionnel (peu importe le sens), on restreint le nombre de noeud qu'on peut explorer
- on les restreint même de plus en plus au fur et à mesure qu'on avance dans l'algo :
    + supposons que le graphe ait 10000 nodes
    + alors lorsqu'un node relaxé a l'order 9500, il ne nous reste plus que 500  (au lieu de 10000) noeuds possibles à explorer

NdM = pourquoi est-ce important de répartir la contraction uniformément ?
- car (en reprenant l'exemple juste au dessus), les 500 nodes restants sont tous dans mon voisinage, il faudra que je les explore tous
- si à l'inverse ils sont répartis uniformément sur le graphe et que du coup je n'en ai que 20 dans mon voisinage, alors je n'aurais que 20 noeuds (au lieu de 500) à explorer pour terminer l'exploration de cette branche du graphe
- Dit autrement : la répartition des noeuds du graphe permet d'élaguer rapidement des branches du graphe qu'on n'explorera pas.
- Une autre façon de voir les choses :
    + le meeting-node où se rejoignent les deux dijkstra unidirectionnels sera le node d'ordre le plus élevé sur le chemin
    + si l'ordering a regroupé TOUS les nodes d'ordre élevés à un endroit (par exemple au nord)
    + alors on est sûr que le meeting point se trouvera au nord
    + et si on veut faire un trajet du sud au nord, le dijkstra qui part du sud devra explorer toute la France pour trouver son meeting-point au nord

Après seconde lecture de l'article, ce qui ne reste pas clair :
- différence entre limiter le searchSpaceSize et limiter le nombre de hops
- implémentation du dijkstra pour choisir si on ajoute un shortcut, notamment comment utiliser la limite max
- le stall-on-demand

Sans doute que la lecture de la thèse aiderait...

Au sujet de la difficulté à trouver un bon ordering, la [thèse WCH](https://i11www.iti.kit.edu/_media/teaching/theses/weak_ch_work-1.pdf) donne des références sur le sujet :
- But finding an order which minimizes theamount of added shortcuts is known to beNP-hard [BCK+10]
    + Reinhard Bauer, Tobias Columbus, Bastian Katz, Marcus Krug, and DorotheaWagner.
    + Preprocessing Speed-Up Techniques is Hard.
    + In Proceedings of the 7thConference on Algorithms and Complexity (CIAC’10), volume 6078 ofLectureNotes in Computer Science, pages 359–370. Springer, 2010.

Par ailleurs, concernant le preuve que CH accélère les queries en limitant le search space size, la même thèse donne des références :
- In [BCRW13,Col12], a theoretical framework to study Contraction Hierarchies was developed. It enables proving upper bounds on the search space size for certain classes of graphs
    + Reinhard Bauer, Tobias Columbus, Ignaz Rutter, and Dorothea Wagner.
    + Search-Space Size in Contraction Hierarchies.
    + In Proceedings of the 40th Inter-national Colloquium on Automata, Languages, and Programming (ICALP’13),volume 7965 ofLecture Notes in Computer Science, pages 93–104. Springer,2013.

### Copie de mes notes manuscrites (à trier puis merger avec les autres notes)

Heuristique de contraction :
- edge difference
- spread = contract nodes uniformely

bidirectional dijkstra :
- forward dans G↑
- backward dans G↓
- meeting node = le noeud qui a l'ordre le plus élevé du path trouvé à la query
- (NdM : d'où l'importance de spread uniformément la contraction : si tous les noeuds d'ordre élevés sont au même endroit, ça va pas aller)


#### Chapitre 2 = Contraction :
- "witness" path = un chemin qui témoigne (i.e. qui "prouve") qu'il est inutile d'ajouter le shortcut pour le triplet (u,v,w) considéré
- (en d'autres termes, un witnesse path est un plus court chemin de u à w qui ne passe pas par v)
- pas clair : "hop" ? Semble être un moyen de limiter le coût du dijkstra local = de la vérification de si oui ou non on ajoute un shortcut. hop est adapté dynamiquement, pas clair comment.

#### Chapitre 3 = Node-Ordering :

L'ordering utilise une priority queue sur une combinaison linéaire de différentes facteurs.

Quand le node v est contracté, ça affecte la priorité d'autres nodes :-/

Pour mitiger :
- lazy updates
- recalcul prio des voisins de v
- recalcule régulièrement TOUTES les priorités

La combinaison linéaire pour définir les priorités utilise :
- edge difference
- uniformité pour spread la contraction un peu partout (QUESTION : maximal-independent set = ??)
- deleted neighbours = voisins déjà contractés
- voronoi regions :
    + VR(v) = { u∈V ; u est plus proche de v que de n'importe quel autre noeud }
    + on utilise sqrt(size(VR)) dans la priority function et on contracte les noeuds à petite VR d'abord
- optionnel = cost of contraction
- optionnel = cost of queries (pas clair)
- optionnel = global measure (NdM = FRC7 ?)

#### Chapitre 4 = Query

Stall-on-demand = pas clair

EDIT : dans un autre papier sur TCH, il y a une mention qui peut m'aider à comprendre :

> During the bidirectional search we perform stall-on-demand [4, 2]: The search stops at nodes when we already found a better route coming from a higher level.

Dans [ce papier](http://algo2.iti.kit.edu/documents/routeplanning/tch_alenex09.pdf), j'ai :

> The stall-on-demand technique identifies such nodes w by checking whether(downward) edges coming into w from more important nodes give shorter paths than the (upward) Dijkstra search.

Il y a un chapitre très détaillé sur le Stall-on-demand à la page 219 de [la thèse sur TCH](http://digbib.ubka.uni-karlsruhe.de/volltexte/documents/3569195)

#### Chapitre 6 = Experiments

Éléments de l'heuristique d'ordering :
- E = edge difference
- D = deleted neighbours
- S = search space size (??) ... semble appelé "cost of contraction" ailleurs ?
- W = relative betweenness (??) ... serait appelé "global measures" ailleurs ?
- V = sqrt(voronoi region size)
- L = limit search space on weight calculation (??)
- Q = upper bound on edges in search path (??) ... serait appelé "cost of queries" ailleurs ?

L'average degree explose si on est trop drastique sur les hop-limits
- NdM : mon interprétation = car on rajoute des raccourcis "inutiles" -> on a plus d'edges au final
- NdM : du coup, ça semble confirmer que le nombre d'edges est bien une mesure importante pour quantifier la qualité d'un node ordering

La hop-limit est dynamique au cours de l'ordering : en fonction du degré moyen, on augmente le nombre de hops.

B/node = byte / node = space-size (taille du graph dump)

NdM = pour un nombre de nodes donné, la taille du graphe quantifie aussi la qualité de l'ordering.

#### Chapitre 7 = Conclusions

Goal-directed technique = c'est quoi ? EDIT : algos orientant la recherche de la solution "vers" la cible (e.g. A*)

Il y a également une définition et quelques exemples de techniques) dans [ce papier](https://publikationen.bibliothek.kit.edu/1000014952) :

> Goal-Directed Approaches direct the search towards the target t by preferring edges that shorten the distance to t and by excluding edges that cannot possibly belong to a shortest path to t. Such decisions are usually made by relying on preprocessed data.


### Page 3 - chapitre 2 "contraction"

#### Ma compréhension des notations

- d(u,w) = distance(u,w) = coût du plus court chemin de u à w. (u,w) n'est pas forcément un edge
- c(u,w) = cost(u,w) = poids de l'edge (u,w). (u,w) est forcément un edge.
- Même si (u,w) est un edge, on n'a PAS FORCÉMENT d(u,w) == c(u,w) :
    + sur le graphe original G, il existe peut-être un AUTRE chemin que l'edge (u,w) permettant d'aller de u à w, de façon plus courte
    + sur l'overlay graph G′, si (u,w) est un raccourci, il existe peut-être un autre  chemin que le raccourci pour aller de u à w, de façon plus courte

> Recall from the introduction that when contracting node v, we are dealing with an overlay graph G′=(V′,E′) with V′=v..n and an edge set E′ that preserves shortest path distances wrt the input graph.

Au moment où on contracte le noeud v, l'overlay graph G' :
- contient les noeuds d'ordre v à n (les autres noeuds ont déjà été contractés, donc ne sont plus dans l'overlay graph)
- contient les edges originaux, ainsi que les raccourcis créés lorsqu'on a contractés les noeuds de 1 à (v-1)

> In G′, we face the following many-to-many shortest-path problem: For each source node u ∈ v+1..n with (u,v) ∈ E′and each target node w ∈ v+1..n with (v,w) ∈ E′, we want to compare the shortest-path distance d(u,w) with the shortcut length c(u,v) + c(v,w) in order to decide whether the shortcut is really needed

- considérant deux NOEUDS u et w VOISINS de v tel que u → v → w soit un chemin (en effet, (u,v) ∈ G′ et (v,w) ∈ G′)
- (dit autrement, on s'intéresse à tous les PRÉDÉCESSEURS u de v, et tous les SUCCESSEURS w de v)
- on veut calculer la longueur d(u,w) du plus court chemin PCC(u,w), et la comparer avec c(u,v) + c(v,w) = la longueur du raccourci qu'on ajouterait en supprimant v
- l'idée est de NE PAS ajouter le raccourci (u,w) si c(u,v) + c(v,w) > d(u,w), car ça voudra dire que le PCC(u,w) NE PASSE PAS par v (mais "contourne" v)

> A simple way to implement this is to perform a forward shortest-path search in the current overlay graph G′ from each source, ignoring node v, until all targets have been found

- pour CHAQUE prédécesseur u de v, on calcule un dijkstra pour calculer le plus court chemin de u à N'IMPORTE QUEL noeud de G′
- (en effet, dijkstra peut calculer le plus court chemin d'une source à TOUS les sommets du graphe)
- (note : comme on est dans l'overlay graph, les prédécesseurs de v incluent les prédécesseurs via des raccourcis -> il peut y en avoir beaucoup, et très éloignés de v !)
- ainsi, on calculera toute les distances de u à tous les noeuds de G′ et notamment à tous les w, donc on aura calculé d(u,w), ce qui nous intéresse

> We can also stop the search from u when it has reached distance d(u,v) + max{c(v,w) : (v,w) ∈ E′}
- lorsqu'on cherche le PCC de u à w (mais ne passant pas par v), même si on en trouve un...
- ...si le PCC trouvé a un coût supérieur à d(u,v) + max{cost(v, w)}...
- ... alors ce PCC NE NOUS SERVIRA PAS à exclure le raccourci passant par v (car on est alors sûr que le PCC trouvé sera plus LONG que le raccourci)

> Our actual implementation uses a simple, asymmetric form of bidirectional search inspired by [10]: For each target node w we perform a single-hop backward search. For each edge (x,w) ∈ E′ we store a bucket entry (c(x,w), w) with node x. 

- pour tous les PRÉDÉCESSEURS de w, on stocke une entrée dans un bucket (hashtable?) avec le coût de l'edge du prédécesseur à w

> This way, forward search from u can be limited to distance :

```
c(u,v)+ max{c(v,w)} − min{c(x,w)}
        w:(v,w)∈E′    x:(x,w)∈E′
```
L'objectif semble être de prune au plus vite le dijkstra pour pas qu'il nous coûte trop cher.  On semble additionner :
- le coût de l'edge d'entrée (u,v)
- le coût MAXIMAL de l'edge de sortie (v,w)
Auquel on retranche : le coût du plus petit prédécesseur de w (QUESTIOn : même s'il n'est pas un successeur de u ?).

QUESTION : je ne comprends pas bien pourquoi on peut limiter le coût max du dijkstra à ça ?

- Je suppose en préambule qu'il s'agit d'être sûr que tous les PCC au delà de cette valeur seraient plus LONG que c(u,v) + c(v,w)
- ça revient donc à dire qu'en majorant  max{c(v,w)} − min{c(x,w)}  ,  on est sûr de majorer  c(v,w)  :
    + max{c(v,w)} − min{c(x,w)}   est FORCÉMENT supérieur à   c(v,w)
- Je vois 4 cas :
    1.  (v,w) a le coût le plus GRAND des successeurs de v  ET  (v,w) a le coût le plus PETIT des prédécesseurs de w
        si v est le plus petit des successeurs de w, par définition, (v,w) est le moyen le plus rapide de relier v à w
    2.  (v,w) a le coût le plus GRAND des successeurs de v  ET  (v,w) a le coût le plus GRAND des prédécesseurs de w
    3.  (v,w) a le coût le plus PETIT des successeurs de v  ET  (v,w) a le coût le plus PETIT des prédécesseurs de w
    4.  (v,w) a le coût le plus PETIT des successeurs de v  ET  (v,w) a le coût le plus GRAND des prédécesseurs de w

Mmmmh, non, décidément, c'est pas clair...

> When reaching a node x, we scan its bucket entries. For each entry (C,w), we can infer that there is a path from u to w of length d(u,x)+C.

- pour chaque node x atteint dans le dijktra, on sait qu'on peut atteindre w en d(u,x) + c(x,w)
- ce dernier terme c(x,w) est déjà dans la bucket-list

> Since exact shortest path search for contraction can be rather expensive, we have implemented two ways to limit the range of searches: We can limit the number of hops (edges) used in any path〈u,...,w〉, and we can limit the total search space size of a forward search

- mon interprétation de "limiter le total search space size" = on limite le nombre de NOEUDS possibles dans la recherche du plus court chemin (on arrête d'explorer quand on a exploré Nmax noeuds)
- mon interprétation de "limiter le nombre de hops" = on limite le nombre d'EDGES possibles dans un plus court chemin (on arrête d'explorer quand un chemin contient Emax edges)
- QUESTION = mais tel que l'article le présente, limiter le nombre de hops = limiter le nombre d'edges dans le chemin ? Du coup quelle différence avec limiter le search space ? Dans le code RE, on dirait que nombre de hops == search space size...
- Dans le code RE :
    + le dijkstra est limité par un coût max "cutoff"
    + numHops est le nombre d'edge (possiblement contractés, si je dis pas de bêtises) entre le noeud source et le noeud actuellement exploré
    + le searchSpace est le nombre max d'hops effectués par le dijkstra
    + le maxSearchSpace d'un node v en cours de contraction est le searchSpace maximal sur tous les chemins calculés entre u=un prédécesseur de v et w=un successeur de v
    + pour un node en cours de contraction donné, le maxSearchSpace est utilisé pour calculer l'heuristique d'ordering (s'il est grand, le noeud aura un "coût" d'ordering un peu plus grand) ; au même titre que l'edgeDiff, qui compte d'ailleurs 100 fois plus que le maxSearchSpace

Apparemment, leur implémentation "hop" s'inspire [de ça](http://algo2.iti.kit.edu/documents/routeplanning/distTable.pdf) (à regarder ?)
