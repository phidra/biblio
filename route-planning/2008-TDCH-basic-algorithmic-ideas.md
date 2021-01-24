# (ARTICLE) Time Dependent Contraction Hierarchies — Basic Algorithmic Ideas

- **url** = [PDF](http://algo2.iti.kit.edu/documents/tdch.pdf) (md5sum=`b0cf56b0732dd81f490615dd3360b31c`), [copie locale](./LOCALCOPIES/tdch.pdf)
- **source** = conf: [ALENEX 2016](https://epubs.siam.org/doi/book/10.1137/1.9781611974317)
- **auteurs** = G. VEIT BATZ, Robert GEISBERGER and Peter SANDERS (Karlsruhe Institute of Technology)
- **date de publication** = 2008
- **date de rédaction initiale de ces notes** = juin 2020

**TL;DR** — idées clés derrière TDCH, résumées en quelques pages, donc plus rapides à lire que les papiers et thèses sur le sujet.

# Transfert de mes notes brutes

Challenge de taille = dans un graphe time-dependent, utiliser un bidirectional dijkstra (indispensable pour CH) nécessite de connaître l'heure d'arrivée... or, c'est justement ce qu'on cherche !

* première version simple de l'ordering = ordering statique sur la moyenne des TTFs (suppose que l'importance d'un node n'est pas très affecté par son pattern de traffic)
* deuxième version simple de l'ordering = ordering en se basant sur un petit subset des "morceaux" de la TTF
* généralisation de dijkstra aux profils (i.e. au cas où, pour représenter son coût, on associe à un edge : un PROFIL dépendant du temps plutôt qu'un scalaire)
    + tentative_distance dans dijkstra est remplacé par TTF
    + ajouter le coût d'une edge est remplacé par le linking d'une TTF
    + prendre le minimum entre deux edges/chemins est remplacé par prendre le minimum des TTF

----

* pas clair = pourquoi parle-t-il de `length` (dans time-dependent shortcut length) Mon interprétation = c'est une (mauvaise) façon de dire "le coût"
* un shortcut est à garder lorsqu'À UN MOMENT DONNÉ il fait partie du plus court chemin (même principe que pour contraction CH sur un scalaire, mais appliquée à un time-dependant weight)
* query CH = forward search sur un upward graph G↑ et downward search sur un downward graph G↓
* pour TCH forward c'est à peu près identique à CH, sauf qu'il faut tenir compte de l'heure pour attribuer un poids à un edge
* pour TCH backward c'est différent : on marque TOUS les nodes qui PEUVENT atteindre la target (sans doute le fameux corridor), puis on fait un disjktra classique entre les noeuds upwards trouvés, et les noeuds du corridor
* problème = plus on compose des TTF, plus la complexité du profil résultant augmente...
    + e.g. profil d'un edge raccourci dont l'expansion comporte beaucoup d'edges réels
    + e.g. "tentative_distance" lors du dijkstra d'un noeud assez éloigné du noeud de départ (i.e. le diamètre du dijstra-search space est grand)
* *solution* = approximation "partielle". Le truc cool, c'est que malgré l'approximation, le calcul du plus court chemin reste EXACT : on ne dégrade pas le calcul.

----

Modification de la local search 1 :
* on approxime les TTF utilisées lors des local searches par des bornes supérieures
* ça va avoir tendance à surestimer le coût de ces TTF, et donc les chemins utilisant ces approximations vont moins avoir tendance à être de plus courts chemins (witnesses)
* dit autrement, on va avoir plus facilement tendance à considérer qu'il faut ajouter un shortcut (ce qui est pas critique tant qu'il y en a pas trop)

Modification de la local search 2 :
* quand on ajoute un shortcut (i.e. on n'a pas trouvé de witness), au lieu d'utiliser sa TTF exacte, on utilise le couple {borne supérieure ; borne inférieure}
* plus tard, quand on contracte un node v et qu'on se demande si on doit ajouter le shortcut <u,v,w>, on compare la borne inférieure du shortcut à la borne supérieure du witness (c'est le cas le plus favorable au shortcut) ; si même dans ce cas favorable, le shortcut n'est pas intéressant, on peut le dégager sans remords
* sinon, c'est PEUT-ÊTRE un shortcut réellement nécessaire (mais peut-être pas), on l'ajoute quoi qu'il arrive
* en faisant ça pour la contraction de TOUS les noeuds, les shorcuts qu'on obtient ne sont pas de "vraies" TTF, uniquement des couples {borne sup;borne inf}
* on peut alors soit calculer les TTFs EXACTE de tous ces shortcuts approximés, soit modifier la query pour le faire au runtime

----

* pas clair = je n'ai pas compris cette phrase : _We can prune the forward search by marking all nodes v in the backward search space with a lower bound l(v) on the travel time to t._ (à éclaircir en lisant un papier plus détaillé)
* d'une façon générale, mieux affiner les bornes supérieure et inférieure est une source d'amélioration de TCH
* sur ATCH, on commence par calculer (sur le graphe "simplifié", i.e. n'utilisant que {borne sup;borne inf}) un set restreint d'edges shortcuts pouvant appartenir au plus court chemin
* derrière, on élague certains edges pour restreindre le set encore plus, puis on expande tous ces shortcuts pour avoir des edgs réels (avec leur TTF réelle : non-approximée)
* on fait alors un dijkstra exact sur ce cet d'edges réels
