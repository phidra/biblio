# (ARTICLE) Time-Dependent Contraction Hierarchies and Approximation

- **url** = [PDF](https://algo2.iti.kit.edu/download/sea10_atch.pdf), (md5sum=`e09a9a3ed6416b2540b666caff79a2ab`), [copie locale](./LOCALCOPIES/sea10_atch.pdf)
- **source** = [SEA 2010](https://dblp.org/db/conf/wea/sea2010.html)
- **auteurs** = Gernot Veit Eberhard BATZ, Robert GEISBERGER, Sabine NEUBAUER, and Peter SANDERS (Karlsruhe Institute of Technology)
- **date de publication** = 2010
- **date de rédaction initiale de ces notes** = juin 2020

# Transfert de mes notes brutes

## Section 1 = Introduction

* Semble introduire le terme "ATCH" = approximated TCH
* L'objectif du papier est de réduire le volume des graphes (le papier ne présente pas TCH, mais l'une de ses améliorations).
* Deux types de requêtes :
    - time-dependent routing (= ce qui nous intéresse : trouver le chemin permettant d'arriver le plus tôt dans un graphe time-dependent)
    - travel time profiles = quel sera mon temps de parcours du trajet A->B sur les prochaines 24 heures, en fonction de mon heure de départ (e.g. pour choisir la meilleure heure à laquelle partir). C'est une généralisation du time-dependent routing (qui calcule le temps de trajet pour UN point donné dans le temps, alors que le profile le calcule pour N points)
* Approximation des fonctions linéaires par morceau (représentant le poids d'un edge en fonction de l'heure) : n'empêche pas l'exactitude du résultat !
* Dans un TCH de base, il y a beaucoup de redondance d'information.
* Approximation des shortcuts, mais edges "réels" conservés exacts.
* bidirectional search dans cette Approximated-TCH nous renvoie un CORRIDOR de shortcuts.
* Ça limite la recherche aux shortcuts du corridor (on est sûr que la réponse exacte recherchée est dans le corridor).
* Derrière, on n'a plus qu'à expand les shortcuts du corridor, et rechercher le chemin exact dans les shortcuts expanded (qui, eux, ne sont pas approximés)
* (Pour calculer les travel-time profiles, on récupère le CORRIDOR aussi, puis on le CONTRACTE pour faire les différents calculs permettant de construire le travel-time profile.)
* En acceptant d'être un peu moins précis que l'exactitude parfaite, on accélère grandement les calculs.
* EA = earliest arrival = la généralisation du problème du plus court chemin quand le poids des edges est time-dependent (en effet, le chemin à renvoyer n'est plus nécessairement le plus court)


## Section 2 = Preliminaries

Les TTFs utilisées pour TCH et leurs opérations :
* notamment : les TTF suivent le FIFO property (on NE PEUT PAS arriver plus tôt en reportant son départ)
* notation : |f| est la "complexité" d'une TTF, i.e. le nombre de "morceaux" dans la fonction par morceaux représentant la TTF

Avec une TTF f, on peut :

1. **evaluation** = calculer f(τ)

2. **linking** = composer des TTFS :
* si on a deux edges adjacents :
    - (u,v) de TTF f, qui à τ associe f(τ)
    - (v,w) de TTF g, qui à τ associe g(τ)
* on veut calculer la TTF `g∘f` du "chemin" `<u,v,w>` :
    - à τ, g∘f associe f(τ) + g(τ+f(τ))
    - f(τ) est le temps de parcours de (u,v) : c'est au temps τ qu'on cherche à aller de u à v
    - g(τ+f(τ)) est le temps de parcours de (v,w) : c'est au temps τ+f(τ) qu'on cherche à aller de v à w
* TL;DR : le linking, c'est le fait de calculer la TTF composée de 2 edges
* calculable en O(|f| + |g|)
* la complexité de la composée g∘f est au max la somme des composées
* le linking est une opération associative : f∘(g∘h) = (f∘g)∘h

3. **minimum** = merge :
* à partir de deux edges E et E' parallèles, i.e. qui ont même extrêmités ut et v (et leurs TTFs f et f')
* on MERGE E et E' en un seul edge E" préservant les plus courts chemins
* sa TTF f" associe à tout τ : min{f(τ), f'(τ)}

Notes :
* dans un graphe time-dependant, pour un même couple {source s ; target t}, en fonction de l'heure de départ τ, il peut y avoir PLUSIEURS plus courts chemins/temps de trajets
* travel-time profile (TTP) = pour une paire {s,t} de noeuds donnée, c'est la TTF qui à tout τ associe le plus court chemin de s à t (et son temps de parcours)
* si on appelle f cette TTF (ou plutôt ce TTP), on peut définir une fonction `arrf` pour calculer l'heure d'arrivée en fonction de τ : `arrf` associe à tout heure de départ τ l'heure d'arrivée τ+f(τ)
* inversement, la fonction réciproque qui à une heure d'arrivée donnée associe l'heure (ou les heures) à laquelle il faut partir pour l'atteindre est notée : depf = (arrf)⁻¹

NdM : dijkstra classique = applicable au problème du PLUS COURT CHEMIN, i.e. pour un graphe statique
* dans un dijkstra classique, lorsqu'on visite un noeud, on peut modifier son label (tentative_distance)
* ce label est la plus courte distance connue JUSQU'ICI nous permettant de rejoindre ce noeud (initalement, le label vaut +∞)
* lorsqu'on relaxe un edge (u,v) :
    - soit le label de v est déjà plus court (on a rejoint v par un chemin plus long que celui du label)
    - soit on a trouvé un chemin plus court que celui du label, on remplace alors le label par  d(u) + len(u,v)
    - on synthétise ça en disant que le label d'un noeud v lorsqu'on relaxe l'edge (u,v) prend pour valeur : min{d(v), d(u) + len(u,v)}
* lorsque le noeud en question finit par être en tête de la priority_queue, son label devient sa plus courte distance DÉFINITIVE (le noeud est settled)

----

**Time-Dependent Dijkstra** = généralisation de dijkstra au problème de l'EARLIEST ARRIVAL, i.e. pour un graphe TD
* les valeurs associées aux noeuds ne sont plus des distances, mais des heures d'arrivée
* lorsqu'on relaxe un edge (u,v), on prend en compte l'heure d'arrivée à u pour savoir quel poids ajouter à d(u) (en effet, celui-ci est une TTF)
* en dijkstra classique, la nouvelle tentative_distance de v valait : min{d(v), d(u) + len(u,v)}
* en TD-dijkstra, la nouvelle tentative_distance de v vaut          : min{d(v), d(u) + len[u](u,v)}   (len(u,v) dépend de l'heure d'arrivée à v = la "distance" de u !)
* du coup, le noeud de départ n'a plus pour label 0, mais a pour label τ, le departure_time

----

**Profile Search** = renvoie le profil d'un itinéraire {s,t} (i.e. le temps de parcours de l'iti en fonction de l'heure de départ)
* dans le dijkstra, au lieu que le label soit un scalaire représentant la plus petite distance (ou l'heure d'arrivée la plus tôt) au noeud, le label est une fonction dépendant du temps : à tout temps τ, elle associe l'heure d'arrivée la plus tôt (connue jusqu'ici) au noeud
* en gros, on généralise le scalaire en une fonction dépendant du temps τ de departure_time depuis la source
* pour relaxer un edge (u,v), on merge la TTF actuelle de v avec celle de la composition TTF(edge:u→v) ∘ TTF(u)
* (si v n'a pas encore de TTF au moment où on arrive dessus (pour la première fois, donc), on l'initialise à la TTF constante infinie)

----

**Interval Search** = même principe que ProfileSearch, mais ne travaille que sur les bornes inf/sup des TTFs, ce qui rend les calculs BEAUCOUP moins lourds
* les labels utilisés dans dijkstra sont des 2-uple [inf, sup]
* si un noeud N a pour label [inf, sup], on sait qu'on NE PEUT PAS atteindre ce noeud avant inf, et après sup
* le label initial de chaque edge est [0,+∞]
* supposons qu'un noeud v ait pour label [inf(v), sup(v)], alors lorsqu'on relaxe un edge (u,v), si (u,v) nous permet d'arriver à v AVANT inf(v), on modifie le label de v
* dit autrement, si `inf(u) + inf(u,v) < inf(v)`, alors inf(v) prend comme nouvelle valeur inf(u) + inf(u,v)
* à noter qu'on parle de 2-uple [inf,sup] aussi bien pour un noeud v (auquel cas, c'est son dikstra-label) que pour un edge (u,v) (auquel cas c'est une approximation de sa TTF)

----

**Corridor** :
* sous-graphe du graphe contracté, dans lequel un iti de s à t est possible (donc : un corridor est propre à un iti {s,t})
* l'intérêt d'un corridor est qu'il y a BEAUCOUP moins de noeuds que dans le graphe contracté initial
* on peut donc y faire des recherches "exhaustives" (i.e. profile-search ou TD-dijkstra) de façon plus efficace
* si je comprends bien : 1. on utilise interval search pour réduire l'espace de recherche en calculant un corridor 2. on fait un TD-dijkstra dans le corridor

----

**Earliest Arrival queries on TCH** :
* **step 1 = on fait un bidirectinal search modifié** :
    - forward search = TD-dijkstra
    - backward search = interval search
    - à l'issue de la step1, on a trouvé des candidates (i.e. des meeting points)
    - par ailleurs, l'interval search en backward a permis d'exclure des nodes/edges qui ne sont pas sur le plus court chemin à coup sûr
    - dit autrement : l'interval search en backward a permis de restreindre les nodes/edges possibles → il a créé un corridor

* **step 2 = on relance un TD-dijkstra depuis CHAQUE candidate** (en initialisant son τ à la valeur donnée par le forward search)
    - détail crucial : ce TD-dijkstra est LIMITÉ aux nodes/edges du corridor (vu que seuls eux peuvent accueillir le plus court chemin)
    - QUESTION : quand décide-t-on qu'on a suffisamment de meeting-points ? Ou bien on ne fait la step2 que quand on a trouvé TOUS les meeting points possibles ?

## Section 3 = Applying Approximations

### Approximated TCH

* étant donné un ε ∈ [0,1] (e.g. 0.05 = 5% d'erreur), on va approximer les TTF des shortcuts à ε près
* on laisse les TTF des edges originaux (au moins pour un τ donné) inchangés
* pour les autres TTFs (qui sont donc des TTF de shortcuts uniquement quel que soit le temps) on les remplace par une TTF upperbound f↑
* cette TTF-upperbound f↑ est toujours supérieure ou égale à la TTF non-approximée f, mais n'est jamais supérieure de plus de ε
* en revanche, elle peut la "lisser" et donc la simplifier : l'objectif est que f↑ soit beaucoup plus simple que f, et donc qu'on ait `|f↑| << |f|`
* note : la TTF "jumelle" de f↑ qui sera la TTF-lowerbound f↓ est simplement le "miroir" de f↑ mais EN DESSOUS de f : f↓ = f↑ / (1+ε)
* note bis : comme les TTF d'edges originaux ne sont pas approximées, on a toujours pour ces edges ε=0 soit f = f↓ = f↑
* l'algo géométrique utilisé pour approximer les TTF minimise de façon optimale la complexité |f↑| pour un ε donné (les références sont dans le papier)
* NdM : je vois l'Appproximated TTF comme l'application d'une érosion morphologique à une courbe (qui a tendance à la lisser)

### Min-Max TCH

Un cas extrême d'Approximated TCH : où la TTF f est approximée par deux nombres : min(f) et max(f)

### Arrival interval search

C'est juste l'application d'un IntervalSearch à un Approximated graph dans le sens "classique", i.e. en partant de la source s

### Backward Travel Time Interval Search

* C'est l'application d'un IntervalSearch à un Approximated graph dans le sens "backward", i.e. en partant de la source t
* le principe, c'est d'essayer de répondre à la question : "si j'arrive à t entre σ et σ', à quelle heure étais-je sur tel prédécesseur de t" ?
* par exemple, pour un prédécesseur P de t, tel que P a pour Approximated-TTF [min=13, max=25], on peut rejoindre t en étant en P entre [min=σ-25 et max=σ'-13]
    - NOTE : attention il faut soustraire au min le MAX du prédécesseur (et vice-versa)
    - un moyen mnémotechnique = le min doit toujours rester inférieur au max, même si σ == σ'
    - par ailleurs, on SOUSTRAIT des deux côtés, car dans les deux cas, on arrive en P *AVANT* d'arriver en t
* en continuant le process pour un prédécesseur P2 de P, qui a pour A-TTF [min=5, max=18], on peut rejoindre t en étant en P2 entre [min=σ-43 et max=σ'-18]

QUESTION : pas clair = ce que l'algo produit ? l'intervalle de temps (et les plus courts chemins possibles) pour aller d'un noeud sorce s à t ?
* ma compréhension de ce que produit l'algo : pour chaque noeud v settled, l'algo indique l'intervalle de temps nécessaire pour faire le trajet entre v et t et arriver entre σ et σ'
* dit autrement, si l'algo produit [min(v) ; max(v)], il nous permet d'affirmer : "pour arriver en t entre σ et σ', je n'aurais pas besoin de partir avant min(v), et je n'aurais pas le droit de partir après max(v)"
* du coup il produit des informations sur un ENSEMBLE de noeuds (plutôt que sur un seul noeud source)
* il sert donc plutôt à associer des labels à un jeu de noeuds settled, et pour chaque noeud à limiter les edges pouvant se trouver sur le plus court chemin (donc à calculer un corridor pour chaque noeud relaxé)
* derrière, si parmi le jeu de noeuds relaxés on a des candiates trouvés par un forward-dijkstra, on peut restreindre un dijkstra exact au corridor calculé
* (c'est pas entièrement clair la question de savoir si les edges du corridor font partie de ce que renvoie l'algo ou non)

QUESTION : pas clair = la formule permettant de relaxer un edge :

notations :
* on s'intéresse à un edge (u,v), donc on peut parcourir l'edge de u à v
* MAIS comme on s'intéresse à une propagation backward, on "part" de v pour faire le calcul en "remontant" vers u
* pour simplifier, on suppose que u n'a jamais été visité
* par ailleurs, v a pour labels [min(v) ; max(v)]  (dit autrement : pour rejoindre t depuis v entre σ et σ', il faut partir entre min(v) et max(v)]
* comme précédemment, min(X) peut représenter :
    - si X est un noeud : son label-min = l'heure à partir de laquelle il faut partir pour arriver en t entre σ et σ'
    - si X est un edge : la valeur min de sa Approximated-TTF = le temps le plus petit possible nécessaire pour traverser l'edge (u,v)

ma compréhension : u va se voir attribuer le label min(u) :
* on prend le cas pessimiste de l'edge (u,v) où on met BEAUCOUP de temps pour le traverser, soit max(u,v)
* dans ce cas, le plus petit label pour u est le plus petit label pour v auquel on soustrait la plus grande valeur (pessimiste) de l'edge (u,v)
* dit autrement on va attribuer le label min : min(u) = min(v) - max(u,v)

étape intermédiaire = autres billes pour comprendre la formule :
* max (  depf↑uv (σ-qv)  )
* depf↑uv (σ-qv)  = l'heure à laquelle il faut partir de u pour rejoindre v en (σ-qv), en considérant que le temps de trajet de u à v est la constante f↑uv = max(u,v)
* du coup je comprends pas trop que vient faire "max" ici : depf↑uv (σ-qv) est DÉJÀ une valeur scalaire pour moi...
* ah non pas forcément : f↑ n'est une constante QUE dans le cas d'une Min-Max TCH, et pas dans le cas d'une Approximated-TCH !

du coup je reprends : billes pour comprendre la formule :
* σ-qv l'heure de départ e v avant laquelle on est sûr d'arriver avant σ en t. Dit autrement, il NE FAUT PAS partir de v avant σ-qv si on veut rejoindre t après σ
    - en effet, qv étant max(v), c'est l'hypothèse "pessimiste" selon laquelle on prendra le plus de temps possible pour faire le trajet
    - en partant de v avant σ-qv, on est sûr d'arriver AVANT σ en t
* depf↑uv (σ-qv)  = l'heure à laquelle il faut partir de u pour rejoindre v en (σ-qv) en considérant que le temps de trajet de u à v est donné par la TTF upper-approximée f↑uv 
* max (  depf↑uv (σ-qv)  ) = la valeur max possible (donc optimiste sur le temps de trajet ?) de l'heure de départ de u sur toute la TTF upper-approximée f↑uv, pour rejoindre v en (σ-qv)
* ----------------------------------------
* σ'-pv l'heure après laquelle il ne faut pas partir de v si on veut être sûr de pouvoir rejoindre t avant σ'
    - en effet, pv étant min(v), c'est l'hypothèse "optimiste" selon laquelle on prendra le moins de temps possible pour faire le trajet
    - en partant de v après σ'-pv, on est sûr d'arriver APRÈS σ' en t
* depf↓uv (σ'-pv) = l'heure à laquelle il faut partir de u pour rejoindre v en (σ'-pv) en considérant que le temps de trajet de u à v est donné par la TTF lower-approximée f↓uv 
* min( depf↓uv (σ'-pv) ) = la valeur min possible (donc pessimiste sur le temps de trajet ?) de l'heure de départ de u sur toute la TTF lower-approximée f↓uv, pour rejoindre v en (σ'-pv)

Bon, ça m'aide un peu à avancer, mais je n'ai toujours pas fini de comprends la formule...

### Queries

L'algo décrit est valable pour ATCH et min-max TCH

Éléments en entrée :
* un noeud source s
* un noeud target t
* un departure_time τ0

#### phase 1 = bidirectional search / upward

* search forward et backward, et dans les deux cas en "upward" (i.e. en ne considérant que les noeuds d'ordre supérieurs)
* forward search = interval search à partir de s (de label initial [τ0;τ0]). Chaque noeud visité v a pour label [arrival_min;arrival_max] = la plage d'heures d'arrivée possibles.
    - si v a pour label [τ0+145;τ0+196], ça veut dire qu'en partant de s à τ0, on est CERTAIN de ne pas arriver à v avant τ0+145 ni d'arriver après τ0+196
* backward search = interval search depuis t (de label initial [0;0]). Chaque noeud visité v a pour label [travel_to_t_min;travel_to_t_max] = la plage de temps de parcours possibles.
    - si v a pour label [145;196], ça veut dire que pour faire le chemin de v à t, on est CERTAIN qu'il ne faut pas moins de 145 et pas plus de 196 secondes
* les meeting points des deux searchs sont les CANDIDATES

QUESTION : c'est pas clair quand on arrête les search : dès qu'on a trouvé des candidates ? Dès qu'on a SETTLED tous les candidates ?

QUESTION : d'ailleurs, dans un interval search, ça veut dire quoi SETTLE un candidate ? y a-t-il une priority queue en interval search et si oui, sur quoi porte-t-elle ?
* dans un dijkstra classique, la priority queue porte sur la plus courte distance "actuellement connue" entre la source et le noeud v
* dans un TD dijkstra, l'équivalent serait la notion de "plus court intervalle" entre la source et le noeud v
* en gros, si un chemin permet de diminuer la borne inférieure (resp. sup) de l'intervalle, il faut mettre à jour l'intervalle et les edges du chemin "lower"
* par contre, ça ne dit pas comment "classer" les noeuds dans la priority queue
* en effet, dans un dijkstra classique, une fois qu'un noeud (ayant la plus petite priority) est dépilé, il est traité à jamais
* alors que dans un TD-dijkstra, lorsqu'un noeud est dépilé, il se peut qu'on doive mettre à jour sa borne supérieure, car un plus court chemin permet d'arriver plus tard ?
* hmm, en fait non : aussi bien la borne inférieure que la borne supérieure ne peuvent que DIMINUER (sinon c'est pas des plus courts chemins)
* dans quel cas mettrait-on à jour la borne min d'un noeud v :
    - c'est qu'on a trouvé un chemin (différent) qui permet d'arriver à v plus rapidement que celui qu'on avait jusqu'ici, en choisissant les poids OPTIMISTES
    - dit autrement : en retenant le plus petit poids possible pour chaque edge, on a trouvé un meilleur chemin que celui qu'on avait jusqu'ici
* dans quel cas mettrait-on à jour la borne max d'un noeud v :
    - c'est qu'on a trouvé un chemin (différent) qui permet d'arriver à v plus rapidement que celui qu'on avait jusqu'ici, en choisissant les poids PESSIMISTES
    - dit autrement : en retenant le plus grand poids possible pour chaque edge, on a trouvé un meilleur chemin que celui qu'on avait jusqu'ici
* ma compréhension des choses : peu importe si on ordonne la priority queue par borne min ou borne max, car les noeuds ne sont JAMAIS settled
* en effet, si on ordonne par poids optimistes (= borne min), même une fois un noeud dépilé, il se peut qu'on trouve un autre chemin plus court pour les poids pessimistes
* Dans le code, pour une min-max TTF, on classe dans la priority queue en fonction de la borne min (et seulement en second temps par la borne max)
* HUMM, en fait, on pourrait tout de même ordonner ?
    - quand même avec les poids pessimistes, tous les noeuds non-visités ont un poids pessimiste plus grand que le mien je suis sûr que je ne vais plus améliorer mon PCC pessimiste ?
    - EDIT : ben non : certes je ne vais plus améliorer mon poids pessimiste, mais je peux tout de même améliorer mon poids optimiste
* Du coup, pour moi, la réponse à ma question : un noeud est SETTLED (i.e. n'évolue plus) lorque tous les noeuds restants à visiter ont à la fois :
    - un poids OPTIMISTE plus grand que mon poids optimiste
    - un poids PESSIMISTE plus grand que mon poids pessimiste

In fine, à l'issue de cette première phase, tous les noeuds CANDIDATE c (i.e. visités à la fois par le forward search et le backward search) :
* (forward)  ont un intervalle d'arrivée depuis s = [arrival_min ; arrival_max]
* (backward) ont un intervalle de temps de parcours pour arriver à t = [travel_to_t_min;travel_to_t_max]
* ça nous renseigne beaucoup sur la réponse finale recherchée = l'earliest arrival time à t en partant de s à τ0
* l'arrival time à t en partant de s à τ0 (et en passant par c) est inclus dans l'intervalle : [arrival_min+travel_to_t_min ; arrival_max+travel_to_t_max]

#### phase 2 = pour chaque candidate c, forward interval search, downward
* l'objectif est de raffiner le backward search de la phase 1 (qui partait de t) en utilisant les infos que nous a apporté le forward search de la phase 1
* en effet, on a maintenant plus d'infos sur les edges qui nous intéressent, vu qu'on sait qu'ils doivent être accessibles depuis c dans l'intervalle renvoyé par la phase 1
* à partir de chaque candidate c, on fait un forward interval search :
    -  on initialise les intervalles de chaque c à leur valeur renvoyée par la phase 1
    -  le forward search n'utilise QUE les noeuds touchés par le backward search de la phase 1
    -  de plus, ce forward search se contente de DESCENDRE les ordres (l'inverse de ce qui est fait d'habitude)
* pourquoi un downward search ?
    -  comme C est un meeting_point entre la recherche forward et backward, ce sera le point avec le plus haut ordering du trajet
    -  du coup, comme le search de la phase 2 part de C pour rejoindre t, il faut nécessairement faire une recherche downward
* à l'issue de cette phase, non seulement on a réduit le corridor entre les c et t mais on a également pour chaque c un intervalle d'arrivée [arrival_min;arrival_max] en t !

#### phase 3 = backward interval search
* ici aussi, l'objectif est de raffiner le backward search de la phase 1 (qui a déjà été raffiné à la phase2) pour limiter les edges concernés
* on va utiliser la nouvelle information à notre disposition = on connaît maintenant un intervalle de temps d'arrivée en t [arrival_min;arrival_max]
* on refait un interval search backward (et upward) qui part de t, avec comme valeurs initiales [arrival_min;arrival_max]
* on se limite aux edges considérés comme intéressants dans la phase 2 (i.e. on ignore les edges qui ne peuvent pas faire partie d'un plus court chemin)
* dès qu'on atteint un node qui était settled par le forward search de la phase 1, ce node est de nouveau un candidate c

#### phase 4 = dijkstra bidirectionnel final, en utilisant les TTF réelles, et en se limitant aux edges intéressants

en résumé :
* la phase 1 forward a permis de définir un subset de noeuds et d'edges pouvant faire partie du plus court chemin pour rejoinder les candidates c
* la phase 1 forward a (dit autrement) permis d'écarter certains edges/noeuds (ceux qui, quelle que soit l'heure, n'apparaissent pas dans le label des candidates)
* la phase 1 backward, a permis de même d'écarter des edges/noeuds qui à coup sûr ne font partie des plus courts chemins entre les différents c et t
* la phase 2 a permis de restreindre encore plus les edges potentiellement utiles entre les différents c et t.
* la phase 3 a permis de restreindre ENCORE PLUS les edges potentiellement utiles entre les différents c et t, et de définir un nouveau set de candidates.

À ce stade, on a donc fortement réduit le set d'edges pouvant faire partie d'un PCC entre s et c d'une part, et entre c et t d'autre part, on peut maintenant expand tous les raccourcis de ces edges (on a donc un corridor = un sous-graphe constitué d'edges originaux).

Comme les raccourcis ont été expanded, le corridor ne contient que des edges originaux, qui ont une TTF EXACTE !

On peut donc faire un TD-dijkstra bidirectionnel "normal" (ne visitant que les edges du corridor)  utilisant ces TTFs exactes pour trouver l'earliest arrival \o/

À noter que les phase 2 et 3 sont en quelque sorte optionnelles, mais servent à réduire les edges potentiellement intéressants entre les différents c et t

### Vrac

Je laisse de côté les requêtes de type TTP (TimeTravelProfile = pour un couple {s,t} donné, l'heure d'arrivée pour chaque heure de départ possible)

**Inexact earliest arrival** : en gros, on va approximer les TTF (même originales !) par une Approximated-TTF de la moins grande complexité possible, qui est toujours à ε de la vraie TTF. Idem, je laisse ça de côté.

QUESTION : stall-on-demand toujours pas clair.

## Section 4 = Experiments

### paramètres 

Deux sets de données :
* Germany = ~5M nodes ~10M edges, environ 8% d'edges avec des speed-profiles
* Europe = ~20Mnodes ~40M edges, la plupart des edges ont des SP

Ils laissent de côté l'affichage du chemin le plus court (ils ne s'intéressent qu'à l'heure d'arrivée).

Ils comparent l'occupation de RAM à celle nécessaire pour faire un TD-dijkstra sans approximation :
* Germany = 95 byte / node
* Europe = 76 byte / node

1000 couples source+target random, τ0 random aussi

Ils monitorent également des métriques sur la performance du dijkstra : nombre d'edges "relaxés", nombre d'évaluation de TTF, etc.

Leur référence est un TD-disjktra "exhaustif" (i.e. hors ATCH, voire hors TCH ?)

### un point intéressant de l'analyse des algos :

* ils classent les requêtes en fonction du nombre de noeuds settled par le dijkstra
* c'est une mesure de la "complexité" de la requête
* si source et target sont TRÈS éloignés l'un de l'autre, le dijkstra va avoir tendance à explorer et settle beaucoup de noeuds
* (c'est utilisé dans pas mal de leurs autres papiers)

### principales conclusions :

* dans l'idée avec ATCH, on gagne en espace, mais on perd en performances au runtime
* en pratique, avec ε suffisamment petit, il n'y a pas de ralentissement au runtime
* c'est donc un tradeoff RAM / runtime
* citation importante dans le contexte du merge contraction et ordering : _As our preprocessing works in two phases (node ordering and contraction) there are preprocessing times like 0:28+0:09 for TCH based techniques (28 min node ordering and 9 min contraction)._


