# Idée Générale
Le but de ce TP est d'implémenter les fonctions _de recherche, d' intersection, de suppression et  d'insertion_ sur une table de hachage avec un hachage et une fonction de stockage définis préalablement.
## Fichier openfile
#### Le fichier open_file ne contient qu'une seule fonction, la fonction `open_file`.
Cette fonction a pour argument le nom du fichier de type `:str` et renvoie une liste contenant tous les mots du fichier sans le `\n`.  

---
## Fichier Hachage.py
Le fichier hachage.py contient les fonctions de hachage et de stockage codées précédemment. 
Trois fonctions de ce fichier sont utilisés pour ce TP.

### Fonction `hachage_2` 
À partir d'un mot et de la taille de la TAD génère la valeur de hachage du mot.  
La fonction `hachage_2` réalise un hachage de jenkins pondéré d'une chaîne de caractère : 
#### Exemple
```python
X = 'a'
M = 786307
print(hachage_2(X,M))
```
```
86309
```
### Fonction `chaining`
La fonction `chaining` prend en argument : 
- A : Une table de hachage avec un stockage en adressage fermé de type `:list`
- x : Un mot à y introduire de type `:str`
- h : La valeur de hachage du mot de type `:ìnt`

Elle modifie le tableau A entré en paramètre.
#### Exemple
```python
A = [[]]
x ='a'
h = 0
A = chaining(A,h,x)
print(A)
```
````
[['a']
````
### Fonction `hash_table`
La fonction `hash_table` prend en argument :
- hachage : Une fonction de hachage (par exemple `hachage_2`)
- stockage : Une fonction de stockage (par exemple stockage)
- *arg : 
    - Une liste contenant la liste des mots que l'on veut hacher cette liste est obtenue par la fonction open_file du fichier openfile.py
    - Un entier correspondant à la taille de la table de hachage

Elle renvoie une table de hachage de taille M contenant les éléments de la liste contenu dans *args.
#### Exemple
```python
hachage = hachage_2
stockage = chaining
f1 = open_file('/Users/charlesouazana/Downloads/word2.txt')
M = nextprime(len(f1)/0.3)
hash_table = hash_table(hachage, stockage, [f1,M])
print(hash_table[:10])
```
````
[['affusion', 'averruncation'], ['hydrocholecystis'], [], [], ['breathlessly'], [], ['svantovit'], [], ['cordonnet', 'misderivation'], ['cubitale']]
````
___
## Fichier intersection.py
Le fichier intersection.py contient les fonctions __recherche__, __insertion__, __suppresion__, __intersection__.

### Fonction `recherche`
La fonction `recherche` prend comme argument : 
- f1 : Une table de hachage de type ```list```
- word : Un mot de type ```str```
- M : Un entier de type ```int```

Elle renvoie un booléen selon que le mot est dans la table de hachage ou non. 
#### Exemple
```python
f1 = open_file('/Users/charlesouazana/Downloads/word2.txt')
M = nextprime(len(f1)/0.3)
print(recherche(f1, 'aaron'))
```
```
True
```
#### Complexité 
- La complexité en temps est _O(1+N/M)_
- La complexité en espace est _O(1)_
### Fonction `intersection`
La fonction `intersection` prend en argument deux chemins de fichier et renvoie une liste des mots communs et la longueur de cette liste sous forme de tuple.
#### Exemple
```python
path1 = '/Users/charlesouazana/Downloads/word2.txt'
path2 = '/Users/charlesouazana/Downloads/texte_Shakespeare.txt'
L , n = intersection(path1,path2)
print(L[:10],n)
```
````
['a', 'aaron','abandon', 'abandoned', 'abase', 'abash', 'abate', 'abatement','abbess', 'abbey']
---------------------------------------
13821
````
#### Complexité
- La complexité en temps est _O(N*(1+N/M))_
- La complexité en espace est en _O(N)_ car on stocke le fichier file_1 au cours de l'éxécution de la fonction.

### Fonction `insertion` 
La fonction `insertion` prend en argument : 
- file : Le chemin de fichier dans lequel on veut insérer le mot de type `:str`
- f1 : La table de hachage associé à ce fichier de type `:list`
- word : Le mot que l'on souhaite insérer de type `str` 
- M : La taille de la table de hachage de type `:int`

Elle modifie la table de hachage en insérant et insère le mot (plus `\n`) à la fin du fichier mais elle ne retourne rien.

### Fonction `suppression`
La fonction `suppression` prend en argument : 
- file : Le chemin de fichier dans lequel on veut insérer le mot de type `:str`
- f1 : La table de hachage associé à ce fichier de type `:list`
- word : Le mot que l'on souhaite insérer de type `str` 
- M : La taille de la table de hachage de type `:int`

Elle modifie la table de hachage en retirant le mot dans l'alvéole correspondante.  
En outre, elle réécrit le fichier en réécrivant chacune des lignes, à l'exception de la ligne égale au mot que l'on souhaite retirer.
___

