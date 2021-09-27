# (POWERPOINT) Multi-criteria shortest paths

- **url** = [PDF](https://www.labri.fr/perso/anlentz/Presentation/GTAlgoDist.pdf) (md5sum=`be2358ff72e35a275019b2de7c2d9749`), [copie locale](LOCALCOPIES/GTAlgoDist.pdf)
- **source** = la [page perso d'Antonin LENTZ au LABRI](https://www.labri.fr/perso/anlentz/en.php)
- **auteurs** = Antonin LENTZ, post-doc au [LABRI](https://www.labri.fr/)
- **date de publication** = 7 janvier 2019
- **date de rédaction initiale de ces notes** = 27 octobre 2020

Présentation donnée en janvier 2019 à un working-group "Distributed Algorithms"

Calcul d'itinéraire uni-critère très bien adressé (temps de réponse en µs pour 10⁶ nodes)

Calcul d'itinéraire multi-critère très mal adressé (15 min pour 10⁴ nodes en tri-critère)

Nécessité de simplifier le front de Pareto :
- K-paths = on se limite à K chemins
- heuristics = parmi ceux sur le front de Pareto, on essaye d'évaluer de meilleurs itis que d'autres
- approximation = domination à ε près

La présentation semble se concentrer sur quelques algos d'approximation de fronts de Pareto
