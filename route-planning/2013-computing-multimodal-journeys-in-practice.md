# (ARTICLE) Computing Multimodal Journeys in Practice

- **url** = [PDF](https://renatowerneck.files.wordpress.com/2016/06/ddpww13-multimodal.pdf) (md5sum=`e5958187cc9a8b479025125ae6accf34`), [copie locale](LOCALCOPIES/ddpww13-multimodal.pdf)
- **source** = semble avoir été publié lors de [SEA 2013](http://sea2013.dis.uniroma1.it/accepted.php), même si l'URL de download indique trompeusement 2016
- **auteurs** = Daniel Delling, Julian Dibbelt, Thomas Pajor, Dorothea Wagner, and Renato F. Werneck
- **date de publication** = 2013
- **date de rédaction initiale de ces notes** = 29 octobre 2020

Note vrac (je n'ai fait que survoler) :
- MCR = Multimodal multiCriteria Raptor = extension de McRAPTOR pour gérer le multimodal (utilise également MLC = multi-label correcting)
- derrière, post-processing pour réduire le pareto-set
- modes utilisés : _we use walking, cycling, and the public transportation network_
- critères utilisés : _arrival time, number of trips, and walking duration._
- un deuxième test ajoute le taxi comme mode, et le coût du taxi comme critère (le coût du PT reste ignoré, considéré comme bien moins cher que le taxi)
- Tests sur le réseau londonien.
- Les meilleurs résultats sont de l'ordre de la dizaine de ms, mais certains résultats sont de l'ordre de la seconde. Si on ajoute le coût des taxis, on est facilement sur plusieurs secondes.
