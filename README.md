# Introduction to Machine Learning

## Course description

Machine learning is one of the main concepts behind data science and artificial intelligence (AI). The term machine learning or statistical learning refers to the science of automated detection of patterns in data. It has been widely used in tasks that require information extraction from large data sets. Examples of tasks include SPAM detection, fraudulent credit card transaction detection, face recognition by digital cameras, and voice commands recognition by personal assistance on smart-phones. Machine learning is also widely used in scientific domains such as Bioinformatics, medicine, and astronomy. One characteristic of all these applications is that a human developer cannot provide an explicit and detailed specification of how these tasks should be executed, due to the complexity of the patterns that need to be detected.

## Course objectives

This course aims to introduce the main concepts underlying machine learning, including for instance, (a) what is learning,  (b) how can a machine learning, (c) what kind of problems can be solved by using machine learning approach, (d) how to formalize them as a machine learning problem, and (e) how to compare and evaluate the performance of different machine learning. We will focus on methods that are successfully used in practice, including regression, supervised and unsupervised techniques, and neural networks.

### Topics
 * Introduction to machine learning
 * Supervised learning: linear regression, logistic regression, linear discriminant analysis, k-nearest neighbors (kNN), decision trees, support vector machines (SVM), and boosting
 * Unsupervised learning: k-means, principal component analysis (PCA)
 * Model selection and validation
 * Neural networks

## Grading scheme

Grades will be based on:

 * Lab assignments: 10 pts 
 * Final project report: 40 pts -- ([Guidelines](04-project/guidelines.MD))
 * Written exam: 50 pts

### Assignments

An assignment will be set after each lecture. One point per assignment will be given based on a reasonable solution to the proposed problem. A reasonable solution is one that shows you have attempted to solve the problem.

### Final project

Students must organize into groups (up to 4 members) and they must implement a machine learning project that is (a) relevant to the topics discussed in class, (b) requires a significant effort from all team members, and (c) be unique (i.e., two groups may not choose the same project topic). The projects will vary in both scope and topic, but they must satisfy these criteria. Students are encouraged to begin to think about projects that interest them early on. If a group is unable to come up with its own project idea, it may ask the instructor to provide suggestions on interesting topics.

[Project Report Format](04-project/guidelines.MD)

### Written exam

It will be a written close book exam with questions based on the mandatory readings and topics discussed during the classes.

## References

