
---
layout: ../../layouts/Post.astro
title: "Bachlor thesis"
description: "Bachlor Thesis - Intrusion detection machine learning"
type: blog
tags: ["Python", "Machine Learning", "Threat Detection"]
backLink: /blog
backLabel: Back to blog
---


# Bachlors Thesis- machine learning threat detection

[Full Thesis Here](https://www.diva-portal.org/smash/get/diva2:1979453/FULLTEXT01.pdf)

Normal signature based threat detection is in the past, it comes with multiple cons:
### Cons
 - Take yesterdays attacks, signature based detections are based on known attributes
 - Doesn't see a pattern just an "event"
 - Not Scalable, When a new attack has been identified a rule must be setup to detect it, leading to an overwhelming amount of work considering the threat landscape today.

### pros with machine learning for threat detection
- Sees a pattern and not just an event.
- Scalable - a well-trained model can detect attacks that has not previously been seen if it has similarities with the training data.

# Dataset
The CICIDS2017dataset was chosen, since this dataset is used for classification purposes of malicious network traffic. CICIDS2017 was created at the University of
New Brunswick because they felt that there was a lack of updated datasets for
intrusion detection. The generated traffic came from a topology with routers,
switches, firewalls and operating systems such as windows and Mac OS X to
simulate a “complete network".

# Algorithms
All the algorithms were supervised meaning that the dataset contained both features and labels. The algorithms were chosen to have different attributes/features.
### Random Forest
 - Ensamble Method - consists of decision trees using bagging
 - HyperParameters tested:
	 - N-Estimators - number of trees:
		 - 2–200
	- Max_depth - how deep each tree can go:
		- 1, 5, 10, 15, 20, 25
	
 ### XGBoost
 - Ensable Method aswell but uses gradient boosting instead of bagging
 - HyperParameters tested:
	- N-Estimators:
		-  2-200
	- Max_depth - how deep each tree can go:
		- 1, 5, 10, 15, 20, 25
	- Learning Rate - how much each tree learns from previous trees mistake
		-	0.1, 0.5, 1

 - ### KNN (K-Nearest Neighbors)
-   Instance-based learning algorithm that predicts using nearby data points
-   Hyperparameters tested:
    -   N-neighbors — how many nearby data points to check when making a prediction:
           -   1–50            
    -   Metric — how the distance to nearby data points is calculated:        
        -   Manhattan, Minkowski            
    -   Algorithm — the method used to calculate the nearest data points:    
        -   Ball tree, Kd tree, Brute
    -   Weights — how valuable the neighboring data points are
        -   Uniform, Distance
   
   ### Linear SVC

-   Linear Support Vector Classification model used for binary/multiclass classification    
-   Hyperparameters tested:    
    -   Penalty — type of regularization used to prevent overfitting:        
        -   L1, L2
    -   C — controls margin size and strength of regularization:        
        -   0.0001, 0.001, 0.01, 0.1, 1, 10, 100
            

### Logistic Regression

-   Linear classification model that estimates class probabilities, uses sigmoid Function 
-   Hyperparameters tested:    
    -   C — controls the strength of regularization to prevent overfitting:
           -   0.0001, 0.001, 0.01, 0.1, 1, 10, 100           
    -   Solver — optimization algorithm used to fit the model:
           -   liblinear, lbfgs, saga, sag, newton-cg, newton-cholesky



# Methodology

![Methodolgy Steps](./images/BachlorMethodology.png "Methodolgy Steps")


# Results
### Why not Accuracy
Most previous work are using accuracy when determining which algorithm/model perfrmed the best, this works great when you have a balanced dataset but imbalanced it gives a false sense of performance. if you have 95% of normal traffic and 5% attack traffic you can have an accuracy of 95% but recall is 0% because you missed all of the minory class. Using accuracy with imbalanced datasets doesn't work mathematiclly 

### Precision
How many of the predictions were correct - given in percentage

$$
\text{Precision} = \frac{TP}{TP + FP}
$$

### Recall
How Many of the Attacks did we catch, how many of the True Positives - gives in percentage

$$
\text{Recall} = \frac{TP}{TP + FN}
$$

### F1-score
Mean of both precision and recall, better than accuracy when you are using an imbalanced dataset. Given score on 0-1 range
$$
F1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}
$$

![Results](./images/BachlorResults.png "Results")
