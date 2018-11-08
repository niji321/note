title: kaggle quora 比赛总结
author: xgwang

categories:
  - 比赛相关
  
date: 2018-07-05 21:31:00
---

https://www.kaggle.com/c/quora-question-pairs

我在参加 cikm 2018 比赛时候主要参考了这个比赛的东西。现在总结如下

##### 不同的预处理形成多组数据
- raw data
- cleaned and not cleaned
- shared word removed or not removed
- lowercase and unchanged
- **punctuation replaced in different ways  ?**
- stop words included and excluded and stopwords only
- stemmed and not stemmed
- generate unigram bigram phrases of stemed courpus

##### 预处理流程
1. Lower case
- Text cleaning
- Stop words removal
- Word stemming (**porter stemmer?**)
- Punctuation Cleaning
- Shared words deletion
- Part of Speech Group

#### 特征
##### Embedding features
1. Word vector features
	1.  Max matching signals
	2.  Word move distance
	3.  noun phrases similarities
    
2. Sentence vector features
	1.	Weighted Sum of Word Embedding (TF-IDF)
    2.	Sum of Word Embedding
    3.	Various distance 

##### Statistic features
1.	Powerful words features
1.	Interrogative words features
1.	Edit and sequence matching distances, percentage of common tokens up to 1, 2, ..., 6 when question ends the same, or starts the same
2.	Number of capital letters, question marks etc... 这个简单的特征还真的没做


##### NLP features
1.  [WordNet similarity](https://github.com/sujitpal/nltk-examples/blob/master/src/semantic/short_sentence_similarity.py)
1.	 Word mover distances
1.	 NER-based features. I got the tags with Stanford CoreNLP
1。	 POS-based features. 
1. Similarity measures on LDA ,LSI, NMF embeddings.
2. Similarity measures on bag of character n-grams ( TFIDF reweighted or not) from 1 to 8 grams.
1. Key Phrase Extraction **SG Rank**
2. Dependency Parsing **Principal component extraction** **Semantic tree**
5. Brown-cluster

##### Graph features
1. Build Graph
	1.	Directed Graph / Undirected Graph
	2.	Co-occurrence Based Graph / Similarity Based Graph
2. Statistic of nodes and edges
	-	In-degree of the nodes
3. Structure information
	-	Component analysis
    -	Clique analysis
4. Propagation information
	-	PageRank / Hits
	-	Neighbor analysis
	-	Features propagation (shortest path)
5.	K-core scores
6.	frequency features

##### Magic features
1. Using the best available model, predict the test set.
2. Build a sparse square matrix of size len(train+test) x len(train+test)
3. For all pairs with duplicate probability higher than threshold (say 0.3), put a '1' in two corresponding cells of said matrix
4. For each pair from train and test, compute cosine similarity between corresponding rows.

###### 主要用到的深度模型有:

-	Siamese LSTM
-	Dec attention
-	ESIM
-	BiLSTM
-	LSTM
-	Recurrent Highway Networks

##### 给模型的特征描述有：
-	rank 1:	
	-	neural networks trained on both text sequences and our graph / text mining features proved to be our best single models.
	-	train siamese models on a **character level** to provide further diversity to our stacking, but it is hard to tell whether it was really helpful.


##### Stacking 模型有：

**The rank 1st team made a 4 layers stacking :**

- Layer 1 : Around 300 models, Paul and Lam’s neural nets, and classical algorithms like XGB, LGBM, which worked pretty well, and a lot of Scikit-learn classification algorithms (ET, RF, KNN, etc.)
- Layer 2 : Around 150 models using:
	- All the inputs features
	- Predictions of aAll the algorithms above
	- We added hidden layers of the best L1 pure text ESIM model
- Layer 3 : 2 Linear models
	- Ridge by perimeter (3 perimeters were created, based on min/max degrees) on 3 least Spearman correlated L2 predictions
	- Lasso with logit preprocessing of all L1 and L2 predictions
- Layer 4 : Blend
 	- 55/45, based on public LB score (final and best submission)
    

**The rank 2nd team solutions:**

Final solution was a simple weighted ensemble of 7 models -- 6 lightGBM and one NN. After producing the ensembled predictions, we had a post-processing phase where we recalibrated the probabilities based on some graphicial properties, similar to what Jared described.

Each of us independently created both graphical and NLP features, which we then shared. We had one LGB model that included all useful features (many thousands, including sparse ngram vectors). The others included different subsets of the features and used different LGB architectures. Our single superset model scored about 0.116-0.117 on the LB.

**The rank 5th team solutions:**

Our final model is an XGB with ~600 features, of these ~25 are oof-model predictions (LightGBMs, NNs/LSTMs and some SGDs) or likelihoods and the others are "normal" features. We used an oversampled training set with prior ~0.13 to train our submission models and we validated on the last n rows. Out-of-fold predictions have been created via 5 stratified folds on non-oversampled data.

**The rank 7th team solution:**
he used 8 dnn to catch features

-	Recurrent Highway Networks
-	BiLSTM character-level
-	BiLSTM character-trigram level
-	1-D convolutional net with decomposable attention
-	All above without attention mechanism

Minimal preprocessing was used, no stop words were removed because it harmed performance. The predictions of the networks (after sigmoid) were used as features for the XGB. To get out-of-fold predictions on the whole train set, 10-fold cv was used.

**The rank 8th team solution:**

We using the same 5 fold stacking
-	stack level 1 : lgb、xgb、mlp with dense feature、mlp with sparse tfidf weighted feature、et、rf and so on.
-	stack level 2 : we use lgb、xgb、mlp、rf and et, and our final submit was a simple avg of those models.
