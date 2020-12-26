# Glossaire

Terminologie utilisée dans les notes ou les papiers :
- `RN` = **Road Network** (réseau routier, par opposition au réseau de transport en commun)
- `PTN` = **Public Transport Network** (réseau de transport en commun, _schedule based_).
- Le terme général **Transportation Network** désigne à la fois les RN et les PTN.
- Le terme anglais **connection** semble utilisé dans les papiers sur les PTN comme équivalent du terme **route** pour RN : c'est un "itinéraire". Attention : ce terme N'A PAS comme en français une connotation de "changement de véhicule" (qu'on retrouve par exemple en français dans "inter-connexion"). Ainsi, on peut très bien parler de *direct connection* pour un trajet sans changement.
- `PCC` désigne **Plus Court Chemin**
- `EAP` désigne l' **Earliest Arrival-Problem** : `A@t → B`, sachant que je pars de `A` à `t`, à quelle heure au plus tôt puis-je arriver à `B` ?
- `EAT` désigne l' **Earliest Arrival Time** = l'heure la plus tôt à laquelle on peut arriver (en quelque sorte, la "solution" à l'EAP)
- `TDD` = **Time-Dependent Dijkstra** (= dijsktra sur time-dependent graph) dans un contexte d'algos travaillant sur PTN.
- `TED` = **Time-Expanded Dijkstra** (= dijksra sur time-expanded graph) dans un contexte d'algos travaillant sur PTN.
- `TP` = **Transfer Pattern**, dans le contexte où on en parle.
- `UW` = **Unrestricted Walking**, le fait de pouvoir marcher librement dans un calcul d'iti TC (par opposition au fait de limiter la marche piétonne aux seuls transferts définis dans le GTFS)
- `HL` = **Hub-Labeling**, technique permettant de connaître **très** rapidement le poids d'un plus court chemin entre deux nodes, utilisée pour l'UW.
