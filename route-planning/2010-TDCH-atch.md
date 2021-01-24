# (ARTICLE) Time-Dependent Contraction Hierarchies and Approximation

- **url** = [PDF](https://algo2.iti.kit.edu/download/sea10_atch.pdf), (md5sum=`e09a9a3ed6416b2540b666caff79a2ab`), [copie locale](./LOCALCOPIES/sea10_atch.pdf)
- **source** = [SEA 2010](https://dblp.org/db/conf/wea/sea2010.html)
- **auteurs** = Gernot Veit Eberhard BATZ, Robert GEISBERGER, Sabine NEUBAUER, and Peter SANDERS (Karlsruhe Institute of Technology)
- **date de publication** = 2010
- **date de rédaction initiale de ces notes** = juin 2020

# Transfert de mes notes brutes

## Section 1 = Introduction

* Semble introduire le terme "ATCH" = approximated TCH
* L'objectif du papier est de réduire le volume des graphes (le papier ne présente pas TCH, mais l'une de ses améliorations).
* Deux types de requêtes :
    - time-dependent routing (= ce qui nous intéresse : trouver le chemin permettant d'arriver le plus tôt dans un graphe time-dependent)
    - travel time profiles = quel sera mon temps de parcours du trajet A->B sur les prochaines 24 heures, en fonction de mon heure de départ (e.g. pour choisir la meilleure heure à laquelle partir). C'est une généralisation du time-dependent routing (qui calcule le temps de trajet pour UN point donné dans le temps, alors que le profile le calcule pour N points)
* Approximation des fonctions linéaires par morceau (représentant le poids d'un edge en fonction de l'heure) : n'empêche pas l'exactitude du résultat !
* Dans un TCH de base, il y a beaucoup de redondance d'information.
* Approximation des shortcuts, mais edges "réels" conservés exacts.
* bidirectional search dans cette Approximated-TCH nous renvoie un CORRIDOR de shortcuts.
* Ça limite la recherche aux shortcuts du corridor (on est sûr que la réponse exacte recherchée est dans le corridor).
* Derrière, on n'a plus qu'à expand les shortcuts du corridor, et rechercher le chemin exact dans les shortcuts expanded (qui, eux, ne sont pas approximés)
* (Pour calculer les travel-time profiles, on récupère le CORRIDOR aussi, puis on le CONTRACTE pour faire les différents calculs permettant de construire le travel-time profile.)
* En acceptant d'être un peu moins précis que l'exactitude parfaite, on accélère grandement les calculs.
* EA = earliest arrival = la généralisation du problème du plus court chemin quand le poids des edges est time-dependent (en effet, le chemin à renvoyer n'est plus nécessairement le plus court)


## Section 2 = Preliminaries

Les TTFs utilisées pour TCH et leurs opérations :
* notamment : les TTF suivent le FIFO property (on NE PEUT PAS arriver plus tôt en reportant son départ)
* notation : |f| est la "complexité" d'une TTF, i.e. le nombre de "morceaux" dans la fonction par morceaux représentant la TTF

Avec une TTF f, on peut :

1. *evaluation* = calculer f(τ)

2. *linking* = composer des TTFS :
* si on a deux edges adjacents :
    - (u,v) de TTF f, qui à τ associe f(τ)
    - (v,w) de TTF g, qui à τ associe g(τ)
* on veut calculer la TTF `g∘f` du "chemin" `<u,v,w>` :
    - à τ, g∘f associe f(τ) + g(τ+f(τ))
    - f(τ) est le temps de parcours de (u,v) : c'est au temps τ qu'on cherche à aller de u à v
    - g(τ+f(τ)) est le temps de parcours de (v,w) : c'est au temps τ+f(τ) qu'on cherche à aller de v à w
* TL;DR : le linking, c'est le fait de calculer la TTF composée de 2 edges
* calculable en O(|f| + |g|)
* la complexité de la composée g∘f est au max la somme des composées
* le linking est une opération associative : f∘(g∘h) = (f∘g)∘h

3. *minimum* = merge :
* à partir de deux edges E et E' parallèles, i.e. qui ont même extrêmités ut et v (et leurs TTFs f et f')
* on MERGE E et E' en un seul edge E" préservant les plus courts chemins
* sa TTF f" associe à tout τ : min{f(τ), f'(τ)}

Notes :
* dans un graphe time-dependant, pour un même couple {source s ; target t}, en fonction de l'heure de départ τ, il peut y avoir PLUSIEURS plus courts chemins/temps de trajets
* travel-time profile (TTP) = pour une paire {s,t} de noeuds donnée, c'est la TTF qui à tout τ associe le plus court chemin de s à t (et son temps de parcours)
* si on appelle f cette TTF (ou plutôt ce TTP), on peut définir une fonction `arrf` pour calculer l'heure d'arrivée en fonction de τ : `arrf` associe à tout heure de départ τ l'heure d'arrivée τ+f(τ)
* inversement, la fonction réciproque qui à une heure d'arrivée donnée associe l'heure (ou les heures) à laquelle il faut partir pour l'atteindre est notée : depf = (arrf)⁻¹

NdM : dijkstra classique = applicable au problème du PLUS COURT CHEMIN, i.e. pour un graphe statique
* dans un dijkstra classique, lorsqu'on visite un noeud, on peut modifier son label (tentative_distance)
* ce label est la plus courte distance connue JUSQU'ICI nous permettant de rejoindre ce noeud (initalement, le label vaut +∞)
* lorsqu'on relaxe un edge (u,v) :
    - soit le label de v est déjà plus court (on a rejoint v par un chemin plus long que celui du label)
    - soit on a trouvé un chemin plus court que celui du label, on remplace alors le label par  d(u) + len(u,v)
    - on synthétise ça en disant que le label d'un noeud v lorsqu'on relaxe l'edge (u,v) prend pour valeur : min{d(v), d(u) + len(u,v)}
* lorsque le noeud en question finit par être en tête de la priority_queue, son label devient sa plus courte distance DÉFINITIVE (le noeud est settled)


----



	Time-Dependent Dijkstra = généralisation de dijkstra au problème de l'EARLIEST ARRIVAL, i.e. pour un graphe TD
		les valeurs associées aux noeuds ne sont plus des distances, mais des heures d'arrivée
		lorsqu'on relaxe un edge (u,v), on prend en compte l'heure d'arrivée à u pour savoir quel poids ajouter à d(u) (en effet, celui-ci est une TTF)
		en dijkstra classique, la nouvelle tentative_distance de v valait : min{d(v), d(u) + len(u,v)}
		en TD-dijkstra, la nouvelle tentative_distance de v vaut          : min{d(v), d(u) + len[u](u,v)}   (len(u,v) dépend de l'heure d'arrivée à v = la "distance" de u !)
		du coup, le noeud de départ n'a plus pour label 0, mais a pour label τ, le departure_time
	Profile Search = renvoie le profil d'un itinéraire {s,t} (i.e. le temps de parcours de l'iti en fonction de l'heure de départ)
		dans le dijkstra, au lieu que le label soit un scalaire représentant la plus petite distance (ou l'heure d'arrivée la plus tôt) au noeud...
		... le label est une fonction dépendant du temps : à tout temps τ, elle associe l'heure d'arrivée la plus tôt (connue jusqu'ici) au noeud
		en gros, on généralise le scalaire en une fonction dépendant du temps τ de departure_time depuis la source
		pour relaxer un edge (u,v), on merge la TTF actuelle de v avec celle de la composition TTF(edge:u→v) ∘ TTF(u)
		(si v n'a pas encore de TTF au moment où on arrive dessus (pour la première fois, donc), on l'initialise à la TTF constante infinie)
	Interval Search = même principe que ProfileSearch, mais ne travaille que sur les bornes inf/sup des TTFs, ce qui rend les calculs BEAUCOUP moins lourds
		les labels utilisés dans dijkstra sont des 2-uple [inf, sup]
		si un noeud N a pour label [inf, sup], on sait qu'on NE PEUT PAS atteindre ce noeud avant inf, et après sup
		le label initial de chaque edge est [0,+∞]
		supposons qu'un noeud v ait pour label [inf(v), sup(v)]...
		... alors lorsqu'on relaxe un edge (u,v), si (u,v) nous permet d'arriver à v AVANT inf(v), on modifie le label de v
		dit autrement, si inf(u) + inf(u,v) < inf(v), alors inf(v) prend comme nouvelle valeur inf(u) + inf(u,v)
		à noter qu'on parle de 2-uple [inf,sup] aussi bien pour un noeud v (auquel cas, c'est son dikstra-label) que pour un edge (u,v) (auquel cas c'est une approximation de sa TTF)
	Corridor :
		sous-graphe du graphe contracté, dans lequel un iti de s à t est possible (donc : un corridor est propre à un iti {s,t})
		l'intérêt d'un corridor est qu'il y a BEAUCOUP moins de noeuds que dans le graphe contracté initial
		on peut donc y faire des recherches "exhaustives" (i.e. profile-search ou TD-dijkstra) de façon plus efficace
		si je comprends bien : 1. on utilise interval search pour réduire l'espace de recherche en calculant un corridor 2. on fait un TD-dijkstra dans le corridor
	Earliest Arrival queries on TCH :
		step 1 = on fait un bidirectinal search modifié :
			forward search = TD-dijkstra
			backward search = interval search
			----------------------------------------
			à l'issue de la step1, on a trouvé des candidates (i.e. des meeting points)
			par ailleurs, l'interval search en backward a permis d'exclure des nodes/edges qui ne sont pas sur le plus court chemin à coup sûr
			dit autrement : l'interval search en backward a permis de restreindre les nodes/edges possibles -> il a créé un corridor
		step 2 = on relance un TD-dijkstra depuis CHAQUE candidate (en initialisant son τ à la valeur donnée par le forward search)
			détail crucial : ce TD-dijkstra est LIMITÉ aux nodes/edges du corridor (vu que seuls eux peuvent accueillir le plus court chemin)
		QUESTION : quand décide-t-on qu'on a suffisamment de meeting-points ? Ou bien on ne fait la step2 que quand on a trouvé TOUS les meeting points possibles ?
