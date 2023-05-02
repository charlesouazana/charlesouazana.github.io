# Analyse du graphe des catégoreis wikipedia

## Pré-requis
Ce programme nécessite l'installation si ce n'a pas encore déja été fait des packages suivants : 
```python
import pandas as pd
from typing import List
import subprocess
import networkx as nx
import igviz as ig
import timeit
```

## Répertoire
Ce répertoire est composé de deux fichiers : 
- `sort_rewrite.sh`
- `analyse_graph.py`

<br>

### sort_rewrite.sh
Ce fichier contient un script shell qui : 
- Sélectionne puis trie les deux premières colonnes du fichier texte et les réécrits dans un fichier `identifiers.txt`
- Sélectionne les paires ID-Name correspondant aux parents du graphe et les réécrit dans un fichier `parents.txt`
- Sélectionne toutes les paires uniques de noeuds et les réécrit dans un fichier `vertices.txt`

<br>

### analyse_graph
Ce fichier contient le code principal permettant de traiter le graphe. 
Il implémente une classe nommées CategoryGraph qui est initialisée avec trois DataFrames : 
- df_identifiers : Ce dataframe stocke les paires parents-enfants sous forme d'int
- df_parents : Ce dataframe stocke les paires ID_name pour tous les parents. Il est indexé par l'ID pour récupérer plus efficacement le nom de la page
- df_identifiers : Ce dataframe stocke les paires ID-name pour les noeuds du graphe. Il est lui aussi indexé par ID.

Cette classe possède un certain nombre de méthodes : 
- `get_loops` : Cette méthode renvoie le dataframe des boucles. Output (5 premiers éléments): 

|   | Parents | Name                         |
|---|---------|------------------------------|
| 0 | 74386   | Utilisateur_Anti-Gates       |
| 1 | 74829   | Utilisateur_RHIEN            |
| 2 | 74885   | Utilisateur_Sims             |
| 3 | 75175   | Utilisateur_argot_parisien-3 |
| 4 | 75474   | Utilisateur_fro-2            |

- `get_symetrical_arc` : Cette méthode renvoie le dataframe des arcs symétriques. Output (5 premiers éléments) : 

|   | Children | Parents | Name_Children        | Name_Parents              |
|---|----------|---------|----------------------|---------------------------|
| 0 | 230824   | 4566    | Grand_Londres        | Administration_de_Londres |
| 1 | 29991    | 5952    | Escalade             | Alpinisme                 |
| 2 | 53997    | 6151    | Nouvelle-France      | Amérique_française        |
| 3 | 31981    | 6169    | Finance_d'entreprise | Analyse_financière        |
| 4 | 121479   | 7072    | Pays_catalans        | Andorre                   |

- `has_cycles` : Cette méthode renvoie un booléen suivant si le graphe contient un cycle.
````python
True
````

- `tarjan` : Cette méthode applique l'algorithme de Tarjan pour trouver les composantes fortement connexes à ce graphe. Il renvoie une liste des composantes fortement connexes du graphe.

- `visualize_scc` : Cette méthode permet de visualiser et d'intéragir avec les composantes conexes du graphe. Cette méthode prend en argument deux seuis correspondant aux longeurs min et max des composantes que vous souhaitez visualiser. La méthode ouvre ensuite un graphe plotly dans votre navigateur qui permet d'intégragier avec le graphe.

Avec la commande _visualize_scc_ on obtient plusieurs graphes similaires à celui-ci : 
![alt](https://user-images.githubusercontent.com/104632559/235008669-dd20e417-1945-44a5-a164-d16d7df62d00.png)


## Performances de l'algorithme 
L'algorithme a obtenu un meilleur score que l'algorithme de tarjan implémenté ici https://pypi.org/project/tarjan/#files.
En effet :
````python
%timeit tarjan_perso(dico_adjacency_list)
>>> 214 ms ± 36 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
%timeit tarjan(dico_adjacency_list)
>>> 382 ms ± 34.3 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
```` 

La performance totale de l'algorithme qui comprend la lecture des fichiers et leurs modifications a un temps d'éxécution de : 
````python
def exec_time():
    a = CategoryGraph(file_path_fr)
    a.tarjan()
    
%timeit create_class_and_component()

>>> 4.34 s ± 23.2 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
````

Je pense néanmoins que cette algorithme pourrait potentillement être utilisé en n'utilisant pas de shell et en utilisant uniquement python pour la lecture et le traitement des fichiers.
