# (ARTICLE) Engineering Route Planning Algorithms

- **url** = [article](https://i11www.iti.kit.edu/extra/publications/dssw-erpa-09.pdf) (md5sum=`068e2a9bb39a88d9ae3fcaa6ed60709a`, [copie locale](./LOCALCOPIES/dssw-erpa-09.pdf)).
- **source** = l'article semble faire partie du livre [Algorithmics of Large and Complex Networks](https://www.springer.com/gp/book/9783642020933) (et possiblement de "Lecture Notes in Computer Science, vol 5515" ?).
- **auteurs** = Daniel DELLING, Peter SANDERS, Dominik SCHULTES, and Dorothea WAGNER / Karlsruhe Institute of Technology
- **date de publication** = 2009
- **date de rédaction initiale de ces notes** = 2020

## Notes vrac

**PRÉAMBULE** : rédiger des notes plus détaillées semble d'intéret mineur, puisqu'un survey plus récent et plus intéressant existe : [Route Planning in Transportation Networks](https://arxiv.org/pdf/1504.05140.pdf).

Excellent  article néanmoins.

Donne des pistes avec références pour plusieurs de nos problèmes actuels :
- [Mobile route planning](http://algo2.iti.kit.edu/schultes/hwy/mobileSubmit.pdf) (2008 — Peter Sanders, Dominik Schultes, and Christian Vetter) :
    + Notamment : encodage efficace des graphes, 140 Mio pour représenterl'Europe
    + représentation efficace du graphe (en réarrangeant pour mettre ensemble ce qui va ensemble) + compression
    + n'utilise pas CH mais plutôt highway node routing
    + comme c'est tourné pour rendre efficace les acccès-disques, l'intérêt est à contextualiser
- Turn-costs :
    + idée de base = inversion = représenter les edges et noeuds du réseau routier dans un graphe où les edges sont des noeuds et les noeuds sont des edges
    + on peut ainsi attribuer un poids au "edges" dans ce graphe en fonction des deux "noeuds" qu'ils relient, i.e. en fonction des deux edges réels reliés
    + problème = ça augmente la taille du problème (en effet, dans ce graphe inversé, il y a autant de "noeuds" que de paires d'edges réels)
	+ papier 1 = [Route Planning in Road Networks with Turn Costs](http://algo2.iti.kit.edu/documents/routeplanning/volker_sa.pdf) (2008 — Lars VOLKER)
	+ papier 2 = [Efficient Routing in Road Networks with Turn Costs](https://algo2.iti.kit.edu/download/turn_ch.pdf) (2011 — Robert GEISBERGER and Christian VETTER)

D'une façon générale, il y a beaucoup de références utiles sur les sujets liés au routing.

Leurs mesures laissent à penser qu'il y a des algos plus rapide que CH (ch + edge flags, ou encore plus rapide : transit node routing). Les techniques basées sur le hub-labeling ne sont pas mentionnées, peut-être car elles sont sorties APRÈS cet article ?
