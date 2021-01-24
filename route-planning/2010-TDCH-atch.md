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

