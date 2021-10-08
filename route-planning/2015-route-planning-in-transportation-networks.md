# (ARTICLE) Route Planning in Transportation Networks

- **url** = [PDF](https://arxiv.org/pdf/1504.05140.pdf) (md5sum=`48e430f2e679428473c1529d96a5388b`), [copie locale](LOCALCOPIES/1504.05140.pdf)
- **source** = c'est la mise à jour d'une [publication de recherche de Microsoft](https://www.microsoft.com/en-us/research/publication/route-planning-in-transportation-networks/)
- **auteurs** = Hannah BAST, Daniel DELLING, Andrew GOLDBERG, Matthias MÜLLER-HANNEMANN, Thomas PAJOR, Peter SANDERS, Dorothea WAGNER and Renato F. WERNECK
- **date de publication** = 2015
- **date de rédaction initiale de ces notes** = 28 octobre 2020

**TL;DR** = un très bon survey sur l'état de l'art du calcul d'itinéraires en 2015 :
- PRÉAMBULE IMPORTANT : garder à l'esprit que les techniques récentes n'apparaissent pas dans ce survey.
- Le survey distingue trois catégories de besoins :
    - road network = raisonnablement adressé aujourd'hui y compris à échelle continentale. Il n'y a pas un seul algo qui emporte le consensus car chacun a ses forces et ses faiblesses selon le problème à adresser.
    - public transportation network = problème difficile : ok à l'échelle d'une ville, mais difficile à l'échelle d'un pays (ou d'un continent), quelques algos sortent du lot.
    - multimodal (qui mélange schedule-based et unrestricted) = très difficile, même à l'échelle d'une ville.
- Le papier est très dense : 261 références -> les techniques mentionnées ne sont qu'effleurées.
- En plus du fait de présenter très brièvement les techniques, le papier est surtout intéressant car il les compare les unes aux autres :
    - toute la section 3 compare les performances des algos sur RN
    - la sous-section 4.5 compare l'état de l'art (à date de rédaction de l'article) des algos sur PTN
- Je n'ai fait que survoler mais il y a _beaucoup_ d'infos intéressantes pour le calcul d'itinéraire sur RN dans les sections 2 et 3 (e.g. turn-penalties, ou itinéraires alternatifs).


* [(ARTICLE) Route Planning in Transportation Networks](#article-route-planning-in-transportation-networks)
   * [section 1 = introduction](#section-1--introduction)
   * [section 2 = shortest paths algorithms](#section-2--shortest-paths-algorithms)
   * [section 3 = route planning in road networks](#section-3--route-planning-in-road-networks)
   * [section 4 = journey planning in public transit networks](#section-4--journey-planning-in-public-transit-networks)
   * [section 5 = multimodal journey planning](#section-5--multimodal-journey-planning)
   * [section 6 = final remarks](#section-6--final-remarks)

## section 1 = introduction

Parmi les auteurs, on note les usual suspects (Bast, Delling, Sanders, Werneck...), et le fait que plusieurs GAFAM sont intéressés par le sujet : Google, Microsoft et Amazon.

RN correctement adressé : query time au niveau de la ms même sur réseau continental

PTN : plus dur car naturellement time-dependent, moins structuré, et multi-critère. Queries sur un réseau d'une ville ok, mais à l'échelle du pays, il faut des simplifications 

multimodal = encore plus difficile, on doit approximer même si on ne regarde que sur des villes.

Mentionne un ancien papier "Engineering route planning algorithms", en précisant qu'il a beau être assez récent, il est déjà obsolète.

Les algos nettement dominés par d'autres sur les variables ci-dessus ne sont pas mentionnés. (e.g. si un algo A est rapide et robuste au traffic temps-réel, B ne sera pas mentionné même s'il est aussi rapide que A, mais ne sait pas traiter le traffic)

Seuls les algos publiés avant janvier 2015 sont analysés -> les algos récents sont invisibles.

## section 2 = shortest paths algorithms

D'après l'introduction : focus sur les graphes statiques.

Pas d'algo définitif pour le route-planning sur RN, car on fait un trade-off entre plusieurs variables :
- preprocessing time
- space requirements
- query time
- robustesse au real-time traffic

Dans l'état de l'art aujourd'hui, les algos ne sont plus approximés : les PCC renvoyés sont exacts.

Le papier mentionne des améliorations des perfs théoriques de Dijkstra en fonction de la priority queue employée.

Bellman-Ford est en pratique souvent plus rapide que Dijkstra, même si plus lent asymptotiquement.

Phrase de synthèse intéressante sur les **techniques hiérarchiques** : *Sufficiently long shortest paths eventually converge to a small arterial network of important roads, such as highways.*

**Bounded-hop techniques** = précalculer les distances entre des paires de noeuds bien choisis. transit-node routing en fait partie. Hub-labelling aussi (technique la plus rapide à la query au moment de la rédaction de l'article : quelques centaines de ns, au détriment d'un espace de stockage important).

La section 2.7.3 est dédiée à la prise en compte du traffic temps-réel, avec plusieurs stratégies :
- mettre à jour (plutôt que rebuilder) la donnée préprocessée
- garder le preprocessing original, mais l'utiliser autrement à la query
- faire tout le preprocessing de façon metric-independent
- faire une partie du preprocessing de façon metric-independent et refaire la partie metric-dependent quand il y a des updates (stratégie retenue WCH/CCH)

La section 2.7.4 est dédiée au traffic histo. Bidirectional search plus compliqué si time-dependent, vu qu'on ne connaît pas l'heure d'arrivée, nécessaire pour connaître le poids des arcs.

La section 2.7.5 est dédiée au multi-critères. Une idée intéressante est d'utiliser une combinaison linéaire pour merger deux critères (et ainsi revenir sur un dijkstra mono-critère).

## section 3 = route planning in road networks

Très intéressant graphique classant les performances des diverses techniques (query-time vs. preprocessing-time) : pareto-set des techniques de routing.

Notion de _realistic setting_ : turn-restrictions, turn-costs, prise en compte du traffic, user-preferences, ...

Autre intéressant graphique classant le query-time en fonction de l'écart entre source et target (mesurée comme le dijkstra rank de la target = l'index du moment où on settle la target si on lance un dijkstra "normal" depuis la source). Point rigolo : TNR est plus rapide pour les itis longues distances que pour les itis courtes distances (vu que les courtes distances sont traitées par un graph-search au lieu d'utiliser les transit nodes).

Section 3.2 = beaucoup de références pour trouver des alternatives "raisonnables" au shortest path : _short, smooth, and significantly different from the shortest path_.

## section 4 = journey planning in public transit networks

Modélisation d'un PTN par un _time-expanded graph_ ou un _time-dependent graph_.

Sur PTN, on a plus de problèmes "naturels" que sur RN : en plus de l'EAP, range-problem, number of transfers, total price, ...

Côté _uncertainty_, une étude montre que si on la prend pas en compte, l'expérience utilisateur est pas ouf.

Les transfer patterns calculés sur des données sans delay restent valable avec delay dans 99% des cas.

CSA, Frequency-based search for public transit, ACSA...

_Accelerating time-dependent multicriteria timetable information is harder than expected_

ACSA = Accelerated CSA, _excellent query and preprocessing times on country-sized instances_.

CH sur un time-expanded graph créée trop de shortcut pour être applicable.

Notion de robustness d'un trajet aux delays.

La sous-section 4.5 = experiments and comparison est la plus intéressante pour évaluer les techniques les unes par rapport aux autres.

La comparaison exacte n'est pas si facile, car pour des mêmes données, chaque groupe de recherche les interprètent différemment.

À date de rédaction de l'article, le plus grand réseau métropolitain est [celui de Londres](https://data.london.gov.uk/topic/transport) : 21k stops, 2k lines, 133k trips, 46k footpaths, et 5M connections par jour.

Les données du réseau de train de l'Allemagne (assez utilisées par Bast et ses équipes) proviennent apparemment de sources propriétaires.

Côté algos basés sur dijkstra, TDD est plus rapide que TED, car le graphe est moins gros.

Du côté des algos sans preprocessing, RAPTOR et CSA sont plus rapides que ceux basés sur dijkstra, et CSA est un peu plus rapide que RAPTOR.

En terme de vitesse, Transfer-Patterns explose tout (~10 fois plus rapide que RAPTOR/CSA), mais au prix de lourds preprocessing : > 500h pour l'Allemagne, et encore, c'est en utilisant le frequency-based model pour exploiter la répétabilité. En revanche, les approximations du papier original (hubs + 3-legs heuristics) ne sont du coup plus appliquées car plus nécessaires.

Un point intéressant : si on ne s'intéresse qu'à l'EAT et au nombre de transfers, le front de Pareto n'est pas trop grand, car les deux critères sont corrélés : un trajet avec peu de transfers des chances d'être plus rapide qu'un trajet avec beaucoup de transfers.

## section 5 = multimodal journey planning

Approche générale = merger les différents graphes représentant les modes de façon individuelle.

Plus en détail, 3 approches :
- optimisation d'un seul coût, qui peut être la combinaison linéaire de plusieurs critères (somme des durées des différents modes, évitement des transferts, prix, ...)
- label-constrained shortest path problem (LCSPP) = les itinéraires calculés obéissent à certaines contraintes (typiquement, d'enchaînement de modes, pour exclure/inclure certains patterns). Accélérable (Access-Node Routing a un query-time en ms sur un réseau intercontinental, [SDALT sur IDF de l'ordre de dizaine/centaine de ms](2012-label-correcting-for-multi-modal.md), CH...). Ici aussi, un seul itinéraire est calculé (pas de pareto-set).
- calcul multicritère avec des critères choisis pour privilégier des trajets qui utilisent des modes différents (exemple [MCR = extension de RAPTOR](2013-computing-multimodal-journeys-in-practice.md), avec un query time qui descend péniblement sous la seconde, jusqu'à la dizaine de ms)

## section 6 = final remarks

_Experiments on real data are very important, as properties of production data are not always accurately captured by simplified models and folklore assumptions._

_careful engineering is essential to unleash the full computational power of modern computer architectures : CSA, [...], RAPTOR achieve their good performance by carefully exploiting locality of reference and parallelism_ 

Algorithmes utilisés par quelques acteurs :
- TomTom uses a variant of Arc Flags with shortcuts to perform time-dependent queries.
- Microsoft’s Bing Maps use CRP for routing in road networks.
- OSRM, a popular route planning engine using OpenStreetMap data, uses CH for queries.
- The Transfer Patterns algorithm has been in use for public-transit journey planning on Google Maps since 2010.
- RAPTOR is currently in use by OpenTripPlanner.

Multimodal : _Systems like Rome2Rio provide a simplified first step, but a more useful system would take into account real-time traffic and transit information, historic patterns, schedule constraints, and monetary costs._
