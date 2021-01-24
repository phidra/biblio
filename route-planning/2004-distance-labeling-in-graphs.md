# (ARTICLE) Time-Dependent Contraction Hierarchies and Approximation

- **url** = [PDF](http://www.wisdom.weizmann.ac.il/~ranraz/publications/Plabeling.pdf), (md5sum=`d010a04bdec3879911cc853a311c6138`), [copie locale](./LOCALCOPIES/Plabeling.pdf)
- **source** = [SODA 2001](https://dl.acm.org/doi/proceedings/10.5555/365411)
- **auteurs** = Cyril GAVOILLE (LABRI), David  PELEG, Stéphane PÉRENNÈS, Ran RAZ
- **date de publication** = 2004
- **date de rédaction initiale de ces notes** = mars 2020

# Contexte

Je m'intéresse au papier suivant pour faire des itis TC : [Fast Public Transit Routing with Unrestricted Walking through Hub Labeling](https://hal.inria.fr/hal-02161283) ; il utilise hub-labeling, et je m'intéresse donc au labeling de graphe.

# Transfert de mes maigres notes brutes

**Principe du hub-labeling** :
* on ajoute à chaque noeud un label (chaîne binaire contenant de l'information sur le noeud vis-à-vis du graphe).
* grâce aux labesl, il devient possible de calculer la distance entre deux noeuds x et y, UNIQUEMENT grâce à leur label

Ma compréhension du truc = c'est pas très compliqué à faire "en général", toute la difficulté est de trouver un labelling efficace.

Une implémentation inefficace possible est :
* pour chaque noeud N d'un graphe G contenant 6000 noeuds, son label est la chaîne suivante : "1=32|2=29|3=119|...|6000=85"
* en gros, le label du noeud contient sa distance à tous les autres noeuds du graphe
* du coup, pour calculer la distance entre x et y, il suffit d'aller la regarder dans le label :-D
* cette solution est en O(n) en espace : chaque label a une taille linéaire en le nombre de noeuds du graphe
* cette solution est en O(log(n)) en temps : on peut faire une dichotomie pour trouver la distance à un noeud souhaité

C'est donc facile de résoudre le problème, ce qui est plus difficile c'est de le faire efficacement :
* en espace = le label doit occuper une taille faible (et avoir une complexité en espace faible) par rapport au graphe
* en temps = au runtime, calculer la distance (à partir des labels de départ et arrivée) doit nécessiter un temps faible (et voir une complexité asymptotique faible)

distance labeling scheme = paire de fonction :
* L = prend en entrée un noeud et son graphe, et retourne son label (qui semble être un entier dans le papier ? C'est louche, j'ai dû mal comprendre...)
* f = prend en entrée les labels de deux noeuds donnés, et retourne la distance entre les deux noeuds
