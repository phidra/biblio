# (BACHELOR THESIS) Multicriteria Trip-Based Public Transit Routing

- **url** = [PDF](https://i11www.iti.kit.edu/_media/teaching/theses/ba-potthoff-20.pdf) (md5sum=`6da0447829c25f08458c09a881d5f420`), [copie locale](./LOCALCOPIES/ba-potthoff-20.pdf)
- **source** = page perdue sur le [site du KIT](https://i11www.iti.kit.edu/)
- **auteur** = [Moritz Timo POTTHOFF](https://pp.ipd.kit.edu/person.php?id=226&lang=en), qui semble avoir été prof au KIT (deux reviewers/advisors sont des co-auteurs d'ULTRA-trip-based : Jonas SAUER et Tobias ZÜNDORF)
- **date de publication** = 2020
- **date de rédaction initiale de ces notes** = septembre 2021

**TL;DR** = papier explorant l'adaptation de TripBased au multi-critère :

* je n'ai fait que lire intro et conclusion, mais c'est pas fi-fou, et ça a l'air moins bien que Bounded-McRAPTOR
* pour utiliser TripBased en multi-critère, il a fallu modifier à la fois preprocessing et query
* de plus, les modifications sont propres à chaque critère ajouté : Walking Trip-Based est différent de Fare Zone Trip-Based
* le gain de temps pour Walking (facteur compris entre 1 et 4) est clairement pas fi-fou, surtout comparé à Bounded-McRAPTOR
* pire : pour Fare Zone, l'algo est plus lent qu'un McRAPTOR de base
* le papier date de 2020, donc la section bibliographique est sans doute intéressante (j'ai pas regardé en détail)


## Notes vrac

Je ne lis que l'introduction et la conclusion :

- Le preprocessing est adapté est optimisé pour chaque critère qu'on veut ajouter (on a un preprocess walking duration, un preprocess fare)
- Le surcroît de temps de preprocessing semble monter assez haut (36 minutes pour Switzerland, malgre 128 (!) threads)
- Extension de l'algo : Walking Trip-Based Algorithm
- Extension de l'algo : Fare-Zone Trip-Based Algorithm

Il a fallu adapter À LA FOIS le preprocessing et la query pour chaque critère...

Ça a pas l'air générique du tout :

> The preprocessing stage of the Trip-Based algorithm was adapted for both extensions to ensure correctness
>
> While both extensions optimize journeys for three criteria, the different additional criterion, minimum walking time versus minimal fare zone subset, required different approaches for the query algorithms.

Note, le papier s'intéresse tout de même (et semble apporter des solutions concrètes) à un problème pas simple = le multi-critère minimisant le prix :+1:

On dirait qu'avec 3 critères (walking), le preprocess TripBased ne peut pas pruner autant de trips qu'avec les deux critères classiques :

> Evaluation using real world public transit networks showed that the transfer reduction becomes significantly less effective for the Walking Trip-Based algorithm compared to the standard variant.

Speedups comparés à McRAPTOR = entre 1 et 4 (c'est clairement pas fi-fou : Bounded-McRAPTOR paraît bien plus intéressant pour le walking)

Pour le Fare-Zone, c'est encore pire : les queries sont deux fois plus LENTES que le McRAPTOR équivalent.

Dans la conclusion, ils disent plus ou moins que ULTRA-trip-based est plus intéressant aussi bien en terme de perfs que du point de vue fonctionnel (mais ils ouvrent la voie à un mix entre ULTRA-trip-based et leur extension Walking-TripBased)

La fin du papier mentionne en passant l'intégration du multi-critère avec ULTRA :

> By integrating ULTRA, McRAPTOR and Trip-Based Routing, the strengths of each algorithm could be combined.
