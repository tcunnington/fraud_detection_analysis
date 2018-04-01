# AMATH 582 Project -- Imbalanced Data -- Fraud Detection Problem


### Goal of the project:
To consider classification methods in the presence of highly imbalanced data--binary data where a majority class far outnumbers the minority class. We will cover how to evaluate methods in a way that captures the importance of classifying the minority class correctly, as well as ways to improve these methods.

### Background

The dataset we use for this project is a perfect example of imbalanced data: credit card transactions from European cardholders where some transactions are fraudulent. To a credit card company instances of fraud are relatively rare in comparison to all transactions, but they are particularly important to be able to identify because they are very costly.

Classifying bank transactions as fraudulent comes with a few challenges: (1) data is highly sensitive and therefore difficult to come by, (2) relevant features are not always clear up front, and (3) classification algorithms have trouble because fraud is (hopefully) much less common than honest purchases (imbalanced data).
To address the first problem, we analyze real credit card data from a challenge on Kaggle.com (https://www.kaggle.com/mlg-ulb/creditcardfraud). However the features have been obscured by a singular value decomposition (PCA) to preserve confidentiality: we are given just the principle values of the original data. This is essentially the U matrix from A = UΣV^T in the usual SVD.
This means we have no way to address problem 2, feature extraction, and so we will not attempt anything in this regard. Instead, we focus on classification algorithms.

There are two main challenges for us in classifying this data:
1. Many classification algorithms don't predict the minority class well when trained on such data.
2. The common classifier performance metrics do not align with our preference for predicting the minority class when working with data such as this.

That popular algorithms have trouble with skew is not all that surprising: these algorithms typically work to minimize the error rate, which inherently favors the majority class. As an example think about what happens if you simply classified everything as the more prevalent class. Our dataset is very imbalanced: only 0.17% of the data points are classified as the minority class. This means that if we always guess the majority class we will get 99.83% accuracy! This also tells us that accuracy (or its inverse error) is not actually the metric we wish to optimize. What we actually care _much_ more about is reducing false negatives (FNs: someone gets away with fraud). We do of course care about false positives as well (FPs: mistakenly flagging a transaction as fraud), but to a much lesser extent. Here is is hopefully clear that FNs would be more expensive to a bank than FPs. Generally speaking, if you are _interested_ in "anomaly detection" situations this will be true. More advantageous metrics than accuracy are recall and the F1 score, as discussed below.

We will cover several methods learned in lecture including quadratic discriminant analysis (QDA), a binary classification tree (CART), and logistic regression.


#### Performance metrics

The baseline metrics for evaluating a classification scheme are all contained in the "confusion matrix". Remembering that positive is a fraudulent result, we can construct a matrix which counts the total number of true positive (TP), false positive (FP), true negative (TN), and false negative (FN) labels: C = [TP FN; FP TN]

Using these terms can can define precision: TP/(TP+FP), recall: TP/(TP+FN), accuracy: (TP+TN)/(TP+TN+FP+FN), and more. While this is a convenient construct it does not provide a singular metric for determining performance. For that we will use recall, or the fraction of positive entries that you correctly identified as positive.
We chose this because identifying fraud, meaning reducing FNs, is far and away the most important goal of our classifiers. For this we are willing to sacrifice precision and accuracy since, again, we assume that FNs are much more costly (or interesting) compared to FNs. We will see that accuracy is nearly always high anyway (as explained earlier). Because our classes are not well separated (see either iPython notebook) we will have to expect some trade offs.

If we still want to consider precision we can do so as part of the F1 score.

Another alternative, which we use in our results, is the geometric mean: sqrt(TPR*TNR). Since this includes the recall (TPR) and still takes into account that it is important to classify negatives correctly (with the TNR) it covers our primary and secondary concern.
The TNR will almost always be high so this will mostly be driven by the recall unless TNR drops significantly.


Precision and recall are often considered together in the aptly named Precision-Recall Curve (PRC), or in the related metric: Area Under the Precision-Recall Curve (AUPRC). We will consider this as well as ROC curves.

#### Methods for imbalanced data

There are some known methods for improving performance when classifying imbalanced data. The four common solution types are given here:
1. Under-sampling – Removing data from the larger class to balance the class sizes. May lead to a poorer choice of decision line due to having less information.
2. Oversampling – Adding additional observations "on top" of existing minority class observations to balance out the class sizes. May lead to overfitting with some classification models since new points are at the exact same locations.
3. Synthetic data generation – Generating artificial data for your minority class from your existing data. Generated data generally stays within the volume that minimally encloses the existing data.
4. Cost functions– Classification algorithms that use cost functions (decision functions) to define their decision boundaries can weigh a FN more heavily than other types of error to encourage the algorithm to essentially prioritize recall.

The first three methods attempt to reduce the class imbalance by reducing the number of majority data point or increasing the number of minority data points. For synthetic data generation we chose to use the SMOTE (Synthetic Minority Over-Sampling Technique) and ADASYN (Adaptive Synthetic Sampling Approach for Imbalanced Learning) algorithms.
SMOTE is ADASYN’s precursor and ADASYN is essentially a specific implementation of of SMOTE.
The goal of both is to improve the class balance by increasing the number of minority class members. SMOTE places synthetic data between existing data points randomly (linear interpolation between neighbors), with no preference shown to any specific points.
ADASYN does the same thing but places more synthetic data points close to the boundary between classes because those are the original data points that are more difficult to learn.

### Results

For our results we will show just the performance of our logistic regression classifier.
Our primary metric is the geometric mean, since it is a composite of the TPR (recall) and the TNR (see metrics section).
The bar plots below shows that all methods did much better than the baseline.

![ROC curve](img/geometric_mean_compare.png?raw=true)

ADASYN, SMOTE, and balanced do best generally, where "balanced" refers to altering the logistic regression algorithm to weight the minority class higher (inversely proportional to it's relative representation in the training data). Just one instance of training/test data is shown here.

Next we have the precision-recall curve.  

![Precision-recall](img/pr_compare.png?raw=true)

SMOTE and ADASYN have the highest AUC scores, although many seem to perform similarly in the high-recall/low precision region which we would certainly choose to operate in.
Class balanced model and oversampling follow.

Lastly we have the ROC curve. Again all the methods solidly outperform the baseline, although for the most part their performance is hard to distinguish from each other here, and all have similar AUC values.
![ROC curve](img/roc_compare.png?raw=true)

Overall we can easily conclude that all of the proposed methods are improvements on the baseline.
The SMOTE, ADASYN and balanced cases performs well in all plots. Balanced had a slightly better geometric mean, which is our primary metric. Oversampling and undersampling appear to be inferior to other methods. Undersampling seems like the weakest according to the ROC and PR curves--perhaps this is explained by the fact that you are losing information to work with.
Based on these results, if I were using logistic regression I would likely just use the balanced class weights since it is the simplest to implement.
If I were using other classification methods I would always compare with SMOTE (it runs faster than ADASYN) if I suspected that imbalanced classes could be a problem, or if I cared about recall more than accuracy.


### Classification algorithms

##### Gaussian methods

Gaussian methods such as linear discriminant analysis and quadratic discriminant analysis attempt to fit a multivariate normal distribution to existing data.
In LDA the covariance matrix is shared by both classes, whereas in QDA they independent. Naive Bayes uses independent covariances but they are constrained to be diagonal (features are uncorrelated).
Each class also has a prior probability that determines how likely each class is to be found (calculated as the fraction of one class to the other by default).
Due to this prior a rare class will have an extremely small probability of occurring and will not be likely unless the two classes have a very good separation, or the minority class is compact in feature space.
If it were you could model it as a sharply peaked MVN that might be able to rise above the advantage the majority class has due to its much larger prior.
This immediately suggests that LDA is a poor choice since it forces a shared covariance matrix--this means that the minority class cannot be sharply peaked relative to the majority class distribution.
For this reason we will consider QDA instead.

##### Decision Trees

Binary decision trees are a common and effective choice for classification tasks. You classify an unknown observation by starting at the root node and at each node in the tree you make binary decision (meaning there are two branches) based on the features of the data used for training. You work your way up the tree until you hit a terminal leaf node.
This node determines your prediction.
You can think about this process as splitting up m-dimensional space into “blocks” (if you imagine the 3-dimensional version), where m is the number of features and each block represents a class prediction.
It starts with the full space and splits it into two, then one of those subspaces in two, and then one of those three subspaces into two, and onwards until it finishes.
The algorithm chooses where to put the boundaries based on minimizing the "Gigi index".
The Gini index naturally gives high weight to the more represented class, which is a bias for the majority class. One potential issue to keep in mind is overfitting.
Trees are known to overfit if you have a high number of features (we have ~30), and since there are relatively few samples of the minority class (492 out of 284807), we may not have enough data on the minority class to make a robust classifier.

##### K-nearest neighbors

##### Logistic regression

Logistic regression is a method that generates probabilities that a test point is in each class using the logistic regression function.


### References and usefulness
http://contrib.scikit-learn.org/imbalanced-learn/stable/user_guide.html
https://www3.nd.edu/~dial/publications/dalpozzolo2015calibrating.pdf
https://en.wikipedia.org/wiki/Receiver_operating_characteristic#Area_under_the_curve
http://machinelearningmastery.com/classification-and-regression-trees-for-machine-learning/
http://danielhnyk.cz/creating-your-own-estimator-scikit-learn/


## Git workflow:
1. `git commit -m "fix typos"`
2. `git pull --rebase`
3. Resolve any conflicts
4. `git push`