1. Hastie, T., Tibshirani, R., and Friedman, J. (2016). [The Elements of Statistical Learning: Data Mining, Inference, and Prediction](http://web.stanford.edu/~hastie/ElemStatLearn/). Springer, 2nd edition
2. Valiant, L. (2013). Probably Approximately Correct: Nature's Algorithms for Learning and Prospering in a Complex World. Basic Books, Inc
3. Daumé III, H. (2017). [A Course in Machine Learning](http://ciml.info/dl/v0_99/ciml-v0_99-all.pdf). Self-published, 2nd edition 
4. James, G., Witten, D., Hastie, T., and Tibshirani, R. (2013). [An introduction to statistical learning](http://www-bcf.usc.edu/~gareth/ISL/). Springer
5. Goodfellow, I., Bengio, Y., and Courville, A. (2016). [Deep learning](https://www.deeplearningbook.org/). MIT press. 
6. Azencott, C.-A. (2018). Introduction au Machine Learning. Dunod

## Resources

* [Google Dataset Search](https://toolbox.google.com/datasetsearch)
* [UC Irvine Machine Learning Repository](https://archive.ics.uci.edu/ml/index.php)
* [Kaggle datasets](https://www.kaggle.com/datasets)
* [Google Colaboratory](https://colab.research.google.com)
* [Jupyter Notebook](https://jupyter.org/)
* [scikit-learn](https://scikit-learn.org/stable/documentation.html)

## Schedule

1. **Introduction** [slides](01-lectures/01_introduction.pdf); [assignment 01](02-assignments/assignments_01.pdf)
	
	This lectures introduces machine learning, its applications, and the kind of problems it can be applied. Likewise, it presents related concepts such as supervised and unsupervised learning, and generalization.
	
2. **Linear and logistic regression** [lecture notes](01-lectures/02_linear_and_logistic_regression.pdf); [lab 01: introduction](03-labs/lab01_manipulating_data.ipynb), [lab 02: regression](03-labs/lab02_regression_algorithms.ipynb); [assignment 02](02-assignments/assignments_02.pdf)

	This lecture introduces parametric approaches to supervised learning and linear models. Linear regressors are expressed as maximum likelihood estimation problem. The discussed concepts include: (a) parametric methods, (b) maximum likelihood estimates, (c) linear regression, and (d) logistic regression.
	
	**References**:
	- Tsanas A, Xifara A. [**Accurate quantitative estimation of energy performance of residential buildings using statistical machine learning tools**](http://people.maths.ox.ac.uk/tsanas/Preprints/ENB2012.pdf). Energy and Buildings. 2012 Jun 1;49:560-7.
	- [Linear Models](http://ciml.info/dl/v0_99/ciml-v0_99-ch07.pdf)
	
3. **Dimension reduction** [slides](01-lectures/03_dimension_reduction.pdf); [lab 03: dimension reduction](03-labs/lab03_dimensionality_reduction.ipynb)

	This lectures discusses how to tackle high-dimensional learning problems, and how to reduce dimension through the **principal component analysis (PCA)** method.
	
	**References**:
	 - [An Introduction to Feature Selection](https://link.springer.com/chapter/10.1007/978-1-4614-6849-3_19)
	 - [An introduction to variable and feature selection](http://jmlr.org/papers/v3/guyon03a.html)
	 - [A tutorial on principal component analysis](https://arxiv.org/abs/1404.1100)
	 - [Visualizing data using t-SNE](http://jmlr.org/papers/volume9/vandermaaten08a/vandermaaten08a.pdf)
	 - [How to Use t-SNE Effectively](https://distill.pub/2016/misread-tsne)
	 - [Linear Dimensionality Reduction](http://ciml.info/dl/v0_99/ciml-v0_99-ch15.pdf)
	 - [sklearn.decomposition.PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html)

4. **Model evaluation and selection** [slides](01-lectures/04_model_assessment.pdf); [lab 04: model assessment](03-labs/lab04_model_assessment.ipynb)

	This lecture discusses model assessment of supervised machine learning. The discussed topics include: (a) training and test sets, (b) cross-validation, (c) bootstrap, (d) metrics of model complexity, and (e) metrics of performance for classification and regression. 
	
5. **Regularized linear regression and nearest-neighbors methods** [slides](01-lectures/05_regularized_linear_regression.pdf)[lab 05: kNN](03-labs/lab05_kNN.ipynb)

	This lecture introduces the concept of regularization as a means to controlling the complexity of the hypothesis space, and apply it to linear models. Furthermore, non-parametric methods are illustrated with the nearest-neighbors approaches. The discussed topics are: lasso, ridge regression, structured regularization, non-parametric learning, and [k-nearest neighbors](01-lectures/05.1_kNN.pdf).

   **References**:
	
	  - [Linear Models](http://ciml.info/dl/v0_99/ciml-v0_99-ch07.pdf)
	  - [Nearest Neighbors](http://ciml.info/dl/v0_99/ciml-v0_99-ch03.pdf)
	  - [The Elements of Statistical Learning: Data Mining, Inference, and Prediction](http://web.stanford.edu/~hastie/ElemStatLearn/)
	     * **Ridge regression**: session 3.4.1
	     * **Lasso**: session 3.4.2
	     * **Regularization**: session 10.12
	
6. **Tree-based methods** [slides](01-lectures/06_tree_based_learning_methods.pdf); [lab 06.1: decision trees](03-labs/lab06_decision_tree.ipynb); [lab 06.2: tree-based methods](03-labs/lab06_tree_based_methods.ipynb)
	
	This lecture discusses decision tree approaches and shows how to combine simple classifiers to yield state-of-the-art predictors.
	
	 **References**:
	 
	  - Quinlan, J. Ross. [Induction of decision trees](01-lectures/quinlan.pdf). Machine Learning 1, no. 1 (1986): 81-106
	  - Breiman, Leo. [Random forests](https://link.springer.com/content/pdf/10.1023%2FA%3A1010933404324.pdf) Machine Learning 45, no. 1 (2001): 5-32
	  - Schapire, Robert E. [The strength of weak learnability](https://link.springer.com/content/pdf/10.1023%2FA%3A1022648800760.pdf). Machine learning 5, no. 2 (1990): 197-227
	  - Breiman, Leo. [Bagging predictors](https://link.springer.com/content/pdf/10.1007/BF00058655.pdf). Machine Learning 24, no. 2 (1996): 123-140.
	  - [An Intoductory Tutorial on kd-trees](https://www.ri.cmu.edu/pub_files/pub1/moore_andrew_1991_1/moore_andrew_1991_1.pdf)
	  - [Voronoi Tessellation](https://philogb.github.io/blog/2010/02/12/voronoi-tessellation/)
	  - [A complete tutorial on tree-based modeling](https://www.analyticsvidhya.com/blog/2016/04/complete-tutorial-tree-based-modeling-scratch-in-python/#ten)
	  - [How to visualize decision trees](https://explained.ai/decision-tree-viz/index.html)
	
7. **Support vector machines**

	This lecture introduces support-vector machine from its principles in the case of linearly separable data and shows how positive-definite kernels can be used to extend the approach to non-linear separating functions.
	
	**References**:
	
	  - Burges, Christopher JC. [A tutorial on support vector machines for pattern recognition](01-lectures/Burges1998.pdf). Data mining and knowledge discovery 2, no. 2 (1998): 121-167
  
	
8. **Clustering** [slides](01-lectures/08_clustering.pdf); [lab 08: k-means](03-labs/clustering.ipynb)
	
	This lecture introduces clustering, the common unsupervised learning problem. Its concepts are illustrated through hierarchical clustering, k-means, and DBSCAN.
	 
<!--9. **Neural networks** [slides](); [lab 09: neural_networks]()

	This lecture introduces the perceptron algorithm, multi-layer networks, backpropagation. Furthermore, it briefly discusses about recent advances in this area such as convolution neural network (CNN) and generative adversarial network (GAN)
-->




