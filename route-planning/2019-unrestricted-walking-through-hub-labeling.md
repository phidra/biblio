# (ARTICLE) Fast Public Transit Routing with Unrestricted Walking through Hub Labeling

- **url** = [PDF](https://arxiv.org/pdf/1906.08971.pdf) (md5sum=`f7b5d138da637a53c6bbe9f9633db01d`), [copie locale](LOCALCOPIES/1906.08971.pdf)
- **source** = conf: [SEA2 2019](https://www.easychair.org/cfp/SEA2019) (Special Event on Analysis of Experimental Algorithms), book: [Analysis of Experimental Algorithms](https://www.springer.com/gp/book/9783030340285)
- **auteurs** = Duc-Minh Phan et [Laurent VIENNOT](https://who.rocq.inria.fr/Laurent.Viennot/), de l'INRIA
- **date de publication** = 2019
- **date de rédaction initiale de ces notes** = 17 décembre 2020
- **code** = [hl-csa-raptor](https://github.com/lviennot/hl-csa-raptor), [hub-labeling](https://github.com/lviennot/hub-labeling)

**TL;DR** :
* dans leurs versions de base, RAPTOR et CSA sont limités 1. à une station de départ fixée 2. aux trajets piétons définis statiquement dans le GTFS
* l'objectif de l'article est de permettre l'unrestricted-walking = 1. partir de la station la plus optimale, même si ce n'est pas la plus proche du point de départ et 2. marcher librement en cours d'itinéraire, si ça fait arriver plus tôt
* l'article merge une technique existante (hub-labeling) avec RAPTOR et CSA, pour permettre l'unrestricted-walking
* step 1 = preprocessing pour calculer le hub-labeling + step 2 = requête RAPTOR modifiée pour les exploiter à la place des transferts piétons
* la façon dont RAPTOR est modifié pour exploiter les hubs est détaillé dans la section 3


* [(ARTICLE) Fast Public Transit Routing with Unrestricted Walking through Hub Labeling](#article-fast-public-transit-routing-with-unrestricted-walking-through-hub-labeling)
   * [Section 1 : Introduction](#section-1--introduction)
   * [Section 2 : Preliminaries](#section-2--preliminaries)
   * [Section 3 : HLRaptor = RAPTOR with Two-Hop Transfers](#section-3--hlraptor--raptor-with-two-hop-transfers)
   * [Section 4 : HLCSA : Connection Scan with Two-Hop Transfers](#section-4--hlcsa--connection-scan-with-two-hop-transfers)
   * [Section 5 : Public Transit Data](#section-5--public-transit-data)
   * [Section 6 : Experiments](#section-6--experiments)

## Section 1 : Introduction

* Indeed, recent work [19] = [Public transit routing with unrestricted walking](https://drops.dagstuhl.de/opus/volltexte/2017/7891/pdf/OASIcs-ATMOS-2017-7.pdf) shows the benefit of using unrestricted walking over sparse transfers by measuring that it can reduce travel time by hours in Switzerland and Germany networks.
* [Unrestricted-walking] is indeed a fundamental step towards computing multimodal journeys as it is considered as a main bottleneck in [7] = [Computing Multimodal Journeys in Practice](http://tpajor.com/assets/paper/ddpww-cmjp-13.pdf)
* we scan hub lists to propagate reachability information
* the technique can easily be adapted to both RAPTOR and CSA based algorithms

## Section 2 : Preliminaries

Définition formelle d'un PTN, présentation rapide de RAPTOR, CSA, et Hub-Labeling.

## Section 3 : HLRaptor = RAPTOR with Two-Hop Transfers

Dans RAPTOR, lorsque la liste des stops améliorés par le round courant a été identifiée, la prochaine étape est normalement d'étendre cette liste, en y ajoutant les stops qui peuvent être atteints via un transfert à pied depuis un stop amélioré.

HLRaptor modifie ceci en deux étapes :
* step 1 = au lieu de suivre les transferts des stops améliorés, on met à jour l'heure d'arrivée aux out-hubs des stops améliorés
* step 2 = on met à jour l'heure d'arrivée aux stops dont les in-hubs sont les out-hubs précédemment améliorés

À partir d'un stop donné dont l'heure d'arrivée a été améliorée par le round courant, on peut ainsi savoir de façon efficace quels sont ses stops "voisins à pied" qui seront améliorés.

La réponse à la question "quels sont les stops qui ont tel hub dans leur in-hubs" peut être précalculée en preprocessing.

Les extensions de RAPTOR pour les range-queries et pour le multi-critère peuvent également utiliser l'UW avec HL.

## Section 4 : HLCSA : Connection Scan with Two-Hop Transfers

Je n'ai pas noté le détail, mais l'UW est possible aussi avec CSA, ainsi qu'avec son extension aux profile queries.

## Section 5 : Public Transit Data

Zones = Londres + Paris + Suisse

Éventuellement, ils ont eu à rendre transitively closed (i.e. transformer le graphe en graphe complet) le graphe des transferts piétons du GTFS.

Le snapping des stops sur le graphe piéton (OSM) est décrit (en gros, certains stops finissent isolés) ; la vitesse de marche retenue est 4 km/h.

Deux jeux de queries : une random + classés par ordre de grandeur de distance de marche.

Les datasets sont publiés.

L'algo de hub-labeling utilisé est celui de [Robust exact distance queries on massive networks](https://www.microsoft.com/en-us/research/wp-content/uploads/2014/07/complexTR-rev2.pdf), ça prend une à deux heures par graphe (NdM : sur mes essais en IDF, je suis plutôt à 40 minutes). Sur Paris, il y a entre 100 et 200 hubs par node.

## Section 6 : Experiments

À la grosse louche, les requêtes RAPTOR sont environ 5 fois plus lentes en unrestricted-walking qu'en walking. Avec CSA, on est plus de l'ordre d'un facteur 10. Ça reste de l'ordre de la dizaine de ms :+1:

Tout le chapitre 6.0.1 donne des billes sur la qualité des résultats si on utilise l'unrestricted-walking :

> We confirm the results of [19] showing the benefit of considering unrestricted walking.  Table 4 presentsthe percentage of time gained by using a journey with unrestrictedwalking compared to the travel timewith restricted walking.  The average gain ranges from 12% to 47% onuniform queries depending on thedataset.  City networks (especially London) seem to benefit less from unrestricted walking than the trainnetwork of Switzerland.  As observed in [19], the gain is less importantduring daytime that is querieswith departure time in the range 6h-20h here.  We observe a higher gain on low rank queries. The mediangain ranges from 13% to 47% for them. More precisely, the gain is at least 13% on half of the low rankqueries for London, 21% for Paris and 47% for Switzerland.

Note : la référence [19], c'est ce papier : [Public transit routing with unrestricted walking](https://drops.dagstuhl.de/opus/volltexte/2017/7891/pdf/OASIcs-ATMOS-2017-7.pdf) d'ATMOS 2017
