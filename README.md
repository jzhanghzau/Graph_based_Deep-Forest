# Graph-based Deep Forest (Graph-based gcForest)
Master thesis, Junjie Zhang
## Introduction
We propose graph-based gcForest, which is an ensemble learning framework that is capable of scanning on a graph. In graph-based gcForest, inspired by the second-order random walk in Node2Vec, we propose a non-linear graph-based Multi-Grained Scanning method, which provides us with the flexibility of feature representation based on the intrinsic graph structure.

Graph-based gcForest is a variant of gcForest. gcForest consists of two components, Cascade Forest and Sliding-window based Multi-Grained Scanning, respectively.
The modifications to the graph-based gcForest focus on the Multi-grained Scanning. We propose a Graph-based Multi-grained Scanning and still employ Cascade Forest.

### Cascade Forest
Illustration of a the cascade forest structure. Each level contains two Random Forests(black) and two complete Random Forests(blue), the red ones are the original features. Suppose there is triple classification problem; Therefore, each forest generates a three-dimensional class vector, which is then  concatenated for re-representation of the original features.
<img src="https://github.com/jzhanghzau/Graph-based-DeepForest/blob/master/images/cascade_structure.PNG" width="600" height="300">

### Sliding-window based Multi-Grained Scanning
Illustration of the sliding-window based Multi-Grained Scanning. Suppose there are three classes to be predicted, original features have 400 dimensions and silding window has 100 dimensions.
<img src="https://github.com/jzhanghzau/Graph-based-DeepForest/blob/master/images/multi-grained.PNG" width="600" height="300">

### Overall structure of gcForest
Illustration of the whole procedure of gcForest. Suppose there are three classes to predict, raw input features are 400 dimensional, and three sliding window sizes(100-dim, 200-dim, 300-dim) are used.
<img src="https://github.com/jzhanghzau/Graph-based-DeepForest/blob/master/images/overall.PNG" width="800" height="400">

### Graph-based Multi-Grained Scanning
Illustration of graph-based Multi-Grained Scanning. Suppose there are three classes, raw input features are 1000-dim, the corresponding graph contains 1000 nodes, the walk length used is 200 nodes and the number of generated walks is set to 300.
<img src="https://github.com/jzhanghzau/Graph-based-DeepForest/blob/master/images/graph-based_silding.PNG" width="600" height="300">

### Overall structure of graph-based gcForest
Illustration of overall procedure of graph-based gcForest. Suppose there are three classes to predict, raw input features are 1000 dimensional, corresponding graph contains 1000 nodes, three walk lengths(400 nodes, 600 nodes, 800 nodes) are used and the number of generated walks is set to 300, 400, 500, respectively.
<img src="https://github.com/jzhanghzau/Graph-based-DeepForest/blob/master/images/overall_graphGC.png" width="800" height="400">


## Packages
Python & 3.7.7

scikit-learn & 0.22.1

Numpy & 1.18.1

Pandas & 1.0.2

Networkx & 2.4



## Usage example

Create a Random Forest:
```python
clf1 = Be.RFC(n_estimators=25, max_depth=None, random_state=None, n_jobs=-1)
```
Create a completely Random Forest:
```python
clf2 = Be.RFC(n_estimators=25, max_depth=None, max_features=1, random_state=None,
              n_jobs=-1)
```
Build two layers and fill each layer with  classifier's set.
```python
c = {"crf1": clf1, "crf2": clf2}
layer1 = layer()
layer1.add(**c)
layer2 = layer()
layer2.add(**c)
```
Import Cascade Forest, initialize the cascade forest structure, save the model generated in the validation step into the 'yeast4' directory, the accuracy is used as metrics.
```python
from Cascade_Forest.Cascade_Forest import cascade_forest
cs1 = cascade_forest(random_state=None, n_jobs=-1, directory='yeast4', metrics='accuracy')
```
Add each layer to the cascade forest structure.
```python
cs1.add(layer1)
cs1.add(layer2)
```
The training and testing process of Cascade Forest
```python
cs1.fit([X_train], y_train)
cs1.predict([X_test])
score = cs1.score(y_test)
print("cascade forest's accuracy :{:.2f} %".format(score * 100))
```
-------------------------------gcForest-----------------------------------------

