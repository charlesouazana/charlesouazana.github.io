# Rapport TP UP1 
<br>

## Visualisation des séries temporelles 

On commence par tracer les trois graphes côte à côte afin d'avoir un premier aperçu des données à analyser.
![alt](https://user-images.githubusercontent.com/104632559/194909363-b8800187-4a44-4583-a66d-b6af606cd5c3.svg)
On remarque que les données ont une forte périodicité et que la série temporelle 'htdd' atteint son maximum lorsque la série 'cldd' atteint son minimum et réciproquement.
<br><br><br>


## Premières prédictions
<br>
Il nous manque les données pour les kwh pour l'année 2020. On réalisera donc les vérifications sur l'année 2019.

La première régression donne des résultats très satisfaisants avec un r2 et un r2-adjusted de 0.930 et 0.929.

Néanmoins, on observe en plottant les résultats que les prédictions se trompent dans une grande partie des cas. 
La RMSE pour ce modèle sur l'année 2019 est 15.27
La RMSE est assez importante par rapport aux valeurs que l'on cherche à prédire.

![alt](https://user-images.githubusercontent.com/104632559/195525812-fc0cfa5f-00ba-4450-8c93-9a170cdbae31.svg)
Néanmoins, cette solution est une solution benchmark car aucune transormation n'a été réalisée sur la série temporelle et car on a pas cherché à vérifier les hypothèses pour la mise en place d'un modèle de régression linéaire.

### Analyse des résidus 
![alt](https://user-images.githubusercontent.com/104632559/195888538-0d964105-4495-4109-8bf4-1bb427d49858.svg)
On remarque une forme de parabole dont le sommet est atteint en 2013.

En outre, on reamrque aussi une corrélation des résidus avec un décalage de 12, nous allons essayer de réduire cela en retirant la saisonnalité de la série.
![alt](https://user-images.githubusercontent.com/104632559/195553390-14115483-d548-433d-82fa-34bd735db287.svg)

Enfin, on analyse la normalité des résidus avec un diagramme quantile-quantile.
![alt](https://user-images.githubusercontent.com/104632559/195557705-408ba454-ac65-420c-b0f3-a944febff76c.svg)
On remarque une certaine déviation avec la droite de henry aux extrémités ce qui nous amène à conclure que le modèle peut être amélioré.

L'analyse des résidus nous montre que le modèle peut être amélioré en effectuant des modifications et que les résidus ne vérifient pas certaines hypothèses nécessaires à la mise en place d'un modèle de régresson linéaire.

## Transformation de la série
<br>

### Stabilisation de la variance
On remarque sur ce graphe que la série des kwh n'est pas stationnaire.
![alt](https://user-images.githubusercontent.com/104632559/195168478-606ed659-ec2a-4486-88d7-749a902bff04.svg)
Néanmoins, les transformations racine et logarithme permettent de stationnariser la variance. 

On voit que la transformation logarithmique permet de stabiliser la variance et on va réaliser des prédictions dessus pour voir si on parvient à améliorer le modèle. 

<br><br><br>


### Prédiction de la série logarithmique
La régression donne des résultats encore meilleurs avec un r2 et un r2-adjusted de 0.943 et 0.942.

On obtient ainsi la prédiction suivante qui semble meilleure que la prédiction réalisée sans la transformation logarithmique.
![alt](https://user-images.githubusercontent.com/104632559/195598080-b4ba6b9a-bd84-4eb2-b6d2-07ce02a762fa.png)

La RMSE pour ce modèle sur l'année 2019 est 13.18.
Le modèle est donc plus performant que le modèle basique.


### Analyse des résidus 
![alt](https://user-images.githubusercontent.com/104632559/195602178-6ff44082-86ce-46fa-b843-41d4928a3838.svg)
La forme parabolique des résidus est toujours bien présente bien qu'atténuée par le lissage effectué.

En outre, on reamrque aussi une corrélation des résidus avec un décalage de 12, nous allons essayer de réduire cela en retirant la saisonnalité de la série.
![alt](https://user-images.githubusercontent.com/104632559/195553390-14115483-d548-433d-82fa-34bd735db287.svg)

Enfin, on analyse la normalité des résidus avec un diagramme quantile-quantile.
![alt](https://user-images.githubusercontent.com/104632559/195557705-408ba454-ac65-420c-b0f3-a944febff76c.svg)
On remarque une certaine déviation avec la droite de henry aux extrémités. Néanmoins, la normalité des résidus semble plus marquée.

Il nous reste à tenter d'améliorer le modèle en utilisant la série différenciée pour les prédictions et en rajoutant un paramètre permettant de réduire la forme parabolique des résidus.


### Prédiction de la série logarithmique différenciée
En réalisant des prédictions en utilisant la série logarithmique différenciée de 13 et comme variable explicatives le temps, et les heating et cooling degree day différencié, on obtient un régression avec un r2 et r2 ajusté moins bon mais on obtient néanmoins une rmse qui vaut 16.56.
LA rmse est moins bonne que la rmse obtenue pour la série logarithmique différenciée. 
Néanmoins, l'analyse des résidus est meilleure.

On conservera donc ce modèle pour les prédictions.
