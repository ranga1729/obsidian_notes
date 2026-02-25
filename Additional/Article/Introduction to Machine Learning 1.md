Recently I started studying Machine learning and I could get an overview understanding how these machine learning models are working and what's happen inside the model when we train them using a training data set.

![[Pasted image 20250919072011.png]]
#### What is "Machine Learning" ?
The term "Machine Learning" was first popularized by the "Arthur Samuel" in 1959. He was a pioneer in computer gaming and artificial intelligence at IBM. 
![[Pasted image 20250919072425.png]]
"Machine learning is the field of study that gives computers the ability to learn without explicitly being programmed." - Arthur Samuel

Arthur Samuel's checkers-playing program was a turning point in the machine learning history which was the first real example of ML in action.

Arthur Samuel Started developing in the early 1950s. 
Samuel built the game in a way that it could improve by playing games and learning from them, instead of just following fixed rules. 

In simple terms, 
His checkers game used a search tree to look ahead at possible moves. But because the tree could get huge, he created an evaluation function to judge board positions and prune the search. 
Samuel let the program play against itself and each time, it adjusted weights of its evaluation function based on game outcomes. 
Over the time, it discovered better strategies and eventually it reached a level where it could beat Samuel himself. (The invention wins over the inventor)
### Fundamentals of  ML
Machine learning is a branch of Artificial Intelligence(AI). 

The core idea of the machine learning if focused on creating models that can make predictions, classify data or find hidden patterns within data sets. 

Machine learning can be broadly divided into two main categories, 
1. Supervised Learning
2. Unsupervised learning
##### 1. Supervised Learning
Supervised learning is a type of machine learning where a model is trained on a labeled dataset. 
Main goal is to let the algorithm learn from input-output mapping so it can predict outputs for new, unseen inputs. 

Ex: 

| Input(x) | Output(y)                | Application          |
| -------- | ------------------------ | -------------------- |
| Email    | Spam or not?(true/false) | Spam filtering       |
| Audio    | Test transcript          | Speech recognition   |
| English  | Spanish translation      | Language translation |
Key point of supervised learning is, we are the one who provide both input and outputs(the training set) for the algorithm. 
Ex: 
	Set of emails labeled as if they are spam or not

After the model has learned from these input-output pairs, they can take a brand new input and try to produce a appropriate corresponding output. 

Supervised learning algorithms are mainly two types,
1. Regression algorithms
2. Classification algorithms

**1. Regression Algorithms**
Purpose is to predict continuous value for a given input.
Expected output is a real valued number. 
Ex:
	Predicting house prices
	![[Pasted image 20250921093728.png]]

**2. Classification Algorithms**
Purpose is to predict a categorical label for a given input. 
Expected output is a single label.

Ex: spam or not spam, disease or not disease
Ex: Breast cancer detection system : Based on the parents data/lump size, it will classify inputs into two categories, "Malignant"(cancer) and "Benign"(not cancer).
	![[Pasted image 20250921093555.png]]

Classification algorithms can be again divided into,
- Binary classification(2 classes) - ex: spam or not spam
- Multiclass classification(3 or more classes) - ex: classify CT scan images based on the organ captured

In supervised learning, we can use more than 2 input values.
Ex: Instead of using just tumor sizes, we can use patient age too. 
	![[Pasted image 20250921094937.png]]
	In here, the learning algorithm will find a boundary line that separates out the malignant and benign tumors. The algorithm will decide where to put the line through data. 

##### 2. Unsupervised Learning
In supervised learning, algorithm is given unlabeled data, no predefined outputs or categories. 
The goal of the model is to find patterns, structures or relationships in the data on its own. 

Common tasks in unsupervised learning,
- Clustering : Grouping similar data
- Dimensionality reduction : Simplifying data while keeping important features. 
- Anomaly detection : Finding unusual data points. 

Ex: You give the system a bunch of images of animals without labeling which ones are cats, dogs, birds. The system might group images into clusters based on visual similarities. 

****

Add reinforcement learnig