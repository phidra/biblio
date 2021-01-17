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
