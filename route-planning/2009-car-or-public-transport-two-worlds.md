# (ARTICLE) Car or Public Transport — Two Worlds

- **url** = [PDF](https://ad-publications.cs.uni-freiburg.de/EfficientAlgorithms_Car_Bast_2009.pdf) (md5sum=`26f6c361887fed4f9cb55ec854be9bc6`), [copie locale](LOCALCOPIES/EfficientAlgorithms_Car_Bast_2009.pdf)
- **source** = [Efficient Algorithms](https://link.springer.com/book/10.1007/978-3-642-03456-5) sorti en 2009, qui semble appartenir à la série plus générale [Lecture Notes in Computer Science](https://link.springer.com/bookseries/558)
- **auteurs** = Hannah Bast (future auteure des Transfer Patterns)
- **date de publication** = 2009
- **date de rédaction initiale de ces notes** = 27 octobre 2020

**TL;DR** : article "survey" (n'introduisant pas d'algo nouveau), mais très intéressant, car mets le focus sur les PTN (plutôt que sur les problématiques RN)
- ATTENTION : garder en tête que cet article a été publié en 2009, beaucoup de techniques ont été développées depuis, tant pour RN que pour PTN (p.ex. RAPTOR et Transfer Patterns).
- introduit le formalisme de représentation d'un PTN par un graphe (notamment time-expanded)
- dresse un résumé des tricks actuellement utilisés pour accélérer dijkstra sur un RN, classés en 5 familles :
    - bidirectional search
    - hierarchy (e.g. highway hierarchies, mais n'inclut pas CH)
    - shortcuts / contraction (CH est ici)
    - goal directed (A*, ALT, arcflags)
    - distance tables (transit node)
- pointe du doigt les différences entre RN et PTN qui font que ces tricks qui marchent bien sur RN ne marchent pas sur PTN :
    - différence n°1 = un PTN est déstructuré et n'a pas comme un RN de notion de hiérarchie -> les speed-up techniques qui s'appuient là-dessus ne marchent pas. Exemple : dans un PTN (au moins au niveau local, à l'échelle d'une ville), toutes les lignes de bus sont à peu près équivalentes, alors que sur un RN, les routes locales sont moins importantes que les nationales, elles-mêmes moins importantes que les autoroutes.
    - différence n°2 = sur un RN, avoir source et target proches géographiquement garantit que le search space exploré sera petit. Alors qu'avec un PTN, même deux stations proches géographiquement peuvent être à 15h de trajet l'une de l'autre, ce qui induit un search space important.

* [(ARTICLE) Car or Public Transport — Two Worlds](#article-car-or-public-transport--two-worlds)
   * [section 1 = introduction](#section-1--introduction)
   * [section 2 = models again](#section-2--models-again)
   * [section 3 = tricks of the trade, why and when they work](#section-3--tricks-of-the-trade-why-and-when-they-work)
      * [bidirectional search](#bidirectional-search)
      * [hierarchy](#hierarchy)
      * [shortcuts / contraction](#shortcuts--contraction)
      * [goal direction](#goal-direction)
      * [distance tables](#distance-tables)
   * [Différences entre RN et PTN](#différences-entre-rn-et-ptn)
      * [Différence n°1 = déstructuration](#différence-n1--déstructuration)
      * [différence n°2 = recherche locale difficile](#différence-n2--recherche-locale-difficile)

## section 1 = introduction

- Constat = on ne sait pas calculer des itis aussi efficacement sur PTN que sur RN.
- Introduction à la modélisation par un graphe (dans les deux cas) :
    - RN : node = jonction ; poids d'un arc = temps de trajet entre jonctions
    - PTN (time-expanded) : node = departure/arrival event à une station ; poids d'un arc = temps d'attente ou temps de trajet
- avec ces modélisations par graphe, trouver le meilleur itinéraire revient à calculer le plus court chemin sur un graphe pondéré -> dijkstra
- problème de la taille des graphes PTN :
    - ordre de grandeur des graphes RN sur Western Europe : 20M nodes, 50M arcs
    - sur les PTN, comme on a un event par départ/arrivée, on a ENCORE plus de nodes que sur les RN : 4M events _juste sur le réseau de Berlin_, donc probablement plusieurs centaines de millions de nodes sur Western Europe

L'article donne un overview de Dijkstra dont j'aime beaucoup la formulation :

- chaque node a un **tentative-cost** (initialisé à +∞, sauf pour le source-node qui est initialisé à 0)
- l'algo commence sur le source-node, et alterne entre deux phases distinctes :
- phase 1 = **relaxer les arcs** du noeud courant :
    - visiter tous les outgoing arcs du noeud courant
    - mettre à jour (= diminuer) le tentative-cost de la destination de chaque arc, si ce tentative-cost est amélioré en passant par cet arc
- phase 2 = **settle** le node suivant :
    - parmi les noeuds pas encore settled, on choisit celui avec le tentative-cost le plus faible (d'où la nécessité d'une priority queue)
    - on itère sur ses outgoing edges, et on relaxe chaque arc
- à noter que lorsqu'on choisit un noeud pour relaxer ses arcs, ce noeud est dit **SETTLED** : son cost est définitif et n'évoluera plus, c'est le coût minimal pour rejoindre ce noeud depuis le source-node
- l'algorithme évolue en disque autour du source-node, où la distance utilisée est le tentative-cost de chaque node (c'est en quelque sorte une variante du BFS)
- du point de vue de la complexité, Dijkstra est en O(n.log(n) + m) :
    - `n` est le nombre de node placés dans la priority queue, donc les nodes visités au moins une fois avant d'arrêter l'algorithme (ce qu'on fait quand on settle le target-node)
    - `m` est le nombre d'out-edges de ces nodes (en effet : pour un même node, il faudra possiblement mettre à jour plusieurs fois son tentative-cost, autant de fois qu'il y a d'arcs qui arrivent dessus)
    - la borne supérieure de `n` est le nombre total de nodes dans le graphe (Western Europe = 20M nodes)
    - la borne supérieure de `m` est le nombre total d'arcs dans le graphe (Western Europe = 50M arcs)
    - avec ces valeurs, même si le fait de settle un node ne prend que 100 ns, faire un Dijkstra sur Western Europe prend plusieurs secondes... d'où la nécessité des speedup-techniques décrites dans l'article
    - détail rigolo = on ne sait pas si cette complexité est optimale, ou s'il existe en théorie des algos plus efficaces

## section 2 = models again

NdM : cet article étant ancien, il ne considère que le cas où on interprète les données d'un PTN comme un graphe, ce qui n'est pas le cas de RAPTOR ni de CSA.

Variante du problème pour RN = time-dependent (le poids d'un arc n'est plus constant, mais dépend de l'heure à laquelle on arrive au node de départ de l'arc)

Deux façons de transformer des données d'un PTN en un graphe :
- **time-expanded** :
    - chaque node est un event (départ ou arrivée) à une station
    - chaque arc est soit un waiting-edge (reliant deux nodes d'une même station, dont le poids représente le temps d'attente d'un train particulier), soit un transit-edge (dont le poids représente le temps de trajet d'une station `A` à une station `B`)
    - une station donnée est donc représentée par un groupes de nodes
- **time-dependent** :
    - chaque node est une station
    - chaque arc est le trajet d'une station à une autre ; le poids de l'arc n'est plus constant, mais une fonction liénaire par morceau dépendant de l'heure à laquelle on prend le transport.
    - ce modèle est plus rapide "de base", mais les spécificités du PTN (e.g. transfer costs) mettent le bazar, et font perdre son avantage de base à ce modèle.

Quelques différences entre RN et PTN :
- problème propre à PTN = **transfer cost** = on veut pénaliser le fait d'avoir beaucoup de changements (note : formalisé par le front de Pareto pour RAPTOR)
- problème propre à PTN = **transfer duration** = le fait de changer de véhicule n'est pas instantané
- le fait de démarrer/arriver sur une localisation géographique donnée (plutôt que sur une station source/target) n'a pas les mêmes conséquences sur RN et sur PTN : en RN, se snapper sur la voie la plus proche est le comportement souhaité. Sur PTN, on n'a pas UNE source et UNE target, mais un SET de sources/targets. Identifier les stations à mettre dans ce set fait partie du problème.
- le calcul d'itinéraire multi-critère, déjà intéressant pour RN, est absolument indispensable pour PTN (p.ex. pour résoudre l'EAT en un minimum de transfers).
- problème propre à PTN = **traffic days** = certaines lignes de transports publics ne sont pas actives tout le temps, mais uniquement certains jours (e.g. en semaine).

Note : on se concentre sur le calcul du coût optimal, car à partir du coût optimal, on peut facilement retrouver le chemin optimal associé à ce coût.

## section 3 = tricks of the trade, why and when they work

La suite du papier est un survey des speedup-techniques, et de pourquoi elles marchent bien sur RN et pas sur PTN.

### bidirectional search

On lance deux dijkstra en parallèle : depuis la source et depuis la target (le backward search est forward search sur le **reversed graph** = graphe où tout arc (u,v) est remplacé par un arc (v,u))

À chaque round dijkstra, le node settled est celui avec le tentative-cost le plus bas _parmi les deux priority queues_.

Quand on settle un node sur une queue alors qu'il était déjà settled sur l'autre queue, c'est qu'on a un point de rencontre : on a trouvé un candidat au shortest-path, avec son tentative-cost.

Mais on continue jusqu'à ce que la somme des deux meilleurs tentative-cost restants dans les deux queues soit plus grande que le coût de notre candidat le plus intéressant : en effet, quand ça arrive, on est alors sûr qu'on ne pourra plus trouver de meilleur chemin (et le meilleur candidat devient le résultat définitif).

Intérêt du bidirectional dijkstra : au lieu d'avoir un search space qui évolue en disque de rayon r autour du source-node, on a un search space qui évolue en deux disques de rayon r/2 autour des source et target nodes.

Le speedup est pas si important que ça, mais la technique est indispensable pour d'autres techniques, e.g. CH.

**Pourquoi c'est pas applicable à PTN ?** On ne peut pas faire de bidirectional dijkstra parce que même si on connaît la target STATION, on ne connaît pas son node (i.e. l'heure à laquelle on y arrive), et c'est même un élément recherché ! Éventuellement, on peut lancer un backward search depuis chacun des nodes de la target station, mais ça rend les choses compliquées.

### hierarchy

Principe de base = quand on settle des nodes suffisamment éloignés du source-node (resp. target-node), au lieu de relaxer tous les arcs, on ne relaxe plus que les routes importantes -> le dijkstra travaille sur moins de nodes/arcs.

NOTE : on parle ici de Highway Hierarchies, et CH n'appartient pas à cette famille mais à la suivante.

**Pourquoi c'est pas applicable à PTN ?** le speedup est nul (voire négatif !) car à la différence d'un RN, un PTN local n'est pas organisé hiérarchiquement, cf. les notes sur le sujet plus bas. De plus, le préprocessing nécessaire pour HH est prohibitif pour PTN (alors qu'il est rapide pour RN), car on ne trouve pas rapidement le niveau suivant dans la hiérarchie (et pour cause : il n'y en a pas dans PTN).

### shortcuts / contraction

La formulation de l'article sur CH est intéressante : on ajoute des shortcuts, et au lieu de remplacer les arcs par les shortcuts, on modifie l'algo pour "filtrer" les arcs relaxés à la query (en l'occurence, en ne gardant que les arcs vers un noeud d'ordering supérieur).

CH est présenté comme une méthode de base ouvrant la porte à d'autres algorithmes plus intéressants.

**Pourquoi c'est pas applicable à PTN ?** CH est intéressant surtout pour les noeuds avec peu d'edges (car sinon on doit ajouter beaucoup de raccourcis en contractant le noeud). Or, dans un PTN, beaucoup de stations sont des jonctions, et donc ont beaucoup d'edges (NdM : ah bon ?!). Exprimé autrement, le graphe est trop dense pour que CH fonctionne bien.

### goal direction

Principe de base = modifier la façon dont dijkstra choisit le prochain node à settle pour "privilégier" les nodes se rapprochant du target-node.

Concrètement, le score permettant de les classer dans la priority queue est remplacé par la somme du tentative-cost ET d'une heuristique (A*). A* est correct tant que l'heuristique sous-estime la distance réelle entre un noeud et le target-node. Si elle était magiquement parfaite, l'heuristique conduirait à relaxer en priorité (donc uniquement) les edges qui sont sur le plus court chemin.

Heuristique couramment utilisée = distance à vol d'oiseau entre un noeud et le target-node (comme elle sous-estime la distance réelle, l'algo est exact) -> amélioration d'un facteur 2 à 3, tant sur RN que sur PTN. Une autre heuristique utilisant du preprocessing est l'utilisation de A* + landmarks + triangle inequality  = ALT

Arc flags = technique la plus performante (à date de rédaction de l'article) : le graphe est partitioné en k régions, et chaque arc a le bit `i` (flag) setté s'il contribue à un plus court chemin passant par la région `i`. À la query, on peut ignorer les arcs qui sont inutiles (i.e. qui n'ont pas le flag) pour les régions auxquelles appartiennent la source et la target. Ça nécessite beaucoup de preprocessing, même si on peut le réduire avec des tricks (e.g. se limiter aux noeuds sur les frontières, ou travailler en multi-level). J'ai vu dans [un autre papier](2017-route-planning-in-transportation-networks.md) que c'est la méthode utilisée par le calculateur tomtom.

**Pourquoi c'est pas applicable à PTN ?** En deux mots, les méthodes goal-directed efficaces nécessitent beaucoup de preprocessing, qui doit donc être optimisé. En pratique, ces optimisations fonctionnent bien pour RN mais pas pour PTN, car elles nécessitent des local-search, qui sont difficiles pour PTN, cf. les notes dédiées aux recherches locales ci-dessous.

### distance tables

transit node routing = méthode conduisant aux temps de réponse les plus rapides à la date de publication de l'article, de l'ordre de la µs, mais ne semble pas permettre la prise en compte traffic. C'est surtout un framework, qui n'est d'ailleurs pas incompatible avec CH.

Principe de base = si on calculait au preprocessing un tableau de N lignes et N colonnes (N = nombre de noeuds du graphe) contenant les PCC, la query serait instantanée. Cette façon de faire n'est pas faisable en pratique car quadratique.

On précalcule une distance table pour un _subset_ de paires de noeuds, les transit nodes, qui ont les propriétés suivantes :
- petit subset : en moyenne, √N (où N = nombre de nodes total) 
- tous les PCC qui couvrent une distance supérieure ou égale à un seuil D passent passent par au moins un transit node
- étant donné un source-node donné, le groupe constitué du premier transit-node de tous les PCC qui en partent sont les access nodes. Pour chaque source-node, ses access nodes sont un petit subset de tous les transit nodes possibles.

On précalcule les M² distances entre les M transit nodes. À la query, on n'a qu'à itérer sur le produit des access nodes de la source et de la target pour trouver le chemin. Pour les requêtes ou source et target sont trop proches pour que le PCC passe par un transit-node, ça veut dire par définition que ces requêtes n'ont pas réellement besoin d'être accélérées.

**Lien avec les PTN** : on peut trouver des transit nodes aussi sur PTN (même s'il faut un peu plus d'access nodes que sur RN). Ici aussi, le problème est la complexité à faire des local-search, car on en a besoin : 1. au preprocessing, pour calculer la distance de chaque node à ses access-nodes, et 2. à la query, pour calculer les itis lorsque la distance est plus petite que la distance seuil D.

## Différences entre RN et PTN

### Différence n°1 = déstructuration

Un RN s'organise naturellement en hiérarchie : les autoroutes, les routes nationales, les routes normales, et les routes de dessertes. À grande distance, les plus courts chemins monteront naturellement la hiérarchie.

Un PTN n'a pas de notion de hiérarhice au niveau local : du point de vue des PCC, toutes les lignes de bus ont à peu près la même importance. Les speed-up techniques qui exploitent la structure des RN ne marchent pas sur PTN.

Une hiérarchie commence à apparaître pour les trains au niveau national (penser TER vs. TGV), mais pas pour les transports en commun au niveau des villes.

### différence n°2 = recherche locale difficile

De quoi on parle quand on dit que les local-search sont difficiles pour PTN et pas pour RN ?

Dans un RN, un node `N` qui est proche géographiquement du source-node `SN` sera settle assez rapidement. Dit autrement, vue la nature du réseau routier, on ne peut pas avoir de cas où le search space est suffisamment grand pour être allé **très** loin géographiquement de `SN` et de `N`, sans pour autant avoir déjà settle `N` : même si des détours peuvent être nécessaires, il y a une relative relation de proportionnalité entre la distance réelle et le coût, qui fait que si on cherche à settle un node qui reste dans un voisinage géographique petit, on n'aura pas besoin d'un gros search space.

À l'inverse, dans un PTN, certains nodes, mêmes proches géographiquement, sont très "éloignés" en terme de coût (à cause de la mauvaise connectivité du réseau), et nécessitent donc un search space très grand pour pouvoir être settle. Par exemple, s'ils sont mal desservis, aller d'un village paumé à un autre village paumé peut très bien nécessiter 15h (notamment si on doit "dormir" au milieu trajet), même s'ils sont assez proches géographiquement.

Or, si le dijkstra en arrive a settle un node situé à 15h de la source, c'est qu'il aura exploré beaucoup de chemins très éloignés géographiquement (e.g. il pourra avoir pris l'avion pour aller dans un réseau de public transports complètement différent du réseau original) : un search-space aussi grand implique une quantité de noeuds/arcs explorés faramineuse... tout ça pour juste pour une recherche locale.

Dit autrement : sur un PTN, la notion de "local search" est compliquée à implémenter pratiquement : _it can take 15 hours to go to the nearby village_.
