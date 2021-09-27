# (ARTICLE) Hub Labeling (2-hop Labeling)

- **url** = [l'article](https://www.researchgate.net/publication/303226533_Hub_Labeling_2-Hop_Labeling) n'a pas de PDF en libre-accès, voici [son DOI](https://doi.org/10.1007/978-1-4939-2864-4_580). (md5sum=`ed17ae4d127ff68547aaffac2512be3e`), [copie locale](LOCALCOPIES/delling2016.pdf)
- **source** = [Encyclopedia of algorithms 2016](https://www.springer.com/gp/book/9781493928637), pages 932-938
- **auteurs** = Daniel DELLING, Andrew V. GOLDBERG, Renato F. WERNECK
- **date de publication** = la synthèse dans l'encyclopédie date de 2016, mais la technique est plus ancienne ([au moins 2003](https://www.researchgate.net/profile/Edith_Cohen/publication/2876868_Reachability_and_Distance_Queries_Via_2-Hop_Labels/links/5584d86c08ae71f6ba8c5a88/Reachability-and-Distance-Queries-Via-2-Hop-Labels.pdf), [copie locale](LOCALCOPIES/Reachability-and-Distance-Queries-Via-2-Hop-Labels.pdf))
- **date de rédaction initiale de ces notes** = 4 novembre 2020

**TL;DR** :
* hub-labeling permet de connaître TRÈS EFFICACEMENT la plus courte distance entre deux noeuds
* toute la magie est dans le calcul des hubs (TODO : décrire les algos existants)
* une fois les hubs calculés, il est **très rapide** de calculer le poids du PCC allant de `A` à `B`
* sans extension, la technique ne permet pas de retrouver le chemin exact, uniquement son poids
* la technique permet également de savoir si `B` est joignable depuis `A` ou non (ont-ils des hubs communs ou non)


* [(ARTICLE) Hub Labeling (2-hop Labeling)](#article-hub-labeling-2-hop-labeling)
   * [Utilisation du hub-labeling avec mes mots à moi](#utilisation-du-hub-labeling-avec-mes-mots-à-moi)
   * [Problem definition](#problem-definition)

## Utilisation du hub-labeling avec mes mots à moi

* on associe à chaque node un set de couple {hub ; poids du PCC entre le node et le hub}
* cover property = pour un node A, tout PCC partant de A passe forcément par un hub de A
* du coup, pour connaître le poids du PCC entre A et B, on n'a besoin que des hubs de A et B :
    - retrouver les nodes qui sont à la fois des hubs de A et de B
    - le PCC entre A et B passe forcément par un de ces hubs communs
    - parmi tous les hubs communs candidats, le PCC est celui qui passe par le hub commun `H` qui minimise la somme des distances `d(A→H) + d(H→B)`
* pour les graphes orientés, chaque node `A` a DEUX sets de hubs :
    - les out-hubs, tels que tout PCC ayant `A` comme origine passe par un out-hub de `A`
    - les in-hubs, tels que tout PCC ayant `A` comme destination passe par un in-hub de `A`
    - pour un trajet A→B, le hub commun est alors à rechercher dans l'intersection des out-hubs de A et des in-hubs de B
* pour que la technique soit efficace, il faut minimiser le nombre de hubs
* toute la "magie" du système est donc dans le calcul des hubs, afin qu'il n'y en ait pas trop

**NOTE** : le hub-labeling permet de retrouver facilement le poids du PCC, mais pas le chemin lui-même.

## Problem definition

Donne quelques définitions formelles.

- stage1 = preprocessing = calculer les labels (les hubs, et leurs distance) de chaque vertex du graphe.
- stage2 = query = retrouver la distance de n'importe quel couple (s,t) à partir de leurs labels.

hub-labeling (ou 2-hop labeling) = algorithme de labeling particulier :
- chaque vertex v a deux labels, `Lf(v)` forward  et `Lb(v)` backward (dans le cas des graphes non-orientés, on n'a qu'un seul label)
- le label forward est un set de vertices, et on attribue à chacun de ces vertices x la distance entre v et x.
- le label backward est un set de vertices, et on attribue à chacun de ces vertices x la distance entre x et v.
- les vertices des deux labels forment les **hubs** de `v`
- **cover property** : quels que soient les vertex s et t, `Lf(s)` et `Lb(t)` ont au moins un vertex commun, situé sur le PCC entre s et t.
- trouver la distance de s à t est alors straitghtforward : on recherche le hub `x` parmi `Lf(s) ∩ Lb(t)` qui minimise `dist(s,x) + dist(x,t)`

FIXME : work-in-progress : il faut décrire l'algo permettant de calculer le hub-labeling d'un graphe donné.

À noter que celui utilisé dans [le papier de Viennot utilisant le hub-labeling pour l'unrestricted walking](./2019-unrestricted-walking-through-hub-labeling.md) est celui-ci : [Robust exact distance queries on massive networks](https://www.microsoft.com/en-us/research/wp-content/uploads/2014/07/complexTR-rev2.pdf).
