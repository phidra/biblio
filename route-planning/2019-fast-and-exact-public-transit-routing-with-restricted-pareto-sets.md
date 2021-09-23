# (ARTICLE) Fast and Exact Public Transit Routing with Restricted Pareto Sets

- **url** = [lien](https://epubs.siam.org/doi/abs/10.1137/1.9781611975499.5), [PDF](https://epubs.siam.org/doi/pdf/10.1137/1.9781611975499.5) (md5sum=`ed3a94cf0dfd4da392760f1b17e3dec6`), [copie locale](./LOCALCOPIES/1.9781611975499.5.pdf)
- **source** = conf: [ALENEX 2019](https://www.siam.org/conferences/cm/conference/alenex19), [proceedings](https://epubs.siam.org/doi/book/10.1137/1.9781611975499)
- **auteurs** = [Daniel DELLING](https://www.researchgate.net/scientific-contributions/Daniel-Delling-20515954), [Thomas PAJOR](https://www.researchgate.net/scientific-contributions/20516545-Thomas-Pajor), [Julian DIBBELT](https://www.researchgate.net/profile/Julian_Dibbelt), ce sont des noms connus (DIBBELT est un ancien du KIT, notamment), il semblerait qu'ils travaillent dorénavant chez Apple.
- **date de publication** = 2019
- **date de rédaction initiale de ces notes** = septembre 2021

* [(ARTICLE) Fast and Exact Public Transit Routing with Restricted Pareto Sets](#article-fast-and-exact-public-transit-routing-with-restricted-pareto-sets)
* [Abstract](#abstract)
* [Section 1 = Introduction](#section-1--introduction)
* [Section 2 = Preliminaries](#section-2--preliminaries)
* [Section 3 = restricted pareto set](#section-3--restricted-pareto-set)
* [Section 4 = Bounded McRAPTOR](#section-4--bounded-mcraptor)
   * [Approche n°1 = self-BMRAP](#approche-n1--self-bmrap)
   * [Approche n°2 = target-BMRAP](#approche-n2--target-bmrap)
   * [Approche n°3 = tight-BMRAP](#approche-n3--tight-bmrap)
      * [Step 1 = RAPTOR normal](#step-1--raptor-normal)
      * [Step 2 = plusieurs reverse RAPTORs](#step-2--plusieurs-reverse-raptors)
      * [Step 3 = un McRAPTOR final](#step-3--un-mcraptor-final)
* [Section 5 = Experiments](#section-5--experiments)
   * [5.1 quality of restricted Pareto-sets](#51-quality-of-restricted-pareto-sets)
   * [5.2 performance of Bounded McRAPTOR](#52-performance-of-bounded-mcraptor)
   * [5.3 other instances](#53-other-instances)
* [Section 6 = Conclusion](#section-6--conclusion)

**TL;DR** = algo dérivé de RAPTOR, très efficace pour calculer des itis multi-critères :

- position du problème = RAPTOR marche bien pour 2 critères (EAT + nombre de correspondances), mais sa variante multi-critère, McRAPTOR, est beaucoup plus lente, au point d'être inutilisable en pratique :
    - cause de lenteur n°1 = McRAPTOR est 4 fois plus lent que RAPTOR de base (i.e. même sans ajouter de critère supplémentaire) car on a des structures plus complexes à gérer
    - cause de lenteur n°2 = le merge des bags de labels (pour éliminer ceux qui sont dominés) est quadratique en le nombre de labels (qui sont d'autant plus nombreux qu'on a de critères)
- non-pertinence d'un Pareto-set exhaustif :
    - avec des critères supplémentaires, tous les itis du Pareto-set ne sont pas forcément intéressants pour l'utilisateur final
    - p.ex. un iti qui diminue le temps de marche à pied de 30 secondes mais retarde l'EAT de plusieurs heures sera sur le Pareto-set exhaustif... alors qu'il n'est jamais intéressant pour l'utilisateur final !
- définition du **restricted Pareto-set** :
    - un Pareto-set non-exhaustif, qui exclut les itis dégradant trop fortement l'EAT ou le nombre de correspondances
    - objectif = ne pas contenir de journeys avec de mauvais compromis (qui fait arriver 2 heures plus tard pour économiser 30 secondes de marche à pied)
    - introduction de slack-values `σarr` et `σtr` = tolérances sur la dégradation de l'EAT ou le nombre de correspondances par rapport au front de Pareto "anchor"
    - expérimentalement, les tolérances suivantes sont de bons comproimis : `σtr` entre `1` et `2` et `σarr` entre `30` et `60` minutes
    - représentation graphique intéressante = le front de Pareto restreint est défini par des rectangles autour des items du front de Pareto "anchor"
- famille d'algos **Bounded-McRAPTOR** pour calculer le restricted Pareto-set, qui utilisent RAPTOR et McRAPTOR comme briques élémentaires
- l'algo le plus intéressant est tight-BMRAP :
    1. RAPTOR normal = calcule les anchor-journeys = les itis de référence
    2. reverse RAPs = utilise le résultat de l'étape 1 pour calculer des bounds à chaque couple `{stop, round}`, qui serviront à pruner l'étape 3
    3. McRAPTOR classique sauf qu'on prune très très vite les trajets, grâce aux bounds calculés à l'étape 2
- note : tout comme RAPTOR, Bounded-McRAPTOR ne nécessite pas de preprocessing
- évaluation expérimentale — bien détaillée :
    - ils introduisent à la section 5.1 un outil permettant le classement des items d'un Pareto-set donné (permet de vérifier si les 5 meilleurs itis d'un Pareto-set exhaustif sont bien également sur le restricted Pareto-set, pour vérifier que Bounded-McRAPTOR "n'oublie" pas d'itis importants)
    - ils annoncent un résultat intéressant = il "suffit" de doubler le temps de calcul d'un RAPTOR classique pour calculer un Pareto-set qui optimise les 4 critères suivants :
        - EAT
        - nombre de transferts
        - temps de marche à pied
        - nombre de bus
    - les temps de calcul de tight BMRAP avec ces 4 critères est entre 30ms et 60ms (à comparer avec 18ms pour un RAPTOR normal), en fonction des `σarr` et `σtr` utilisés.
    - et on "gagne" des itis intéressants pour l'utilisateur : on passe de 2 itis renvoyés en moyenne (RAPTOR normal) à entre 10 et 15 (tight-BMRAP)
    - la **quality value** des itis calculés reste toujours bonne (i.e. le restricted Pareto-set ne skippe pas d'itis importants)

# Abstract

- le problème adressé est le problème MULTI-CRITÈRES ; un RAPTOR classique est déjà bi-critère (EAT + nombre de correspondances), mais là on parle d'encore plus de critère, comme p.ex. le temps de marche à pied
- le pareto-set (= front de Pareto) peut être très grand, vu que le troisième critère peut être continu (e.g. marche à pied)
- l'objectif du papier semble être de réduire le pareto-set :
    1. ils formalisent la définition d'un restricted pareto-set (de façon indépendante de l'algo utilisé pour le calculer)
    2. ils proposent un algo efficace, dérivé de McRAPTOR, pour calculer ce restricted pareto-set (en vrai, une famille d'algos = Bounded-McRAPTOR = BMRAP, le plus efficace est tight-BMRAP)
- résultats : pour calculer un pareto-set à 4 critères, leur algo n'est que deux fois plus lent qu'un RAPTOR à 2 critères (et va 65 fois plus vite qu'en calculant le pareto-set non-restreint), tout en conservant les trajets importants

# Section 1 = Introduction

- calculer un front de Pareto bi-critère est NP-hard en général (mais faisable pour les réseaux de transport publics)
- les techniques metionnées sont :
    - CSA
    - RAPTOR
    - TransferPatterns
    - TripBasedPublicTransitRouting
    - Graph-labeling (je ne connais pas bien cette technique, correspondant à Public-Transit-Labeling)
- ces techniques sont efficaces à 2 critères, mais beaucoup moins efficaces à plus de critères, pour deux raisons :
    - 1. l'algo doit maintenir des structures de données plus compliquées
    - 2. mécaniquement, l'ajout de critères augmente la taille du Pareto-set, possiblement fortement
- quelques critères évoqués, pour lesquels il peut être utile de calculer un front de Pareto :
    - temps de marche à pied
    - fiabilité des TC (certains TC seront moins fiables ?)
    - coûts
    - accessibilité (e.g. privilégier les stations accessibles aux PMR ?)
    - nombre de bus
- possibilité = ignorer des résultats légèrement inférieurs à d'autres sur le front de Pareto
    - ça marche bien si on l'applique À LA FIN de l'algo, pour raffiner un front de Pareto qui a été calculé exhaustivement
    - mais si on l'applique en cours d'algo, on peut très bien louper des algos importants in fine
    - et comme l'objectif est d'éviter de calculer un front de Pareto exhaustif, cette approche n'est pas très utile
- idée principale du papier :
    - calculer le front de Pareto bi-critère complet et exact
    - n'autoriser dans le front de Pareto multi-critère réduit QUE des itis qui ne dégradent pas trop l'EAT ou le nombre de transfers
    - Bounded McRAPTOR utilise RAPTOR et McRAPTOR comme briques de base
- tight-BMRAP = l'une des variantes de la famille Bounded-McRAPTOR:
    - fait une série de RAPTOR normaux pour précalculer des limites (bounds)
    - puis fait un McRAPTOR normal qui utilise les bounds pour pruner des trajets 
- ils annoncent un résultat intéressant = il "suffit" de doubler le temps de calcul pour calculer un RAPTOR qui optimise les 4 critères suivants :
    - EAT
    - nombre de transferts
    - temps de marche à pied
    - nombre de bus
- note : tout comme RAPTOR, pas besoin de preprocessing

# Section 2 = Preliminaries

- rappel des notations
- rappel de RAPTOR :
    - on fonctionne en rounds
    - à chaque round, on regarde toutes les routes qui passent par les stops améliorés au round précédent, et plus précisément le premier trip de la route qu'on peut attraper à ce stop
    - si on peut y arriver plus tôt grâce à ce trip, on met à jour les stops de chacun de ces trips
    - avant de passer au round suivant, on relaxe les transferts piétons de chaque stop amélioré
    - note : en faisant tourner l'algo à l'envers, on peut calculer des trajets qui "arrivent à telle heure", plutôt que des trajets qui "partent à telle heure"
    - possiblement en lien avec l'unrestricted-walking : il y a un paragraphe sur le fait que le graphe piéton ne soit pas constitué de cliques (ils contournent le problème en maintenant un set de labels piétons)
- rappel de McRAPTOR :
    - là où RAPTOR maintient pour chaque stop un label indiquant le meilleur temps d'arrivée en k legs TC, McRAPTOR maintient pour chaque couple `{stop,round}` un **BAG** de labels, où tous les labels du bag sont sur le front de Pareto (i.e. aucun n'est dominé)
    - chaque label est un tuple contenant N éléments (où N est le nombre de critères à optimiser, sauf le nombre de correspondances, déjà pris en compte par la notion de round)
    - la façon dont on scanne les routes semble également modifiée, pour prendre en compte les bag de labels
    - (on généralise le fait de trouver le meilleur trip, en pouvant trouver LES meilleurs trips, un pour chaque label dans le bag)
    - cause de lenteur n°1 = McRAPTOR est 4 fois plus lent que RAPTOR de base (i.e. même sans ajouter de critère supplémentaire) car on a des structures plus complexes à gérer
    - cause de lenteur n°2 = le merge des bags de labels (pour éliminer ceux qui sont dominés) est quadratique en le nombre de labels (qui sont d'autant plus nombreux qu'on a de critères)
    - du coup, McRAPTOR peut vite devenir trop lent pour être utilisé en pratique

# Section 3 = restricted pareto set

- le front de Pareto "exhaustif" contient TOUS les trajets non-dominés
- problème : il suffit d'améliorer le temps de marche à pied de 30 secondes pour inclure dans le front de Pareto un trajet qui dégrade l'EAT de plusieurs heures... Or, c'est un trajet que l'utilisateur n'empruntera jamais en pratique, car rallonger son trajet de plusieurs heures pour gagner 30 secondes de marche à pied n'est jamais intéressant pour un utilisateur...
- ils donnent un exemple illustratif, où on multiplie par trois le temps de trajet total (avec un trajet alambiqué, du coup), pour ne marcher que 250 mètres au lieu de 500 mètres...
- typiquement : une fois le pareto-set obtenu, les itis du front de Pareto sont classés, et on ne les présente pas tous à l'utilisateur
- (NdM = une autre façon de voir les choses, pertinente dans le cadre de cet article : en calculant le Pareto-set exhaustif, on a calculé beaucoup d'itis inintéressants)
- un point intéressant = l'approche intuitive consistant à essayer de pruner les labels en cours d'exécution de l'algo ne marche pas
    - on aurait pu essayer de dégager un label L2 qui dégrade fortement un critère et n'améliore que marginalement le critère qui le place sur le front de Pareto
    - ça ne marche pas 1 = on n'a pas de garantie sur les labels qui "survivront" en fin d'exécution
    - ça ne marche pas 2 = c'est très dépendant de détails d'implémentation de l'algorithme (deux algos différents renverront un front de Pareto restreint différent)
- à l'inverse, l'approche de l'article garantit que deux algos différents renverront un front de Pareto restreint identique
- objectif = dégager les itis qui offre un compromis inintéressant (et auraient été dégagés lors de la phase de post-processing du front de Pareto, donc jamais montrés à l'utilisateur final)
    - on commence par calculer le front de Pareto "anchor", i.e. le front de Pareto classique à deux critères EAT+transfers (les itis sur ce front de Pareto sont les `anchor-journeys`)
    - notation : `J*` est un **anchor-journey**, `τarr(J*)` est son EAT, et `tr(J*)` est son nombre de trips
    - principe = chaque itinéraire sur le restricted-pareto-set ne doit pas dégrader significativement `τarr(J*)` et `tr(J*)`
    - étant donné un iti `J` (sur le front de Pareto exhaustif avec plus que deux critères), son `corresponding anchor-journey` est :
         - parmi les itis `J*` qui ont autant de trips ou moins de trips que `J`, le `corresponding` est celui qui a le PLUS de trips
         - ça peut être le même iti (`J = J*`), dans ce cas, `J` est également sur le front de Pareto `anchor`
         - ça peut être des itis différents, p.ex. `J*` a autant de correspondance, et arrive plus tôt que `J` (car il a ignoré le critère supplémentaire qui a fait que `J` est sur le front de Pareto exhaustif)
    - À CLARIFIER : mieux comprendre pourquoi `J*` est toujours unique, et comment le trouver...
- derrière, en gros, le `restricted pareto-set`, c'est un intermédiaire entre `J*` et `J`, dans lequel tout iti restreint est à au plus `σarr` et `σtr` de son `corresponding-anchor-journey`)
    - (dit autrement, `J` ne "dégrade" son `corresponding anchor-journey` que de `σarr` et/ou `σtr`)
    - définir `σarr` (resp. `σtr`) entre `0` et `∞` permet de positionner le front de Pareto restreint entre le front de Pareto anchor et le front de Pareto exhaustif
- représentation graphique intéressante = quand on graphe le nb de transfers vs. l'EAT, le front de Pareto restreint est défini par des rectangles autour du front de Pareto anchor

# Section 4 = Bounded McRAPTOR

Il y a plusieurs types d'algos dans la famille, selon qu'ils permettent `σtr ≠ +∞` (i.e. qu'ils respectent le caractère restreint du Pareto-set sur le critère du nombre de transfers), et selon qu'ils calculent le restricted Pareto set exact (ou un superset un peu trop large, qui contient le restricted pareto-set).

## Approche n°1 = self-BMRAP

En gros, c'est un McRAPTOR classique, sauf que :

- on maintient le meilleur EAT au stop cible `τ*` (on commence avec `τ* = +∞`, et on le mets à jour à chaque round)
- en cours d'algo, on peut pruner les trajets lorsque leur arrivée à un stop intermédiaire est déjà supérieur à `τ* + σarr`

Problème 1 = pour pruner efficacement, ça n'est pas le `τ* `en cours d'algo qu'il faudrait utiliser, mais le meilleur EAT final (or, on ne le connaît pas encore). En effet, si le meilleur EAT n'est calculé qu'au dernier round, on aura passé tous les rounds suivant à ne pruner qu'avec un EAT trop tardif, et donc à garder dans le front de Pareto des itis qui auraient été à dégager avec l'EAT final.

Problème 2 = ça ignore complètement `σtr`, et ne se concentre que sur `σarr` -> on ne calcule pas vraiment le restricted-pareto-set, mais plutôt un surensemble.

## Approche n°2 = target-BMRAP

En gros, c'est la même approche que self-BMRAP, mais qui corrige son problème n°1, en lançant d'abord un RAPTOR classique pour connaître le meilleur EAT (ou plutôt, le set de meilleurs EAT en fonction du nombre de transfers, i.e. le front de Pareto classique), puis en utiliant ce(s) meilleur(s) EAT pour pruner les trajets au cours de l'exécution du McRAPTOR.

Le pruning devient exact pour le rapport à `σarr`, mais on n'est toujours pas capable de prendre en compte `σtr`.

## Approche n°3 = tight-BMRAP

Ce troisième algo est le meilleur, il produit le restricted Pareto set exact, et notamment il gère le trip-slack (i.e. ça prend en compte `σtr`), tout en étant le plus rapide.

Il suit trois étapes :

1. RAP normal
2. plusieurs reverse RAPs
3. un MRAP final

(à noter que les évaluations de la section 5 montrent que les étapes 2 et 3 consomment moins de temps que l'étape 1 du RAP classique)

### Step 1 = RAPTOR normal

RAP normal = on récupère les anchor-journeys (i.e. les journeys sur le front de Pareto "classique"). De plus, le search-space contient pour chaque stop son propre front de Pareto (i.e. ses earliest-arrival-time pour chaque round) ; en effet, même si on ne s'intéresse qu'à l'EAT au stop cible, c'est un sous-produit de RAPTOR (un peu comme le shortest-path tree est un sous-produit de dijkstra, même si on ne s'intéresse qu'au node cible). Et tout comme dans Dijkstra, les stops qui n'ont pas été visités ont leur EAT à chaque round setté à `+∞`.

### Step 2 = plusieurs reverse RAPTORs

À ce stade, on va calculer les time-bounds exact à chaque stop. Ma compréhension à froid (i.e. avant de lire l'article), c'est quelque chose comme "si l'EAT au stop cible est 10:00:00, la bound au stop-cible est 10:10:00, et donc à tel stop sur la ligne précédente, compte-tenu qu'il me faut 40 minutes pour rejoindre le stop cible, il est inutile que j'arrive après 09:40:00".

Je me contente de survoler la section, mais je confirme que ça a l'air d'être ça.

Note : sans que j'aie regardé les détails, on lance `|J*|` RAPTOR reverse (ce qui, si on tient compte du RAPTOR initial pour trouver `J*`, porte le nombre d'étapes préliminaire au McRaptor final à `1 + |J*|`)

On pourrait penser que du coup ça va être beaucoup plus long, mais en fait, tight-BMRAP est environ 2 fois plus lent que RAP classique

Il y a une illustration graphique de comment lancer les reverse RAP pour calculer les bounds à chaque stop.

### Step 3 = un McRAPTOR final

On lance un MCRaptor qui prune les labels de chaque stop s'ils dépassent les bounds définis à chaque stop par l'étape 2.

Un point à ne pas rater, c'est que c'est valide pour un round donné (i.e. les labels et les bounds à un stop donné sont définis pour un round X, et ils seront différents au round X+1)

Le papier mentionne (mais je n'ai pas regardé en détail) des moyens d'accélérer les reverse raptor servant à calculer les bounds.

# Section 5 = Experiments

Deux volets : performances de l'algo, et qualité du restricted Pareto-set.

Les 4 critères utilisés pour le restricted Pareto-set sont :

- EAT
- nombre de correspondances
- durée de marche totale
- nombre de bus

Tous les GTFS qu'ils ont utilisés sont publiquement accessibles, ils travaillent sur deux jours du GTFS (lundi et mardi), et calculent les footpaths eux-mêmes.

Globalement, ils sont assez détaillés sur leur setup expérimental. Un point intéressant = ils choississent leurs stops source et target en fonction de leur "popularité" (i.e. il y a beaucoup de trips qui servent ce stop).

Rappel : l'objectif du restricted Pareto-set est de ne pas contenir de journeys avec de mauvais compromis (genre qui rajoutent une heure de marche à pied pour faire arriver 2 minute plus tôt).

## 5.1 quality of restricted Pareto-sets

Ils détaillent comment à partir d'un Pareto-set donné (peu importe comment il a été obtenu), ils classent les journeys, avec comme objectif de garder les `k` meilleurs du Pareto-set *complet*, pour voir s'ils sont également dans le restricted Pareto-set ou pas.

Pour classer les itis du front de Pareto, ils s'appuient sur un travail antérieur : Computing Multimodal Journeys in Practice, [lien](https://www.researchgate.net/publication/291055804_Computing_Multimodal_Journeys_in_Practice), [PDF](https://i11www.iti.kit.edu/extra/publications/ddpww-cmjp-13.pdf) par Daniel Delling, Julian Dibbelt, Thomas Pajor, Dorothea Wagner, Renato F. Werneck

J'ai pas regardé en détail, mais il semble s'agir de généraliser la notion de domination pour classer plus bas les itis "plus dominés que d'autres".

Une fois qu'ils ont cet outil de ranking d'un front de Pareto, ils calculent les 5 meilleurs itis du Pareto-set complet, et comparent avec les 5 meilleurs itis du Pareto-set restricted (pour voir si le restricted a sucré des itis qui étaient intéressants), ça donne la notion de **quality value** d'un restricted Pareto-set.

Ils évaluent l'influence de `σtr` et `σarr` sur la **quality-value**, et sans surprise, plus ils sont grands, meilleure est la qualité (i.e. moins le restricted Pareto-set est contraint, plus il a de chance de contenir les meilleurs itis du Pareto-set complet).

Plus en détail : le meilleur `σtr` d'après leurs expériences est entre `1` et `2`, et c'est un bon compromis de fixer `σarr` entre `30` et `60` minutes, ce sont les valeurs qu'ils retiennent pour la suite de leurs expériences.

## 5.2 performance of Bounded McRAPTOR

Dans cette section, ils se limitent à Paris.

Les temps de calcul pour le meilleur algo (tight BMRAP) sur les 4 critères est entre 30ms et 60ms (à comparer avec 18ms pour un RAPTOR normal), en fonction des `σarr` et `σtr` utilisés.

Le nombre de journeys moyens renvoyés passe de 2 pour un RAPTOR classique à entre 10 et 15 pour un tight BMRAP.

Autre point important = même si c'est un peu flou, la **quality value** des itis calculés reste toujours bonne (i.e. le restricted Pareto-set ne skippe pas d'itis importants).

## 5.3 other instances

Sur ces instances plus petites que Paris, ça va pas mal plus vite (RAPTOR entre 3 et 13 ms, tight BMRAP entre 19 et 38 ms), la quality value reste bonne

> From these experiments we conclude that Tight-BMRAP is a fast approach to computing multi-criteria restricted Pareto sets that retaining almost all important journeys of the full Pareto set. \
> Moreover, it scales very well when additional optimization criteria are added.

# Section 6 = Conclusion

Ils proposent 4 pistes pour la suite du travail :

- voir à quel point la qualité du restricted Pareto-set est robuste aux modification de paramètres de l'algo de ranking
- s'intéresser à de la longue distance (là, ils ont fait du TC urbain, alors qu'ils pourraient faire aussi du timetable national)
- définir les slack values `σarr` et `σtr` dynamiquement, en fonction des anchor-journeys
- combiner l'approche avec des speedup techniques comme HypRAPTOR pour aller encore plus vite
