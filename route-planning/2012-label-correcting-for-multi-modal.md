# (ARTICLE) A label correcting algorithm for the shortest path problem on a multi-modal route network

- **url** = [PDF](http://www.lix.polytechnique.fr/Labo/Leo.Liberti/sea12c.pdf) (md5sum=`882248d935539f496358982b9205b591`), [copie locale](LOCALCOPIES/sea12c.pdf)
- **source** = LIX = [Laboratoire d'Informatique de l'École Polytechnique](https://www.lix.polytechnique.fr/), publié lors de [SEA 2012](https://sea2012.labri.fr/) = Symposium on Experimental Algorithms, dont l'édition 2012 a été organisée par le LABRI
- **auteurs** = Dominik KIRCHLER, notamment auteur d'une thèse en soutenue en 2013 sur le calcul d'itinéraire multimodal, Léo LIBERTI encadrant de ladite thèse, Roberto WOLFLER CALVO 
- **date de publication** = 2012
- **date de rédaction initiale de ces notes** = 29 octobre 2020

Note vrac (je n'ai fait que survoler) :
- papier français de 2012, cité par le [survey de 2015](2015-route-planning-in-transportation-networks.md)
- les modes utilisés : _private bike, rental bike, walking, car and public transportation._ 
- étude porte sur graphe IDF = ~4M edges, ~1M nodes
- 15% des edges car sont time-dependent pour prendre en compte le traffic
- en fonction des scenarios, le query time est de l'ordre de la dizaine à la centaine de ms
