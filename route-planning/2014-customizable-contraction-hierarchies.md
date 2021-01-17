# (ARTICLE) Customizable Contraction Hierarchies

- **url** = [article](https://arxiv.org/pdf/1402.0402v5.pdf) (md5sum=`aef7c5b64f8a7775dc755596fdf5ecee`, [copie locale](./LOCALCOPIES/1402.0402v5.pdf)). [la thèse](https://i11www.iti.kit.edu/_media/teaching/theses/weak_ch_work-1.pdf) (md5sum=`36e812ef9773bcc96ab0743e6fadee39`, [copie locale](./LOCALCOPIES/weak_ch_work-1.pdf))
- **source** = l'article a été présenté à [SEA 2014](https://di.ku.dk/sea2014/accepted-papers/).
- **auteurs** = Julian DIBBELT, Ben STRASSER and Dorothea WAGNER / Karlsruhe Institute of Technology
- **date de publication** = 2014
- **date de rédaction initiale de ces notes** = août 2020 (possiblement, notes plus anciennes dans le tas)

## Notes synthétiques

Notes à prendre, notamment sur l'article (les notes vrac ci-dessous ont été prises sur la thèse sur WCH et non l'article ; de plus, certaines notes sont globales à CH/dijkstra plutôt que propre à CCH/WCH).

## Notes vrac

Note sur article vs. thèse vs. ... :
- le code RE fait explicitement référence à la thèse (d'où le fait que la technique soit appelée WCH au lieu de CCH).
- tous les travaux scientifiques postérieurs semblent citer l'article ([exemple1 en 2017](https://arxiv.org/pdf/1504.03812.pdf), [exemple2 en 2019](https://arxiv.org/pdf/1906.11811.pdf), ...)
- je ne l'ai pas lu pour confirmer, mais je crois avoir vu passer que [ceci](https://www.microsoft.com/en-us/research/wp-content/uploads/2011/05/crp-sea.pdf) est un concurrent de CCH (permettant également la customization), qui résume des sections entières du graphe en "sautant" des pans entiers.

Wch permettrait de supporter les turn costs !

La fin de la page 15 (chap 3.1) précise les conditions d'arrêt du dijkstra qui n'étaient pas claires pour moi à la lecture de CH :
- soit la distance minimale de la priority queue du dijkstra est supérieure à la taille du raccourci
- (En effet, tout chemin trouvé par le dijkstra à partir de maintenant sera forcément PLUS GRAND que le raccourci)
- soit tous les successeurs du noeud contracté v sont «settled»
- (en effet, dans ce cas, le dijkstra a trouvé leur distance minimale depuis u, qui ne changera pas : on a déjà trouvé les plus courts chemins qui nous intéressent, inutile de pousser plus loin)

En effet, dans dijkstra, une fois qu'on dépile un noeud de la priority queue, la distance de ce noeud ne sera plus modifiée par la suite :
- le noeud est dit "settled"
- tous les noeuds restants dans la priority queue ont une distance supérieure au noeud en cours ; donc même si on peut atteindre le noeud en cours par un noeud restant, ce sera forcément par un chemin plus long
- inversement, comme le noeud est le premier de la priority queue, ça veut dire que tous les noeuds qui étaient plus proches de nous ont déjà été dépilés ; du coup tous ceux qui auraient pu permettre de l'atteindre d'une autre façon plus rapide ont DÉJÀ été traités
- dit autrement, quand on settle un noeud, on a trouvé le plus court chemin de la source au noeud settled

Minimal contraction hiérarchie = celle où on a poussé le dijkstra jusqu'au bout (on n'aura ajouté de shortcut QUE s'il était nécessaire). En pratique, on arrête le dijkstra local plus tôt, et il se peut qu'on ajoute un shortcut inutile, la contraction n'est pas minimale mais c'est pas bien grave.

Rigolo : trouver un ordering qui minimise le nombre de shortcuts ajoutés a été prouvé comme étant NP-difficile

Visualiser concrètement que CH accélère les queries :
- D'abord, calculer le degré moyen d'un noeud sur le graphe original
- Puis, calculer le degré moyen d'un noeud sur le graphe contracté upward (resp. backward), je m'attends à ce qu'on voie un graphe BEAUCOUP plus petit.
- Et comme dijkstra est en O(E + V.log V), on y gagne
- (Alternative = compter E et V avec et sans CH) 

Witnessless graph contraction :
- On ajoute tous les shortcuts possibles
- (au lieu de n'ajouter que ceux indispensables pour maintenir les plus courts chemins du graphe)
- En fin de contraction, l'overlay  graph est une clique.
- Même witnessless, le résultat de la contraction est toujours TRÈS DÉPENDANT de l'ordering.

Définition de Formal contraction hiérarchie : tous les arcs (réels ou shortcuts) forward (resp. backward) tel que le PCC entre les deux noeuds de l'arc soit l'arc lui même.

Définition de Weak contraction hiérarchie :
- une contraction hiérarchie telle que tout shortcut (i.e. arc n'existant pas dans le graphe original) est issu de la contraction d'un node d'ordre plus petit que les extrémités du shortcut.
- Ndm : c'est obligatoire vu la définition de la contraction hiérarchie!
- Si je comprends bien, «weak» car ne fait pas intervenir la notion de coût/poids ou la notion de plus court chemin pour définir les raccourcis.
- Dit autrement, on se contente d'avoir la contrainte sur "un shortcut est ajouté en contractant les nodes dans l'ordre donné par l'ordering" sans avoir la contrainte "on n'ajoute un shortcut QUE s'il est indispensable pour préserver les plus courts chemins sur le graphe"

Maximal contraction hiérarchie :
- une CH où on ajouté TOUS les shortcuts de chaque noeud contracté.
- Présente des similitudes avec le «elimination game», qui produit un "elimination tree"

Élimination tree :
- pas encore hyper clair, mais la description textuelle sous la définition 3.6 aide :
    + The elimination tree is constructed from the result of the elimination game.
    + For each node it contains only the arc to the lowest higher node.
    + The node with the highest rank is the tree’s root
- Le point important, c'est que tout chemin dans une weak contraction hiérarchie appartient à l'élimination tree du couple graphe+ordering.
- Or, l'elimination game et l'elimination tree qui va avec sont des sujets déjà massivement étudiés dans la littérature.
- Du coup en choisissant un ordering qui minimise la hauteur de l'élimination tree (ce qu'on sait faire avec des heuristiques!), on minimise le chemin dans la contraction hiérarchie.

Nested dissection :
- coupe binaire récursive du graphe.
- Pour un ordering calculé utilisant les nested dissection, la contrainte suivante est toujours vraie = les noeuds de coupe ont un ordre supérieur à tout noeud des deux moitiés coupées.

TODO = tracer la courbe du degré d'un node en fonction de son index d'ordering (pour vérifier visuellement que les derniers noeuds ont une quantité d'edges faramineuse).
