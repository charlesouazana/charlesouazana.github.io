This school work aims is to use pyspark to apply clustering methods to big datasets.

### Packages list


```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import isnan, when, count, col
from pyspark.ml.clustering import KMeans as KMeansml
from pyspark.ml.evaluation import ClusteringEvaluator
from pyspark.ml.feature import PCA, VectorAssembler, StandardScaler
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.colors import rgb2hex
from mpl_toolkits.mplot3d import Axes3D
import seaborn as sns
import numpy as np
from sklearn.cluster import KMeans
```

### Data load and analysis


```python
spark = SparkSession.builder.appName('pySparkSetup').getOrCreate()
```

    23/02/05 01:00:13 WARN Utils: Your hostname, Charless-MacBook-Pro.local resolves to a loopback address: 127.0.0.1; using 172.20.10.2 instead (on interface en14)
    23/02/05 01:00:13 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
    WARNING: An illegal reflective access operation has occurred
    WARNING: Illegal reflective access by org.apache.spark.unsafe.Platform (file:/Users/charlesouazana/opt/anaconda3/envs/charles/lib/python3.10/site-packages/pyspark/jars/spark-unsafe_2.12-3.2.1.jar) to constructor java.nio.DirectByteBuffer(long,int)
    WARNING: Please consider reporting this to the maintainers of org.apache.spark.unsafe.Platform
    WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
    WARNING: All illegal access operations will be denied in a future release
    Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
    Setting default log level to "WARN".
    To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
    23/02/05 01:00:13 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable



```python
df_white = spark.read.csv('winequality-white.csv',header='true', inferSchema='true', sep=';')
df_red = spark.read.csv('winequality-red.csv',header='true', inferSchema='true', sep=';')
```

We will now show the schema of the dataframe and get a grasp at what it contains


```python
df_white.printSchema()
print("Rows: %s" % df_white.count())
```

    root
     |-- fixed acidity: double (nullable = true)
     |-- volatile acidity: double (nullable = true)
     |-- citric acid: double (nullable = true)
     |-- residual sugar: double (nullable = true)
     |-- chlorides: double (nullable = true)
     |-- free sulfur dioxide: double (nullable = true)
     |-- total sulfur dioxide: double (nullable = true)
     |-- density: double (nullable = true)
     |-- pH: double (nullable = true)
     |-- sulphates: double (nullable = true)
     |-- alcohol: double (nullable = true)
     |-- quality: integer (nullable = true)
    
    Rows: 4898


Since the head method of pyspark doesn't broadcast results in the form of a table we will just convert to pandas a small fraction of the df.
This yields the following results.


```python
df_red.printSchema()
print("Rows: %s" % df_red.count())
```

    root
     |-- fixed acidity: double (nullable = true)
     |-- volatile acidity: double (nullable = true)
     |-- citric acid: double (nullable = true)
     |-- residual sugar: double (nullable = true)
     |-- chlorides: double (nullable = true)
     |-- free sulfur dioxide: double (nullable = true)
     |-- total sulfur dioxide: double (nullable = true)
     |-- density: double (nullable = true)
     |-- pH: double (nullable = true)
     |-- sulphates: double (nullable = true)
     |-- alcohol: double (nullable = true)
     |-- quality: integer (nullable = true)
    
    Rows: 1599