Import Multi-Grained Scanning, initialize a Sliding-window based Multi-grained Scanner.
```python
from Cascade_Forest.Multi_grained_scanning import scanner
sc1 = scanner(window_size=(24, 48, 96), stratify=True, clf_set=(clf1, clf2), n_splits=3, random_state=None)
```
Transform the features through the Sliding-window based Multi-grained Scanning, and then training and testing through Cascade Forest.
```python
transformed_train1, transformed_test1 = sc1.transform_feature(X_train, y_train, X_test)
"Training"
cs1.fit(transformed_train1, y_train)
"Predicting"
cs1.predict(transformed_test1)
"The accuracy score"
score = cs1.score(y_test)
print("gcForest's accuracy :{:.2f} %".format(score * 100))
```
-------------------------------Graph-based gcForest----------------------------------

Import Multi-Grained Scanning, initialize a graph-based Multi-grained Scanner.

```python
from Cascade_Forest.Multi_grained_scanning import scanner
sc1 = scanner(stratify=True, clf_set=(clf1, clf2), n_splits=3, random_state=None,
              walk_length=(24, 48, 96), num_walks=1, p=100, q=100, scale=(80, 56, 8))
```
Import Networkx that could handle graph structure. Transform the features through the graph-based Multi-grained Scanning, and then training and testing through Cascade Forest.
```python
import networkx as nx
transformed_train2, transformed_test2 = sc2.graph_embedding(G1, X_train, y_train, X_test)
"Training"
cs1.fit(transformed_train2, y_train)
"Predicting"
cs1.predict(transformed_test2)
"The accuracy score"
score = cs1.score(y_test)
print("Graph_based gcForest's accuracy :{:.2f} %".format(score * 100))
```
## Parameters
- ### `Cascade Structure`:
    1. `metrics`: The metrics for validation step, decide how does the cascade structure grow, `{'accuracy', 'MCC', 'F1 score binary', 'F1 score weighted'},       default: accuracy`.
    2. `tolerance`: How much improvement of metrics will lead the cascade structure grow deeper, `default:0`.
    3. `stratify`: For train_test_spilt and k-fold cross validation, `default:True`.
    4. `directory`: The folder where the model generated during the validation step is saved.
    5. `test_size`: How much of the train_data will be used as test data in validation step, `default:0.2`.
  
- ### `Sliding window based Multi-Grained Scanning`:

    1. `window_size`: The sliding window size, `default:(1,1)`.
    2. `clf_set`: The classifiers used to transform the original input features.
  
- ### `Graph-based Multi-Grained Scanning`:   
      
    1. `walk_length`: Number of nodes in each walk, `default:(1,1)`.
    2. `num_walks`:  Number of walks per node, `default:1`.
    3. `p`: Return hyper parameter, `default:1`.
    4. `q`: Inout parameter, `default:1`.
    5. `scale`: The number of generated walks you need. Suppose there are 100 features in your dataset, but the corresponding graph structure has only 90 nodes.             With `num_walks` equal to 1, the scanner will only generate 90 walks. If you need 100 walks, you have to set `num_walks` to 2, thus it will generate 200             walks, then set `scale` to 100 and will get 100 walks.
    6. `clf_set`: The classifiers used to transform the original input features.
    
    

## Methods
- ### `Layer`:
    1. `add(**X)`: Add a classifier set into layer obeject. X is a classifier set, such as `X = {"crf1": clf1, "crf2": clf2}`.
  
- ### `Cascade Forest`:
    1. `add(X)`: Add a layer obeject into Cascade Forest. X is a layer object.
    2. `fit([X_train], y_train)`: Build a Cascade Forest from the training set. 
    3. `predict([X_test])`: Predict class for X_test.
    4. `score`: Return the sore on the given test data and labels.

- ### `Sliding window based scanner`:
   
    1. `transform_feature(X_train, y_train, X_test)`: Transform the X_train, X_test by using the sliding window based scanner.

- ### `Graph-based based scanner`:
   
    1. `graph_embedding(G1, X_train, y_train, X_test)`: `G1` argument has to be networkx graph. Transform the X_train, X_test by using the graph-based scanner.



    
    
    


