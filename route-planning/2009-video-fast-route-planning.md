# (ARTICLE) Round-Based Public Transit Routing

- **url** = [vidéo YouTube](https://www.youtube.com/watch?v=-0ErpE8tQbw)
- **source** = [Chaîne YouTube Google TechTalks](https://www.youtube.com/channel/UCtXKDgv1AVoG88PLl8nGXmw), _Google Tech Talks is a grass-roots program at Google for sharing information of interest to the technical community._
- **auteurs** = Présentation par Peter SANDERS (un usual-suspect du route-planning)
- **date de publication** = 2009
- **date de rédaction initiale de ces notes** = juillet 2020


## Notes vrac

(notes très vrac, il s'agit beaucoup de mots-clés, plutôt que de vraies notes construites)

* Transit nodes routing = algo le plus rapide
* Many to many
* Ride sharing
* Turn cost
* Deux façons d'introduire du dynamisme :
    - Dynamic 1 = garder la structure du graphe, mais changer la cost function
    - Dynamic 2 = modifier quelques edges du graphe
* Sur appareil mobile
* Time dependent ; TD-dijsktra = le coût d'un edge donné  dépend de l'heure a laquelle  on y arrive,  donc du plus court chemin pour y arriver
* Dans le td preprocessing, on peut avoir à relaxer un noeud plusieurs fois
* TD-CH difficile :
- TDCH difficulté 1 = au preprocess, trouver des witness-path
- TDCH difficulté 2 = à la query, le backward dijkstra nécessite de connaître l'heure d'arrivée !  (pour savoir quels poids appliquer aux tronçons)
* Profile search vs. Minmax label search ?
* Restricted profile search explique le corridor (mais j'ai pas suivi en détail)
* En 2008, ils mettaient moins de 2h pour preprocesser l'allemagne pour TCH (une raison possible = seul 12% de leurs edges sont time dependent)
* Parallelisation semble possible pour 1. contraction et 2. ordering
* Pout contraction : maximum independent sets of nodes ? _Nodes that are least important in their 1-hop neighborhood_

Les futurs travaux :
*  multiple objective (e.g. faire des trajets qui ne passe par aucun tunnel)
*  multicritere (e.g. minimiser durée  ET coût en euros du trajet)
