# AnomalyDetection-Aircraft-systems

# Context
Un expert en systèmes aéronautiques a demandé l'analyse d'un ensemble de données, qui contient 11 variables d'information sur plusieurs vols, pour toutes les seconde des vols. Les données sont étiquetées par jour, par vol et par fênetre, (les fenêtres sont des séquences de mesures, de longueur allan jusqu'à 100 secondes).

Il souhaite disposer d'un algorithme permettant de détecter les fênetres "anormales".

Nous avons ici un jeu de données contaminé : des données dans le jeu de données sont des anomalies et nous n'avons pas apriori de labels sur les données. Il s'agit donc d'un problème d'outlier detection où nous cherchons à détecter quelles données sont rares et différentes des autres.

# Analyse Préliminaire des Données
La première étape, afin de mieux comprendre le jeu de données fourni par l'expert, est d'ajouter des paramètres de vol et de temps.

Un vol typique comportera différentes phases, telles que le décollage, la croisière et l'atterrissage, où les 11 variables auront des valeurs très différentes. Par conséquent, une hypothèse raisonnable consiste à considérer que les points de données sont comparables entre les vols si elles appartiennent à la même phase du vol. En effet, on considère que pour deux vols distincts, les phases d'atterrissages du vol 1 et du vol 2 vont avoir des mesures similaires.

Une fois ajoutée au jeu des données, l'évolution dans le temps de chaque variable p a été tracée pour chaque vol. Les résultats de l'analyse précédente ont montré une variabilité importante entre les données de chaque vol. De plus, ils ont également montré qu'il est possible d'identifier des phases de vol distinctes dans les variables, ce qui a conduit aux premièrse décisions de l'analyse :

Dans les approches proposées, nous ne cherchons pas à comparer des données entre vols, mais à détecter des comportements anormaux entre plusieurs fenêtres de vol consécutifs.

Le paramètre phase du vol sera utilisé pendant les analyses, car on a considéré que les fenêtres se regrouperaient si elles étaient temporellement proches les unes des autres, et qu'à l'inverse, les mesures anormales seraient isolées de toutes les autres fenêtres.

Dans les approches proposées, nous utiliserons l'algorithme de détection d'anomalie "Isolation Forest". Parmi les quatre algorithmes considérés (One Class SVM, Elliptic Envelope, Local Outlier Factor, Isolation Forest), les caractéristiques suivantes du jeu de données ont été fondamentales pour la décision :

Étant donné que les différentes phases de vol devraient donner lieu à des mesures différentes, nous ne pouvons pas définir une zone unique et nous attendre à ce que toutes les bonnes mesures y tombent, donc créer un contour elliptique ne fonctionnera pas.
Les quatre algorithmes considérés sont basés sur la distance relative des points, ce qui signifie qu'ils souffrent tous de la malédiction de la dimension. Lorsque l'on traite de grandes dimensions (un grand nombre de variables), les scores obtenus par les algorithmes perdent de leur signification. Toutefois, entre les quatre choix, étant donné que la "Random Forest" implique des coupes aléatoires, si le nombre d'arbres est suffisant, elle sera plus robuste que les autres options lorsqu'il s'agit de grandes dimensions.

# Proposition
Afin de détecter les fenêtres anormales, nous proposons deux approches, où nous analysons les vols séparément pour détecter les fenêtres qui paraissent incohérentes par rapport aux fenêtre voisines du même vol, en utilisant l'algorithme "Random Forest".

- Première approche
Notre proposons la procédure de détection d'anomalies suivante:

En considérant tous les points de chaque vol et en définissant un taux de contamination par vol (15%), on peut obtenir un étiquetage initial pour séparer les points "anormaux" avec un algorithme de détection d'anomalies : le "Random Forest".

Nous pouvons compter le nombre de points anormaux dans chaque fenêtre et déterminer un seuil (en percentage de points anormaux) à partir duquel on considère qu'une fenêtre est anormale (50%).

- Deuxième approche
Notre proposons la procédure de détection d'anomalies suivante:

Feature Engineering: On synthétise et on représente les 100 secondes de chaque variable de chaque fenêtre par des statistiques sur le fenêtre en question. On utilise pour chaque variable leur moyenne, écart-type, médiane, minimum, maximum, le 1er et 3e quartile et l'écart interquartile. Cela nous donne 8 variables pour chaque variable initiale + la variable de temps qu'on représente par la moyenne de chaque fenêtre. On a alors 8*11+1 = 89 variables. On a donc 89 variables pour 1637 points (1637 fenêtres). Le problème étant à grande dimension. En grandes dimensions, les distances entre les points deviennent relativement proches et il est alors difficile de détecter les anomalies car il y aura moins de points anormaux plus distants des autres.

