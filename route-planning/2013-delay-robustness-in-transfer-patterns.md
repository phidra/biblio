# (ARTICLE) Delay-Robustness of Transfer Patterns in Public Transportation Route Planning

- **url** = [article](https://drops.dagstuhl.de/opus/volltexte/2013/4243/), [PDF](https://drops.dagstuhl.de/opus/volltexte/2013/4243/pdf/5.pdf) (md5sum=`2f795c5f4a75932fad857ffaf02f0c13`), [copie locale](LOCALCOPIES/5.pdf)
- **source** = [ATMOS 2013](http://algo2013.inria.fr/atmos-papers.shtml)
- **auteurs** = Hannah Bast, Jonas Sternisko, Sabine Storandt
- **date de publication** = 2013
- **date de rédaction initiale de ces notes** = 4 novembre 2020

**TL;DR** :
- face à des _delays_, on ne peut pas recalculer les TP (le temps de preprocessing est trop lourd).
- en revanche, on peut utiliser les TP calculés sur des données sans _delays_ pour construire le graphe ad-hoc, et se contenter de mettre à jour les poids du graphe ad-hoc
- si on fait ça, les trajets restent optimaux dans 99% des cas
- dit autrement, incorporer des delays dans les TP conduit tout de même à une fraction des itinéraires qui ne sont plus optimaux.
- limitation de l'article = le modèle de _delay_ est pas très réaliste.
- note : l'article donne pas mal de sources intéressantes pour obtenir des GTFS


* [(ARTICLE) Delay-Robustness of Transfer Patterns in Public Transportation Route Planning](#article-delay-robustness-of-transfer-patterns-in-public-transportation-route-planning)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = related work](#section-2--related-work)
   * [Section 3 = routing with Transfer-Patterns](#section-3--routing-with-transfer-patterns)
   * [Section 4 = delay and robustness](#section-4--delay-and-robustness)
   * [Section 5 = Experiments](#section-5--experiments)
   * [Section 6 = conclusion and future work](#section-6--conclusion-and-future-work)

## Section 1 = introduction

Objectif = prouver empiriquement le fait que les TP restent valides mêmes s'il y a des retards temps-réel + investiguer sur de quels paramètres dépendent la robustesse des TP.

## Section 2 = related work

autre définition de la robustesse (qui n'est pas celle du présent papier) = calculer un trajet en minimisant la probabilité de rater une connection

## Section 3 = routing with Transfer-Patterns

note : une façon de minimiser le nombre de transferts (donc d'être multi-critère) dans le modèle time-expanded graph, est d'attribuer un second poids ( _penalty_) aux edges, qui valent 1 lorsque l'edge passe par un transfer-node, et 0 sinon.

precomputation très très expensive : _this is the price for the very fast query times_ (quelques ms)

## Section 4 = delay and robustness

Pourquoi ça serait pas robuste : si un trip est retardé, il se peut que la route optimale passe par un autre chemin, possiblement pas dans les transfer-patterns de la source.

La structure permettant les _direct-connection lookup_ est très rapide à mettre à jour en cas de delay.

Du coup, en cas de delay, l'algo : 1. utilise les TP calculés sans délais (le graphe ad-hoc ne change pas), mais 2. les utilise dans le calcul des poids du graphe ad-hoc.

Il faut bien voir qu'en théorie, ça peut conduire à calculer et renvoyer des itis non-optimaux !

La question (addressée par l'article) est donc de savoir si 1. est pertinent, i.e. s'il est pertinent ou non de réutiliser les TP calculés sans délais, lorsqu'il y a du delay.

## Section 5 = Experiments

Des données (GTFS + graphe OSM) datant de 2012 sont [ici](https://ad-publications.cs.uni-freiburg.de/ATMOS_types_BBS_2013.materials/), pour 4 villes : Austin, Dallas, New-York, et Toronto. La source des GTFS est alléchante, mais a été fermée en 2016 : http://www.gtfs-data-exchange.com/agencies/bylocation

Le trajet de référence (considéré comme optimal) n'utilise pas les TP, mais un simple multi-criteria Dijkstra.

Les résultats sont classés en :
- optimal = le delay n'a pas d'impact sur l'optimalité du trajet calculé
- almost optimal A = moins de 5% plus lent + moins de 5 minutes plus lent
- almost optimal B = moins de 10% plus lent + moins de 10 minutes plus lent
- bad = les autres cas

On utilise différents patterns de retards pour tester.

La vaste majorité des trajets restent optimaux en cas de retard : à une exception près, entre 99.5% et 100%.

À une exception près, le pourcentage de bad est très faible quel que soit le pattern de retard (au max de l'ordre de 0.1%).

Idem pour almost optimal B, qui est de l'ordre de 0.01%

Almost optimal A un peu plus fréquent.

À noter que de façon surprenante, en essayant de calculer les TP de façon un peu plus lâche (pour être plus robuste aux delays), on n'améliore pas vraiment la robustesse.

## Section 6 = conclusion and future work

L'une des limitations du papier est que les délais dans la vraie vie ne sont pas aussi indépendants les uns des autres que leur modèle de delay. Ils envisagent donc de poursuivre les expériences avec des données rélles.
