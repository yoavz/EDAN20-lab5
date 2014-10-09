Lab 5: Dependency Parsing 
======================
Report by ***Yoav Zimmerman*** for EDAN20 Fall 2014

Parsing Functions
-----------------

In this lab, we developed machine learning models to help us in parsing sentences into dependency relations. As a framework, we used the parsing algorithm proposed by _Joakim Nivre_. This algorithm uses a stack (initialized to empty), queue (initialized to the list of words being parsed), and dependency list (initialized to empty). It defines four parsing actions:

* Left-Arc (`la`) - pop the top of the stack `n` and create a dependency `(n', n)` where `n'` is the front of the queue.
* Right-Arc (`ra`) - remove the front of the queue `n'`, push it into the stack, and create a dependency `(n, n')` where `n` is the previous top of the stack.
* Reduce (`re`) - remove the top of the stack
* Shift (`sh`) - remove the front of the queue and push it onto the stack

In this labratory we will use classification to develop a model for parsing actions. That is, given a certain parsing context, we will develop an algorithm to choose the most optimal parsing action. 

Parsing an Annotated Corpus
---------------------------

Given an annotated corpus with dependency relations, how do we choose the order in which to apply the parsing functions? First, we note that `la`, `ra`, and `re` parsing actions have constraints on them. `la` should only be performed if the top of the stack is the head of the front of the queue. Inversely, `ra` should only be performed if the the front of the queue is the head of the top of the stack. Finally, `re` can only be performed if it already has a dependency in the dependency list, and should not be performed unless there is some word further down the stack that provides an opportunity to `la`. 

A key observation is that, once you apply constraints, only one of the three constrained parsing actions is viable at each step. Therefore, all the program has to do is check which action is possible at each step and perform that action, without ambiguity. If one of the three constrained parsing actions cannot be performed, then the program defaults to the only other action possible, `sh`. 

A skeleton program was provided to students, with some basic relevant data structures defined. Also defined in `ReferenceParser.java` were functions which returned whether or not a parsing action was viable at each step: `oracleLeftArc()`, `oracleRightArc()`, and `oracleReduce()`. These allowed a full dependency graph and transition list to be built for the annotated training set, by simply iterating through all words and checking which parsing action was viable at each step. 

Extracting Features
-------------------

Since our goal was to develop a machine learning model, we needed to extract features from the parsing context at each step, along with the optimal action. Three different feature models were experimented with:

1. The POS tags of the first word on the stack and first word in the queue.
2. The POS tags of the first and second words on the stack and the first and second words in the queue.
3. The POS tags of the first, second, and third words on the stack and the first, second, and third words in the queue.

Each of these models also included two boolean features, encoding whether or not a reduction or a left arc was possible at each step. In addition, parsing actions could either be _labeled_, meaning their function was attached to the action, or _unlabeled_, meaning all actions would be treated equivalently regardless of the label. A _labeled_ parsing action looks like `"la.SS"`, where `"la"` is the parsing action and `"SS"` is the dependency function. For each model, a _labeled_ and _unlabeled_ version was generated, resulting in a total of 6 models, with 4, 6, and 8 features respectively.

Implementation and Results
--------------------------

To implement feature extraction, `ReferenceParser.java` was modified to keep track of the relevant context points at each step in parsing the annotated corpus. The data was then printed into ARFF format so that it could be loaded by Weka to generate the classification models. Here is how the same parsing step looked like in three different models:

```
Model 1 (unlabeled)
++	NN	true    false	la

Model 2 (labeled)
++	NN	NN	AV	true	false	la.++

Model 3 (unlabeled)
++	NN	ROOT	NN	AV	AV	true	false	la
```

In addition to generating the data points, ARFF headers had to be written for each of the six models. Once the ARFF files were complete, they were loaded into Weka GUI tool and the models generated using the **J48** algorithm. The following are the respective accuracies of the six models, calculated by Weka using cross-validation analysis with 10 folds.

| Method | Labeled? | Accuracy (%) |
| ------ | -------- | ------------ |
| 1      | No       | 86.23        |
| 1      | Yes      | 78.46        |
| 2      | No       | 91.54        |
| 2      | Yes      | 84.00        |
| 3      | No       | 91.56        |
| 3      | Yes      | 84.10        |

An interesting and nonintuitive result is that labeling the parsing actions actually leads to a decrease in accuracy. A more intuitive result is that the most accurate model is method 3 (unlabeled), the one with the most features. But the increase from six features (method 2) to eight features (method 3) only results in a very slight increase in accuracy, so the more complex model may not be worth the extra computation costs.

Next week, we will apply these models to parsing an unannotated corpus.
