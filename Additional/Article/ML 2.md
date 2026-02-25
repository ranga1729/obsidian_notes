Let's understand the inner working of a machine learning model using Linear Regression. 
##### Terminology
- The dataset used to train the model is called the **"training set"**. 
- In supervised learning, the training dataset is consist of,
	1. Inputs called the **"input variables"** or **"features"**. Denoted by a lowercase "$x$".
	2. Outputs(correct answers) called the **"output variable"** or the **"target variable"**. Denoted by a lowercase "$y$". 
- In a training set, each row represents a different **""training example"**. 
- A single training example = $(x, y)$
- To denote the $i^{th}$ training example = $(x^{i}, y^{i})$ 
- The total number of training examples are denoted by "$m$".

#### Linear Regression
Linear regression model is a supervised learning algorithm(regression category) used for predicting a continuous numerical values based on one or more input features.

Ex: House price prediction
![[Pasted image 20250921101035.png]]

To train the model, we feed our training set to the learning algorithm. Then our supervised learning algorithm will produce a function $f$ called the "**hypothesis**".

The job of this function is to take new input $x$ and estimate a prediction, denoted by "$\hat{y}$".
(Normal $y$ is to denote targets of the training set.)
The function $f$ is called the "**model**".
![[Pasted image 20250921103854.png]]

For now, lets assume our function is a straight line. 
![[lagrida_latex_editor.png]]

![[Pasted image 20250921105926.png]]
