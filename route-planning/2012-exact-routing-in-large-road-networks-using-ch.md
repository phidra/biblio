# (ARTICLE) Exact Routing in Large Road Networks Using Contraction Hierarchies

- **url** = [article](https://dl.acm.org/doi/10.1287/trsc.1110.0401) (md5sum=`ac1ee816151793ebd45f29ed5836903c`, [copie locale](./LOCALCOPIES/trsc.1110.0401.pdf))
- **source** = [Transportation Science, Volume 46, Issue 3](https://dl.acm.org/toc/trnps/2012/46/3)
- **auteurs** = Robert GEISBERGER, Peter SANDERS, Dominik SCHULTES, Christian VETTER
- **date de publication** = 2012, première soumission en 2010
- **date de rédaction initiale de ces notes** = octobre 2021


**TL;DR** :
- le contexte des notes = ce papier m'intéresse car c'est l'un des trois cités par [ULTRA](./2019-ULTRA.md) pour leur implémentation de Bucket-CH, qui permet de faire du one-to-many efficacement.
- concernant ce contexte, le présent papier mentionne explicitement qu'ils utilisent [l'algo de 2007 utilisant les Highway-Hierarchies et des Buckets](2007-bucket-ch-aka-computing-many-to-many-using-highway-hierarchies.md) en remplaçant les HH par des CH
- l'un des tableaux compare les deux implémentations, et CH va entre 3 et 7 fois plus vite que HH (et plus la taille des sets `|S|` et `|T|` est grande, plus CH est intéressant)
- pas regardé le papier (même pas survolé), mais il a l'air de présenter les CH en général (et comme je ne l'ai pas encore lu, je ne vois pas l'intérêt par rapport à [l'article original de 2008](./2008-contraction-hierarchies.md))
- ils présentent un tableau intéressant (mais ancien puisque daté de 2012) comparant les techniques de routing, et leurs performances selon les 3 critères pertinents :
    - preprocessing time
    - query-time/settled nodes
    - overhead en bytes/nodes
- en 2012, toutes les techniques sur le front de Pareto concernant ces 3 critères (donc non-dominées) sont basées sur CH
- à noter que même si c'est le one-to-many qui m'intéresse, le focus de l'article (et de son inspiration de 2007) est le many-to-many, avec en tête l'objectif de calcul de distance-tables.

----

Citation intéressante dans la section 7, concernant le many-to-many = ils reprennent l'implémentation de many-to-many basée sur HH, en utilisant CH à la place :

> Instead of a point-to-point query, the goal of many-to-many routing is to find all distances between a given set S of source nodes and set T of target nodes.
>
> Knopp et al. (2007) developed an efficient algorithm based on HHs to compute them. Replacing HHs by CHs improves the performance, as Table 8 shows.

Autre citation intéressante, en introduction, qui montre encore une fois que c'est le caractère "hiérarchique" des graphes routiers qui est exploité :

> We exploit the observation that road networks have an inherent hierarchy with a few important and many unimportant roads and junctions, i.e., roads and junctions that are only used for local traffic near the source and target of a route.
