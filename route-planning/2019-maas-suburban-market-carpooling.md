# (ARTICLE) MAAS for the Suburban Market: Incorporating Carpooling in the Mix

- **url** = [lien, qui permet de télécharger le PDF](https://www.sciencedirect.com/science/article/pii/S096585641830939X) (md5sum=`d167b26682dd07eb67b89d3151cf4caf`), [copie locale](LOCALCOPIES/1-s2.0-S096585641830939X-main.pdf)
- **source** = [Transportation Research Part A: Policy and Practice](https://www.sciencedirect.com/journal/transportation-research-part-a-policy-and-practice), et plus précisément le [volume 131](https://www.sciencedirect.com/journal/transportation-research-part-a-policy-and-practice/vol/131/suppl/C). Semble être un journal mensuel dédié à la recherche sur le sujet général _transportation_
- **auteurs** = Steve Wright, John D. Nelson, Caitlin D. Cottrill
- **date de publication** = 2019
- **date de rédaction initiale de ces notes** = 2 novembre 2020

**TL;DR** : article **très** orienté marketing, on dirait qu'il a été rédigé pour faire la promotion du projet socialcar et de l'app qui va avec. Ça fait un peu bizarre, mais il donne tout de même des insights intéressants sur MAAS. Keypoints :
- point-clé de l'article = au lieu de faire du carpooling entre le point de départ et la station TC la plus proche, on fait du carpooling entre le point de départ et n'importe quelle station TC
- conséquence = augmente l'utilisabilité des carpools (ils sont renvoyés dans 7x plus d'itinéraires qu'avant) Dit autremement : on exploitera mieux une densité donnée d'offres carpool existante, même faible, d'où le lien avec le marché des banlieues.
- une étude citée montre que seuls 25% des trajets nécessitent d'être droppé au stop le plus proche du point de départ/arrivée : dans 75% des cas, il faut plutôt un stop plus éloigné, mais qui permet de rejoindre le transport en commun le plus rapide et le plus fréquent.
- algo = TD-dijkstra sur multi-layer graph
- mono-critère : c'est une _objective function_ (combinaison linéaire de différents critères) qui est minimisée
- carpooling appliqué sur la première ou dernière leg du trajet uniquement
- carpool data pas mise à jour en temps-réel, uniquement toutes les 24h
- l'app ne semble plus maintenue, en tout cas le NDD du projet est à vendre
- les carpool data sont converties en GTFS lors d'un preprocess
- panel de 236 testeurs
- critères formels de mesure d'acceptance d'une application tech (TAM/SUS) = permet de voir si l'app est jugée utile
- problèmes principaux = 1. qualité de la donnée et 2. masse critique d'utilisateurs à atteindre
- spawne plusieurs papiers de survey intéressants
    + [2017  Mobility as a Services (MaaS) – does it have critical mass?](https://www.tandfonline.com/doi/full/10.1080/01441647.2017.1280932)
    + [2018  Community transport meets mobility as a service: On the road to a new a flexible future](https://www.sciencedirect.com/science/article/abs/pii/S0739885917302421)
- Plus généralement, pas mal de papiers MAAS [sur cette page](https://www.sciencedirect.com/journal/transportation-research-part-a-policy-and-practice/vol/131/suppl/C)

* [(ARTICLE) MAAS for the Suburban Market: Incorporating Carpooling in the Mix](#article-maas-for-the-suburban-market-incorporating-carpooling-in-the-mix)
   * [Section 1 = introduction](#section-1--introduction)
   * [Section 2 = intermodal carpool journey planning: state-of-the-art](#section-2--intermodal-carpool-journey-planning-state-of-the-art)
   * [Section 3 = description of the SocialCar algorithm](#section-3--description-of-the-socialcar-algorithm)
   * [Section 4 = practical application (RideMyRoute app)](#section-4--practical-application-ridemyroute-app)
   * [Section 5 = Results from RideMyRoute trial](#section-5--results-from-ridemyroute-trial)

## Section 1 = introduction

[fromAtoB](https://www.fromatob.com/fr-FR) = alternative à [rome2rio](https://www.rome2rio.com/fr/) = calculateur multimodal, ils commencent à intégrer du carpooling comme blablacar.


NOTES à transférer dans le repo biblio :

MAAS se fait surtout dans les grandes villes :
- zones très denses
- avec un bon réseau de PT
- les gens sont à la fois proches des PT et des loueurs de voiture
- qu'une large proportion (> 50%) de la population utilise
- les gens ont moins de voitures qu'ailleurs

Un mode peu étudié est l'autopartage, c'est le focus du papier : _how can information on carpooling be incorporated within a MaaS-type system_

Dans le cas de MAAS, l'autopartage est simplifié :
- inutile que le trajet de la voiture matche EXACTEMENT le besoin
- il suffit que le trajet de la voiture matche l'un des points seulement, et une station de PT quelquonque
- en pratique, pas besoin de one-to-one : one-to-many ou many-to-one suffit donc -> on a plus de chances de trouver du carpooling qui nous aide

Notamment, il permettrait d'étendre MAAS à d'autres zones : les banlieues, qui ont moins de facilité à accéder au réseau de PT.

Contexte = [EU-funded SocialCar Project](http://socialcar-project.eu/) (le site est down)

L'app [RideMyRoute](https://apps.apple.com/us/app/ridemyroute/id1210706592), qui a été testée sur 4 villes : Brussels, Edinburgh, Canton Ticino, Ljubljana

## Section 2 = intermodal carpool journey planning: state-of-the-art

Taxi limité aux stations proches du point de départ/arrivée.

**SEQUENTIAL SUBSTITUTION** = calculer un trajet, puis en remplacer des morceaux (e.g. les trajets piétons) avec d'autres modes (e.g. taxi). Différent d'un "vrai" calcul multimodal. Souvent, cette substitution n'est faite que pour le premier/dernier trajet piéton de l'itinéraire.

Amélioration d'une fonction objective -> un seul critère, combinaison de plusieurs critères.

## Section 3 = description of the SocialCar algorithm

Algo :
- graphe multi-layer : chaque mode de transport est un layer
- un node présent dans plusieurs layers représente une possibilité de changer de mode de transport
- un edge présent entre deux layers représente un transfert à pied entre deux modes de transport
- gestion du carpooling : on calcule la route, on regarde quels sont les stops sur son chemin (distance seuil configurable), et on crée un layer ad-hoc
- derrière, on fait tourner un dijkstra qui minimise un unique critère ( **objective function**), combinaison linéaire de critères comme : travel time, walking time, nombre de changements, ...
- walk et carpool uniquement possibles sur la première et dernière leg du trajet
- carpool data fetchées toutes les 24h, et converties en GTFS

Un point important : un papier cité dans l'article a montré que les solutions intermodales sont surtout intéressantes si elles permettent de rejoindre non pas le stop le plus proche du départ/arrivée, mais plutôt le stop qui permet de rejoindre le transport en commun le plus rapide et le plus fréquent.

## Section 4 = practical application (RideMyRoute app)

Projet sur 2016 et 2017, création d'une app. Tests sur environ 4 mois entre 2017 et 2018.

4 villes, de profils assez différents.

Inclut les P+R.

Partenaire = [Taxistop](https://www.taxistop.be/en/), utilisation d'un protocole d'échange de donné RDEX.

## Section 5 = Results from RideMyRoute trial

236 testeurs

Environ 20% des itis proposés aux utilisateurs avaient du carpooling.

Quand on est dans de grandes zones avec peu d'offres de carpool disponibles, on a (logiquement) moins de carpool dans les réponses.

Étendre le carpooling a d'autres stops que celui le plus proche du départ (arrivée) a pour effet de multiplier par 7 le nombre de carpools proposés dans les réponses.

Le nombre de trajets incluant du carpool qui auraient été trouvés même sans cette extension est d'environ 10% partout, sauf sur Edinburgh où c'est plutôt 30%

Façon de juger si l'application apporte quelque chose ou non : TAM/SUS

Limites de l'expérimentation = surtout dûes à la précision ou à l'exhaustivité des données.

Autre limite = manque de masse critique d'utilisateurs.

Les utilisateurs n'était pas inquiets pour leur vie privée (!)
