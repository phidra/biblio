# (ARTICLE) Frequency-Based Search for Public Transit

- **url** = [PDF](https://ad-publications.cs.uni-freiburg.de/SIGSPATIAL_Frequency_based_Search_BBS_2014.pdf) (md5sum=`7c5d37f87f705cdd475188eb3c7b8737`), [copie locale](LOCALCOPIES/SIGSPATIAL_Frequency_based_Search_BBS_2014.pdf)
- **source** = [SIGSPATIAL 2014](http://sigspatial2014.sigspatial.org/)
- **auteurs** = Hannah BAST, Sabine STORANDT
- **date de publication** = 2014
- **date de rédaction initiale de ces notes** = 30 octobre 2020

**TL;DR** :
- idée maîtresse = transformer les horaires d'une ligne sous forme d'une union de triplets : (heure de la première instance, période de répétition, nombre de répétitions)
- ça permet d'accélérer à la fois le stockage des données, et le query-time à la requête
- le papier donne une heuristique efficace pour trouver l'union de triplets représentant les horaires
- cette représentation permet d'implémenter un dijkstra très efficace pour répondre aux profile-queries (qui sont très importantes pour Transfer-Patterns)
- pour que le surcoût induit par cette représentation en vaille la peine, il faut idéalement mutualiser les infos de plusieurs jours : ça marche mieux avec 7 jours qu'avec 1 jour
- grâce à l'apport de FREQ, les deux heuristiques du papier original de TP (hub-stations + 3-legs heuristic) ne sont plus nécessaires, même sur toute l'Allemagne
- même si le papier accélère énormément le calcul des TP, celui-ci reste lourd : 981 heures sur toute l'Allemagne
- on dirait que la méthode n'est pas robuste aux délais


* [(ARTICLE) Frequency-Based Search for Public Transit](#article-frequency-based-search-for-public-transit)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = frequency-based modeling](#section-2--frequency-based-modeling)
   * [Section 3 = frequency-based profile search](#section-3--frequency-based-profile-search)
   * [Section 4 = experimental evaluation](#section-4--experimental-evaluation)
   * [Section 5 = effect on transfer patterns](#section-5--effect-on-transfer-patterns)
   * [Section 6 = conclusions and future work](#section-6--conclusions-and-future-work)

## Section 1 = introduction

L'approche TD (time-dependent) est 5x plus rapide que TE (time-expeanded). Dans des situations réalistes (optimisation du nb de changements + footpaths), TD n'est plus que 1,5x plus rapide que TE.

CSA peut-être vu comme une implémentation efficace de TE.

RAPTOR peut-être vu comme une implémentation efficace de TD.

## Section 2 = frequency-based modeling

On s'intéresse aux connections entre deux stations A et B. Leurs departure times sont un set `T` = {t1, t2, ...}. On cherche à représenter ce set comme une union de triplet (start-time, period, nombre de répétitions).

Le problème est NP-difficile, mais l'article donne deux greedy heuristiques qui marchent bien en pratique. Au final, on trouve un jeu de tuple, dont l'union représente `T`.

## Section 3 = frequency-based profile search

Les profile queries sont à la base du preprocessing pour Transfer-Patterns.

L'article tire parti du frequency-based model pour implémenter un dijkstra efficace répondant aux profile-queries.

## Section 4 = experimental evaluation

À noter que ACSA a l'air d'avoir plus de contraintes que CSA (footpaths forment des cliques, et pas d'optimisation du nombre de changements)

Les résultats de tous ces algos varient selon la quantité de footpaths : CSA est meilleur avec peu de footpaths, mais RAPTOR meilleur avec plus de footpaths. FREQ est meilleur que tout le monde d'un facteur x3 à x10

## Section 5 = effect on transfer patterns

Les hub-stations (utilisées dans le papier original de TP) semblent ne pas très bien marcher avec du traffic, mais grâce à FREQ, elles ne sont plus nécessaires (et la 3-legs heuristique non plus)

Sur toute l'Allemagne, calculer les TP avec RAPTOR prend 4300 heures. Avec FREQ, ça ne prend "que" 981 heures (41 jours).

Si on compare à TP original, en rapportant les temps à la taille du réseau, on gagne un facteur x60 :
- TP original = 63 secondes / station / million de connection
- TP avec FREQ = 1 secondes / station / million de connection

Les query-times restent très petit (de l'ordre de la ms, même sur l'Allemagne entière).

La table 5 donne des chiffres intéressants en fonction de facteurs comme l'optimisation multi-critère, la taille du graphe, le nombre de footpaths.

## Section 6 = conclusions and future work

Le papier laisse à penser que la méthode est pas robuste aux délais.

À noter que le papier a émergé dans le cade du [groupe de recherche Next Generation Route Planning](2012-to-2015-google-research-next-generation-route-planning.md).