```python
df_red.limit(5).toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fixed acidity</th>
      <th>volatile acidity</th>
      <th>citric acid</th>
      <th>residual sugar</th>
      <th>chlorides</th>
      <th>free sulfur dioxide</th>
      <th>total sulfur dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7.4</td>
      <td>0.70</td>
      <td>0.00</td>
      <td>1.9</td>
      <td>0.076</td>
      <td>11.0</td>
      <td>34.0</td>
      <td>0.9978</td>
      <td>3.51</td>
      <td>0.56</td>
      <td>9.4</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7.8</td>
      <td>0.88</td>
      <td>0.00</td>
      <td>2.6</td>
      <td>0.098</td>
      <td>25.0</td>
      <td>67.0</td>
      <td>0.9968</td>
      <td>3.20</td>
      <td>0.68</td>
      <td>9.8</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.8</td>
      <td>0.76</td>
      <td>0.04</td>
      <td>2.3</td>
      <td>0.092</td>
      <td>15.0</td>
      <td>54.0</td>
      <td>0.9970</td>
      <td>3.26</td>
      <td>0.65</td>
      <td>9.8</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11.2</td>
      <td>0.28</td>
      <td>0.56</td>
      <td>1.9</td>
      <td>0.075</td>
      <td>17.0</td>
      <td>60.0</td>
      <td>0.9980</td>
      <td>3.16</td>
      <td>0.58</td>
      <td>9.8</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.4</td>
      <td>0.70</td>
      <td>0.00</td>
      <td>1.9</td>
      <td>0.076</td>
      <td>11.0</td>
      <td>34.0</td>
      <td>0.9978</td>
      <td>3.51</td>
      <td>0.56</td>
      <td>9.4</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_white.limit(5).toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fixed acidity</th>
      <th>volatile acidity</th>
      <th>citric acid</th>
      <th>residual sugar</th>
      <th>chlorides</th>
      <th>free sulfur dioxide</th>
      <th>total sulfur dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7.0</td>
      <td>0.27</td>
      <td>0.36</td>
      <td>20.7</td>
      <td>0.045</td>
      <td>45.0</td>
      <td>170.0</td>
      <td>1.0010</td>
      <td>3.00</td>
      <td>0.45</td>
      <td>8.8</td>
      <td>6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6.3</td>
      <td>0.30</td>
      <td>0.34</td>
      <td>1.6</td>
      <td>0.049</td>
      <td>14.0</td>
      <td>132.0</td>
      <td>0.9940</td>
      <td>3.30</td>
      <td>0.49</td>
      <td>9.5</td>
      <td>6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8.1</td>
      <td>0.28</td>
      <td>0.40</td>
      <td>6.9</td>
      <td>0.050</td>
      <td>30.0</td>
      <td>97.0</td>
      <td>0.9951</td>
      <td>3.26</td>
      <td>0.44</td>
      <td>10.1</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7.2</td>
      <td>0.23</td>
      <td>0.32</td>
      <td>8.5</td>
      <td>0.058</td>
      <td>47.0</td>
      <td>186.0</td>
      <td>0.9956</td>
      <td>3.19</td>
      <td>0.40</td>
      <td>9.9</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.2</td>
      <td>0.23</td>
      <td>0.32</td>
      <td>8.5</td>
      <td>0.058</td>
      <td>47.0</td>
      <td>186.0</td>
      <td>0.9956</td>
      <td>3.19</td>
      <td>0.40</td>
      <td>9.9</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_white.describe().toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>summary</th>
      <th>fixed acidity</th>
      <th>volatile acidity</th>
      <th>citric acid</th>
      <th>residual sugar</th>
      <th>chlorides</th>
      <th>free sulfur dioxide</th>
      <th>total sulfur dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>count</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
      <td>4898</td>
    </tr>
    <tr>
      <th>1</th>
      <td>mean</td>
      <td>6.854787668436075</td>
      <td>0.27824111882401087</td>
      <td>0.33419150673743736</td>
      <td>6.391414863209486</td>
      <td>0.0457723560636995</td>
      <td>35.30808493262556</td>
      <td>138.36065741118824</td>
      <td>0.9940273764801896</td>
      <td>3.1882666394446693</td>
      <td>0.4898468762760325</td>
      <td>10.514267047774638</td>
      <td>5.87790935075541</td>
    </tr>
    <tr>
      <th>2</th>
      <td>stddev</td>
      <td>0.8438682276875127</td>
      <td>0.10079454842486532</td>
      <td>0.12101980420298254</td>
      <td>5.072057784014878</td>
      <td>0.021847968093728805</td>
      <td>17.00713732523259</td>
      <td>42.498064554142985</td>
      <td>0.002990906916936997</td>
      <td>0.15100059961506673</td>
      <td>0.11412583394883222</td>
      <td>1.23062056775732</td>
      <td>0.8856385749678322</td>
    </tr>
    <tr>
      <th>3</th>
      <td>min</td>
      <td>3.8</td>
      <td>0.08</td>
      <td>0.0</td>
      <td>0.6</td>
      <td>0.009</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>0.98711</td>
      <td>2.72</td>
      <td>0.22</td>
      <td>8.0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>max</td>
      <td>14.2</td>
      <td>1.1</td>
      <td>1.66</td>
      <td>65.8</td>
      <td>0.346</td>
      <td>289.0</td>
      <td>440.0</td>
      <td>1.03898</td>
      <td>3.82</td>
      <td>1.08</td>
      <td>14.2</td>
      <td>9</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_red.describe().toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>summary</th>
      <th>fixed acidity</th>
      <th>volatile acidity</th>
      <th>citric acid</th>
      <th>residual sugar</th>
      <th>chlorides</th>
      <th>free sulfur dioxide</th>
      <th>total sulfur dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>count</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
      <td>1599</td>
    </tr>
    <tr>
      <th>1</th>
      <td>mean</td>
      <td>8.319637273295838</td>
      <td>0.5278205128205131</td>
      <td>0.2709756097560964</td>
      <td>2.5388055034396517</td>
      <td>0.08746654158849257</td>
      <td>15.874921826141339</td>
      <td>46.46779237023139</td>
      <td>0.9967466791744831</td>
      <td>3.311113195747343</td>
      <td>0.6581488430268921</td>
      <td>10.422983114446502</td>
      <td>5.6360225140712945</td>
    </tr>
    <tr>
      <th>2</th>
      <td>stddev</td>
      <td>1.7410963181276948</td>
      <td>0.17905970415353525</td>
      <td>0.19480113740531824</td>
      <td>1.40992805950728</td>
      <td>0.047065302010090085</td>
      <td>10.46015696980971</td>
      <td>32.89532447829907</td>
      <td>0.0018873339538427265</td>
      <td>0.15438646490354271</td>
      <td>0.1695069795901101</td>
      <td>1.0656675818473935</td>
      <td>0.8075694397347051</td>
    </tr>
    <tr>
      <th>3</th>
      <td>min</td>
      <td>4.6</td>
      <td>0.12</td>
      <td>0.0</td>
      <td>0.9</td>
      <td>0.012</td>
      <td>1.0</td>
      <td>6.0</td>
      <td>0.99007</td>
      <td>2.74</td>
      <td>0.33</td>
      <td>8.4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>max</td>
      <td>15.9</td>
      <td>1.58</td>
      <td>1.0</td>
      <td>15.5</td>
      <td>0.611</td>
      <td>72.0</td>
      <td>289.0</td>
      <td>1.00369</td>
      <td>4.01</td>
      <td>2.0</td>
      <td>14.9</td>
      <td>8</td>
    </tr>
  </tbody>
</table>
</div>



We can clearly see here the differences between red and white wine features.
We can also see in the quality column that the mean of the white wine is bigger however with a bigger standard deviation.


```python
df_red.select([count(when(isnan(c), c)).alias(c) for c in df_red.columns]).toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fixed acidity</th>
      <th>volatile acidity</th>
      <th>citric acid</th>
      <th>residual sugar</th>
      <th>chlorides</th>
      <th>free sulfur dioxide</th>
      <th>total sulfur dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_white.select([count(when(isnan(c), c)).alias(c) for c in df_white.columns]).toPandas()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fixed acidity</th>
      <th>volatile acidity</th>
      <th>citric acid</th>
      <th>residual sugar</th>
      <th>chlorides</th>
      <th>free sulfur dioxide</th>
      <th>total sulfur dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Finally this last verification allow us to check if there is any 'Nan' values or null values.

