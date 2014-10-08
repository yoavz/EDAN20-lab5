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

In this labratory we will use classification to develop a model for parsing actions. That is, given a certain parsing context, be able to choose which parsing action is the most correct to take.

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

In addition, each of these models were  

Extracting Subject-Verb-Object triples
--------------------------------------
