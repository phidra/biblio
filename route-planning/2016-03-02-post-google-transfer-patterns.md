# (BLOGPOST) An Update on fast Transit Routing with Transfer Patterns

- **url** = https://ai.googleblog.com/2016/03/an-update-on-fast-transit-routing-with.html , [copie locale sous forme d'une single-page HTML](LOCALCOPIES/2016-03-02-post-google-transfer-patterns_LOCALCOPY.html)
- **source** = le [blog des travaux de recherche de Google](https://ai.googleblog.com/)
- **auteur** = Arno Eigenwillig, Software Engineer on Google Maps Directions, [co-auteur](https://research.google/people/author39322/) du papier original sur les Transfer Patterns.
- **date de publication** = 2 mars 2016
- **date de rédaction initiale de ces notes** = 23 octobre 2020

* [(BLOGPOST) An Update on fast Transit Routing with Transfer Patterns](#blogpost-an-update-on-fast-transit-routing-with-transfer-patterns)
   * [Transfer Patterns](#transfer-patterns)
   * [Scalable Transfer Pattern](#scalable-transfer-pattern)
   * [Frequency-Based Search for Public Transit](#frequency-based-search-for-public-transit)

**TL;DR** : depuis 2010, Google utilise [Transfer Patterns](2010-transfer-patterns-article.md) pour le calcul d'itis à base de transports publics. Le post indique comment scaler ça à l'échelle d'un pays en présentant deux nouveaux algos : [Scalable Transfer Patterns](https://ad-publications.cs.uni-freiburg.de/ALENEX_scalable_tp_BHS_2016.pdf) + [Frequency-Based Search for Public Transit](2014-frequency-based-search.md)

> What is the best way to get from A to B by public transit? Google Maps is answering such queries for over 20,000 cities and towns in over 70 countries around the world, including large metro areas like New York [...], and some complete countries, such as Japan or GB.

Google a un moteur de calcul d'itis à base de PT sur tout un pays. [Exemple de requête au Japon](https://www.google.fr/maps/dir/34.6966168,135.4919625/37.8862738,139.0902467/@36.286131,135.0544179,7z/data=!3m1!4b1!4m2!4m1!3e3)

> The technique [...] is the Transfer Patterns algorithm [1], which was created at Google’s engineering office in Zurich, Switzerland, by visiting researcher Hannah Bast and a number of Google engineers.

Ce moteur est implémenté en utilisant Transfer Patterns, depuis 2010.

> I am happy to report that this research collaboration has continued and expanded with the Google Focused Research Award on Next-Generation Route Planning.

De 2012 à 2015, Google [a travaillé avec un groupe de recheche](2012-to-2015-google-research-next-generation-route-planning.md) sur les évolutions à apporter au calcul d'itinéraires, ( *for both road and transit networks* ), avec 3 axes de recherche : *scalability, result diversity, real-time updates*.

> From the project’s numerous outcomes, I’d like to highlight two recent ones that re-examine the Transfer Patterns approach and massively improve it for continent-sized networks:

Environ 40 papiers ont émergé de ce groupe de recherche, dont : Scalable Transfer Patterns et Frequency-Based Search for Public Transit, objets du présent post de blog.

## Transfer Patterns

Le post donne une introduction avec les mains aux transfer patterns : en deux mots, au lieu de s'intéresser aux stops nécessaires pour faire un chemin, on se contente de s'intéresser aux lignes nécessaires.

La suite du post parles des améliorations apportées à transfer patterns.

## Scalable Transfer Pattern

Le papier = [Scalable Transfer Patterns](https://ad-publications.cs.uni-freiburg.de/ALENEX_scalable_tp_BHS_2016.pdf)

Notamment, il mentionne quelque chose qui m'est cher = le caractère central du mode de transport "principal" :

> Also, the individual transfer patterns become quite repetitive: For example, from any stop in Paris, France to any stop in Cologne, Germany, all optimal connections end up using the same few long-distance train lines in the middle, only the local connections to the railway stations depend on the specific pair of stops considered.

Scalable transfer patterns = partitionner le graphe en clusters ayant peu d'interface avec l'extérieur.

> Also, the individual transfer patterns become quite repetitive: For example, from any stop in Paris, France to any stop in Cologne, Germany, all optimal connections end up using the same few long-distance train lines in the middle, only the local connections to the railway stations depend on the specific pair of stops considered.

Chiffres sur l'Allemagne : sur 251,763 stops, seuls 4.32% sont des interfaces entre clusters (ci-dessous : "interface-stops", dénomination maison, FIXME = à mettre à jour si les papiers en mentionnent une autre).

Préprocessing en deux étapes :

- **step A** = calcul des transfer patterns intra-cluster avec deux objectifs : 1. savoir calculer un itinéraire à l'intérieur du cluster 2. savoir relier deux interface-stops d'un même cluster
- **step B** = calcul des transfer patterns inter-cluster, en ignorant tous les stops d'un cluster sauf ses interface-stops

> the researchers estimate [2] that Scalable Transfer Patterns for the whole world could be stored in 30 GB

##  Frequency-Based Search for Public Transit

Le papier = [Frequency-Based Search for Public Transit](2014-frequency-based-search.md)

TL;DR : exploitation des répétitions dans les horaires des transports pour accélérer le calcul des transfer patterns.