Pour pouvoir détecter des données anormales, il faut diminuer le bruit des données et capturer les composants perninents de notre jeu de données. Nous avons donc ensuite besoin de réduire la dimension du problème. Pour ce faire, nous gardons 6 dimensions principales qui sont les projections linéaires des autres dimensions permettant de conserver le maximum d'information (de variance). Nous avons choisi 6 dimension car sur le vol 3, c'est le nombre de fenêtres.

Ensuite, nous pouvons faire une détection d'anomalie sur ces données avec l'"Isolation Forest" avec un taux de contamination défini (15%).

# Comparaison des propositions
Pour aider l'expert à choisir parmi les deux approches, nous avons comparé leurs avantages et inconvénients et les résultats qu'ils fournissent.

# Avantages et inconvénients
La première approche est intéressante car elle permet de travailler sur l'information initiale : les 11 variables telles qu'elles étaient. La principale difficulté est de proprement définit une anomalie : à partir de combien de points anormaux on considère qu'une fenêtre est anormale. Cette décision est arbitraire et dépend de la façon dont l'expert perçoit l'anomalie. Cette méthode est donc intéressante pour conserver toute l'information initiale, mais elle est à éviter si une telle définition d'une anomalie est difficile dans le cas présent.

En plus, parce que la feauture d'une fenêtre n'était pas définie dans la première méthode, un problème possible identifié est l'agrégation des points "anormaux" d'une fênetre donnée. Si les 100 points d'une fênetre avaient tous des valeurs similaires "anormales", l'analyse précédente pourrait ne pas les détecter en tant que valeurs "anormales".

Le dernier problème de cette méthode est également lié à la non-définition d'une feature. Comme nous travaillons avec des données brutes, il est possible que l'algorithme de détection des anomalies identifie une grande distance d'un point dans une direction non intéressante, c'est-à-dire, dans une direction qui ne correspond pas à la direction d'étalement maximal de l'information. Dans ce cas, l'algorithme pourrait considérer à tort ce point comme anormal, ce qui ne serait pas intéressant pour l'expert.

La deuxième approche permet directement de travailler sur les fenêtres de vol. L'avantage est qu'il n'est plus nécessaire de définir ce seuil de point. Il suffit de choisir un pourcentage de fenêtres que nous souhaitons rejeter en tant qu'anormales. En plus, en raison de la définition étendue des features de la fenêtre, la méthode est plus robuste et produit une feature plus représentative des données.

# Comparaison des résultats
Afin d'analyser les résultats des approches proposées, ainsi que pour recevoir le feedback de l'expert et, récursivement, affiner les paramètres d'entrée seuil et taux de contamination, une méthode de réduction de dimension (ACP) a été utilisée afin de réduire les résultats pour chaque vol à 2 dimensions principales et les présenter en 2D.

Pour une analyse cohérente de ces données, toutes les méthodes utilisées doivent être validées par l'expert. Ainsi, les résultats peuvent ne pas être adaptés à la situation réelle.

Néanmoins, un facteur intéressant à noter est que le taux d'anomalie obtenu pour la première approche (10%) était inférieur à celui de la deuxième approche (21%) (pour les paramètres définis ci-dessus). Par contre, lorsque les deux résultats sont visualisés pour chaque vol, la séparation entre les anomalies et les fenêtres normales est beaucoup plus intuitive dans la deuxième méthode.

# Recommandations finales
Après avoir les résultats donnés par chaque approche, on recommande finalement l'utilisation de l'approche d'identification des outliers en utilisant l'aggregation des fenêtres par features. Le premier avantage c'est de ne pas avoir la necessité de prendre des hypothèses par rapport aux points dans une fenêtre, et cela est d'autant plus intéressant du fait que l'intêret est de détecter certaines fenêtres entières comme des outliers. Par ailleurs, en faisant l'analyse point par point, on est soumis à la possibilité de ne pas identifier une fenêtre comme un outlier même si elle l'est dans le cas où tous les points de cette fenêtre ont un biais similaire. Dans ce cas là, même si les valeurs de la fênetre sont anormale par rapport à d'autres fenêtres, ses point ne seront pas isolés, mais bien rapprochés les uns des autres. Comme l'analyse par features réduit chaque fenêtre à un point, nous pensons que le risque de ce genre de problème est réduit.
