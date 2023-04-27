&nbsp;
![Python](https://img.shields.io/badge/python-v3.6+-blue.svg)  

# Idée Générale
Le but de ce TP est d'implémenter une fonction de recherche d'intersection s'appuyant sur la rechercher probabilistique des filtres de bloom. 
Le moyen de stockage des filtres de Bloom est de stocker un fichier sous la forme d'un format de type `bitarray` permettant ainsi une grande économie de place tout en assurant une probabilité de faux positif faible.
__

# Fichier openfile
#### Le fichier open_file ne contient qu'une seule fonction, la fonction `open_file`.
Cette fonction a pour argument le nom du fichier de type `:str` et renvoie une liste contenant tous les mots du fichier sans le `\n`.  
___
# Fichier Bloom 
Le fichier Bloom contient la fonction qui permet de déterminer l'interection entre deux fichiers en utilisant les fonctions de Hachage écrites dans le fichier hash_func.py et dans le fichier _hachage.py_. 
## `dict_func`
Le dictionnaire `dict_func` liste toutes les fonctions de hachage disponible pour la création du filtre de bloom : 
```python
dict_func ={
     0 : hf.le1_bon_hashage ,
     1 : hf.le2_bon_hachage ,
     2 : hf.co1_hachage_original , 
     3 : hf.co1_bon_hachage ,
     4 : hf.al_hachage ,
     5 : hf.el1_bon_hachage ,
     6 : hf.el_autre_hachage ,
     7 : h.hachage_1 ,
     8 : h.hachage_2 ,
     9 : h.hachage_3 ,
    10 : h.hachage_4
}
```
Les fonctions de hachage proviennent du fichier _hash_func.py_ et du fichier _hachage_.
Chacune de ces fonctions prend en argument un mot (de type `str`) et la taille de la table de hachage (de type `int`)

## La fonction `intersection_2` 
Cette fonction prend en argument : 
- Les deux chemins des fichiers (de type `str`) correspondant aux fichiers à comparer.
- Le nombre de fonctions de hachage que l'on souhaite utiliser pour le filtre de bloom (de type `int`)  
Dans cette fonction, on sélectionne tout d'abord le fichier contenant le plus d'élément parmi les deux fichiers donnés en entrée afin de diminuer la probabilité de faux positifs. 

Puis, on stocke ce fichier sous la forme d'un bitarray contenant seulement des zéros sauf aux index correspondant aux valeurs de hachage des mots du fichier stockés. 

Enfin, on crée une liste nommée `intersect` contenant les mots du fichier 2 dont la somme des valeurs prises par les éléments du bitarray aux index des valeurs de hachage du mot est égale à k. Plus simplement, on vérifie que toutes les valeurs de hachage du mot ont été prises par un mot de l'autre fichier. 

Si c'est le cas, on considère que c'est un mot commun aux deux fichiers et on l'ajoute à la liste `intersect`. 

La fonction bloom renvoie ensuite un tuple dont le premier élément est l'intersection des deux fichiers et le deuxième est la taille de cette intersection.

# Fichier results_analysis
Ce fichier a pour but d'analyser les performances du filtre de bloom. On analyse à travers ce fichier les performances en :
- qualité
- rapidité
- mémoire 
du filtre.

## Perfomance qualité
La qualité du filtre correspond au nombre de faux positifs que l'on a obtenu.
![alt](https://user-images.githubusercontent.com/104632559/235009133-835bb8b1-6576-4454-be26-33e6dbd64e67.png)

Sur cette image on observe que le minimum de faux-positifs est atteints pour 4 fonctions de hachage. 

On compare ensuite la probabilité théorique et la probabilité mesurée pour chacune des fonctions de hachage. 
![title](https://user-images.githubusercontent.com/104632559/235009130-08f67659-e539-473f-a1c1-ed0300e76b15.png) 

## Performance temps 
Dans cette partie, on analyse la performance en temps de la recherche des mots.
Pour cela on réalise une recherche de tous les mots du fichier 1 avec la fonction `recherche` et on compare avec le temps pris par la fonction de la table de hachage.
![alt](https://user-images.githubusercontent.com/104632559/235009123-6f71095a-c4ae-48a7-880f-0df95536e4ff.png) 

## Performance mémoire
Enfin, on compare les performances en mémoire des teux tableaux : 
```python
'Mémoire utilisée pour le filtre de bloom': 98353
'Mémoire utilisée pour la table de hachage': 6871984
```

# Conclusion
Le filtre de bloom est moins rapide que la table de hachage pour vérifier si un mot est présent ou non dans un fichier. Cela vient du fait qu'il est couteux de calculer plusieurs fonctions de hachage pour un même mot. 

Néanmoins, l'intérêt du filtre de bloom qui est bien mis en lumière ici est la taille de stockage d'un fichier de sauvegarde de mot. On voit bien ici que le filtre a une taille 70 fois moins grande que la taille de la table de hachage.

___
## Contributors 
Charles Ouazana  
Maxime Callet