Which as we can see is not the case here.

### Data visualisation

We will convert our two dataset two have to pandas to use pandas plotting methods on the datasets.
This should not be done with large size dataframe however this one is rather small so we can do it with too much trouble.
One way to do it on large datasets would be to select randomly a big enough (using statisticial methods to find the size) fraction of the dataset and then plot it.


```python
pd_df_red = df_red.toPandas().apply(pd.to_numeric,axis = 0 , errors='ignore')
pd_df_white = df_white.toPandas().apply(pd.to_numeric,axis = 0 , errors='ignore')
```


```python
fig, ax = plt.subplots(1,2, figsize = (10, 5))
sns.histplot(pd_df_red['quality'],ax = ax[0], kde=True)
sns.histplot(pd_df_white['quality'],ax = ax[1], kde=True)
ax[0].set_title('Red wine')
ax[1].set_title('White wine')
plt.show()
```


    
![alt](https://user-images.githubusercontent.com/104632559/235010955-13c59225-bd82-4837-bbed-9131c0ee2d13.png)
    



```python
mask = np.zeros_like(pd_df_red.corr(), dtype=bool)
mask[np.triu_indices_from(mask)] = True
f, ax = plt.subplots(figsize=(10, 7.5))
plt.title('Pearson Correlation Matrix for red wine',fontsize=23)
sns.heatmap(pd_df_red.corr(),linewidths=0.25, vmax=1.0, square=True, cmap="BuGn",
            linecolor='w', annot=True, mask=mask, cbar_kws={"shrink": .75})
plt.show()
```


    
![alt](https://user-images.githubusercontent.com/104632559/235010956-93773923-506c-4e14-b16a-bd3c0e4d949f.png)
    



```python
mask = np.zeros_like(pd_df_white.corr(), dtype=bool)
mask[np.triu_indices_from(mask)] = True
f, ax = plt.subplots(figsize=(10, 7.5))
plt.title('Pearson Correlation Matrix for red wine',fontsize=23)
sns.heatmap(pd_df_white.corr(),linewidths=0.25, vmax=1.0, square=True, cmap="BuGn",
            linecolor='w', annot=True, mask=mask, cbar_kws={"shrink": .75})
plt.show()
```


    
![alt](https://user-images.githubusercontent.com/104632559/235010958-2e08b38b-4535-416b-86f3-89995f5727e3.png)
    


We can see some correlation between some features.

### Finding the number of clusters


```python
#Generates elbow curve for df
def elbow_curve(df, nb_clusters = 10, featuresCol = 'features'):
    evaluator = ClusteringEvaluator(
        predictionCol='prediction', featuresCol=featuresCol,                        
        metricName='silhouette', distanceMeasure='squaredEuclidean')
    
    assemble=VectorAssembler(inputCols=df.columns, outputCol=featuresCol)
    assembled_data=assemble.transform(df)
    
    cost = np.zeros(nb_clusters)
    silhouette_score = np.zeros(nb_clusters)
    
    for k in range(2,nb_clusters):
        kmeans = KMeansml().setK(k).setSeed(1).setFeaturesCol(featuresCol)
        model = kmeans.fit(assembled_data)
        cost[k] = model.summary.trainingCost
        output = model.transform(assembled_data)
        silhouette_score[k] = evaluator.evaluate(output)
    
    return {'cost' : cost[2:], 'ss' :silhouette_score[2:] }
```


```python
df_white_dropped = df_white.drop('quality')
df_red_dropped = df_red.drop('quality')
```


```python
dct_white = elbow_curve(df_white_dropped)
dct_red = elbow_curve(df_red_dropped)
```

                                                                                    


```python
fig, ax = plt.subplots(2,2, figsize = (15,10))

ax[0,0].plot(range(2, 10), dct_white['cost'])
ax[0,0].set_title('Elbow curve white wine')
ax[0,0].set_xlabel('Number of cluster')
ax[0,0].set_ylabel('Score')

ax[0,1].plot(range(2, 10), dct_white['ss'])
ax[0,1].set_title('Silhouette score curve white wine')
ax[0,1].set_xlabel('Number of cluster')
ax[0,1].set_ylabel('Silhouette score')

ax[1,0].plot(range(2, 10), dct_red['cost'])
ax[1,0].set_title('Elbow curve red wine')
ax[1,0].set_xlabel('Number of cluster')
ax[1,0].set_ylabel('Score')

ax[1,1].plot(range(2, 10), dct_red['ss'])
ax[1,1].set_title('Silhouette score curve red wine')
ax[1,1].set_xlabel('Number of cluster')
ax[1,1].set_ylabel('Silhouette score')
plt.show()
```


    
![alt](https://user-images.githubusercontent.com/104632559/235010959-60782f35-8c8e-42c6-ad62-9cdb5e9a5f09.png)
    


For the white wine, is it obvious looking at both curbs that the number of cluster is 4. The elbow curve shows that the score could be either 3 or 4, however the silhouette score clearly show that the number of lcuster is 4.

For the white wine, it is harder to settle for a score considering both curbs. However we might settle for five I think.

Let's try the same thing but this time using normalization of the dataset before running the K-means algorithm.


```python
#Generates elbow curve for df
def elbow_curve_stand(df, nb_clusters = 10, featuresCol = 'standardized'):
    evaluator = ClusteringEvaluator(
        predictionCol='prediction', featuresCol=featuresCol,                        
        metricName='silhouette', distanceMeasure='squaredEuclidean')
    
    assemble=VectorAssembler(inputCols=df.columns, outputCol='features')
    assembled_data=assemble.transform(df)
    
    scale=StandardScaler(inputCol='features',outputCol=featuresCol)
    
    data_scale=scale.fit(assembled_data)
    data_scale_output=data_scale.transform(assembled_data)
    
    cost = np.zeros(nb_clusters)
    silhouette_score = np.zeros(nb_clusters)
    
    for k in range(2,nb_clusters):
        kmeans = KMeansml().setK(k).setSeed(1).setFeaturesCol(featuresCol)
        model = kmeans.fit(data_scale_output)
        cost[k] = model.summary.trainingCost
        output = model.transform(data_scale_output)
        silhouette_score[k] = evaluator.evaluate(output)
    
    return {'cost' : cost[2:], 'ss' :silhouette_score[2:]}
```


```python
dct_white = elbow_curve_stand(df_white_dropped)
dct_red = elbow_curve_stand(df_red_dropped)
```

                                                                                    


```python
fig, ax = plt.subplots(2,2, figsize = (15,10))

ax[0,0].plot(range(2, 10), dct_white['cost'])
ax[0,0].set_title('Elbow curve white wine')
ax[0,0].set_xlabel('Number of cluster')
ax[0,0].set_ylabel('Score')

ax[0,1].plot(range(2, 10), dct_white['ss'])
ax[0,1].set_title('Silhouette score curve white wine')
ax[0,1].set_xlabel('Number of cluster')
ax[0,1].set_ylabel('Silhouette score')

ax[1,0].plot(range(2, 10), dct_red['cost'])
ax[1,0].set_title('Elbow curve red wine')
ax[1,0].set_xlabel('Number of cluster')
ax[1,0].set_ylabel('Score')

ax[1,1].plot(range(2, 10), dct_red['ss'])
ax[1,1].set_title('Silhouette score curve red wine')
ax[1,1].set_xlabel('Number of cluster')
ax[1,1].set_ylabel('Silhouette score')
plt.show()
```


    
![alt](https://user-images.githubusercontent.com/104632559/235010961-98f6b23c-d842-4042-81a9-fb56793f0717.png)
    


Here we can see that both the white wine and red wine have 3 clusters.

### Cluster visualisation 


```python
def K_means_and_pca(df, nb_clusters, nb_comp = 2, featuresCol = 'standardized'):
    assemble=VectorAssembler(inputCols=df.columns, outputCol='features')
    assembled_data=assemble.transform(df)
    
    scale=StandardScaler(inputCol='features',outputCol=featuresCol)
    
    data_scale=scale.fit(assembled_data)
    data_scale_output=data_scale.transform(assembled_data)
    
    kmeans = KMeansml().setK(nb_clusters).setSeed(1).setFeaturesCol(featuresCol)
    model = kmeans.fit(data_scale_output)
    KMeans_Assignments = model.transform(data_scale_output)
    
    pca = PCA(k=nb_comp, inputCol=featuresCol, outputCol="pca")
    pca_model = pca.fit(data_scale_output)
    pca_transformed = pca_model.transform(data_scale_output)
    
    x_pca = np.array(pca_transformed.rdd.map(lambda row: row.pca).collect())
    cluster_assignment = np.array(KMeans_Assignments.rdd.map(lambda row: row.prediction).collect()).reshape(-1,1)
    
    pca_data = np.hstack((x_pca,cluster_assignment))
    
    if nb_comp == 2 : 
        pca_df = pd.DataFrame(data=pca_data, columns=("1st_principal", "2nd_principal","cluster_assignment"))
        sns.FacetGrid(pca_df,hue="cluster_assignment", height=6).map(plt.scatter, '1st_principal', '2nd_principal' ).add_legend()
        plt.show()
    
    else : 
        pca_df = pd.DataFrame(data=pca_data, columns=("1st_principal", "2nd_principal", "3rd_principal","cluster_assignment"))
        fig = plt.figure(figsize=(6,6))
        ax = Axes3D(fig, auto_add_to_figure=False)
        fig.add_axes(ax)
        colors = plt.cm.get_cmap('hsv', nb_clusters)
        for i in range(len(pca_df)):
            x, y, z = pca_df.iloc[i]['1st_principal'], pca_df.iloc[i]['2nd_principal'], pca_df.iloc[i]['3rd_principal']
            color = rgb2hex(colors(int(pca_df.iloc[i]['cluster_assignment'])))
            ax.scatter(x, y, z, c= color)
        plt.show()
        
        
    
    
```

#### White wine clusters with PCA dimensionality reduction for visualisation


```python
K_means_and_pca(df_white_dropped,nb_comp =3,nb_clusters = 3)
```

    23/02/05 00:56:11 WARN InstanceBuilder$NativeBLAS: Failed to load implementation from:dev.ludovic.netlib.blas.JNIBLAS
    23/02/05 00:56:11 WARN InstanceBuilder$NativeBLAS: Failed to load implementation from:dev.ludovic.netlib.blas.ForeignLinkerBLAS
    23/02/05 00:56:12 WARN LAPACK: Failed to load implementation from: com.github.fommil.netlib.NativeSystemLAPACK
    23/02/05 00:56:12 WARN LAPACK: Failed to load implementation from: com.github.fommil.netlib.NativeRefLAPACK
                                                                                    


    
![alt](https://user-images.githubusercontent.com/104632559/235010962-3148dc77-6aec-4baf-92d8-64f7babf57f6.png)
    



```python
K_means_and_pca(df_white_dropped,nb_comp =2,nb_clusters = 3)
```

                                                                                    


    
![png](output_40_1.png)
    


#### Red wine clusters with PCA dimensionality reduction for visualisation


```python
K_means_and_pca(df_red_dropped,nb_comp =3,nb_clusters = 3)
```


    
![png](output_42_0.png)
    



```python
K_means_and_pca(df_red_dropped,nb_comp =2,nb_clusters = 3)
```


    
![alt](https://user-images.githubusercontent.com/104632559/235010963-bcbed4ae-69a1-4a15-bf10-a0b42b9e0383.png)
    


### Computation time comparison

Let's two functions which will perform K-means algorithm on the raw dataframe one using scikit and one using pyspark.

We will also add to our function the normalisation process in order to compare the whole pipeline training process.


```python
def sprk_k_means(df, nb_clusters = 4):
    assemble=VectorAssembler(inputCols=df.columns, outputCol='features')
    assembled_data=assemble.transform(df)
    kmeans = KMeansml().setK(nb_clusters).setSeed(1).setFeaturesCol('features')
    model = kmeans.fit(assembled_data)
    KMeans_Assignments = model.transform(assembled_data)
    return KMeans_Assignments

def skl_k_means(df, nb_clusters = 4):
    return KMeans(n_clusters=nb_clusters, n_init = 1).fit_transform(df)
```


```python
%timeit sprk_k_means(df_red_dropped)
```

    3.14 s ± 307 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)



```python
pd_df_red_dropped = df_red_dropped.toPandas().apply(pd.to_numeric, axis = 0, errors = 'ignore').values
%timeit skl_k_means(pd_df_red_dropped)
```

It appears to be impossible to generate time mesurements using sklearn due to the jupyter kernel crashing.

My analysis of the cluster is that as we saw in the quality graph analysis there are three pools of quality. High quality, medium quality and low quality.

Clusters could be a way to separate those wines in order to have an insight on their quality.