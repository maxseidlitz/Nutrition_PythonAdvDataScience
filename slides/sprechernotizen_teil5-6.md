# Speaking script, Part 5 & 6 (Max)

Covers the divider "Unsupervised Learning - Clustering" through the "Hyperparameter tuning" slide, nine slides in total, roughly 6 to 8 minutes.

---

## Slide: Divider "Unsupervised Learning - Clustering"

So far, NOVA and Nutri-Score have been the labels we're trying to predict. For this part I want to flip that around. Let's forget both labels for a moment and just ask: do these products naturally fall into groups based on their nutrients alone? That's what this section is about, clustering with KMeans. And it ties directly into one of our hypotheses, H4, which I'll get back to in a second.

---

## Slide: "Clustering on nutrient values (unsupervised)"

We ran KMeans on eight nutrient features, no labels involved at all. You'll notice that's eight and not the nine nutrients we used elsewhere. Salt and sodium measure almost the same thing, salt is sodium times 2.5, and the correlation between them is essentially 1. So we dropped sodium, otherwise that one dimension would have counted twice in the distance calculation. We also deliberately left out the additive count, the ingredient count, and the product category. Those belong more to NOVA than to Nutri-Score, and including them would have tilted the comparison we're about to make.

The clustering sample is 17,787 products, every product that has both a valid Nutri-Score grade and a valid NOVA group, so we can check the resulting clusters against both systems afterward.

To pick the number of clusters, we looked at two things: the elbow method, shown on the left, where the drop in inertia flattens out somewhere around six to eight clusters, and the silhouette score, on the right, which peaks at k equals 9 with a value of 0.3857. Both point in roughly the same direction, so we went with nine clusters. One thing worth saying clearly, the cluster numbers themselves don't mean anything. Cluster 5 isn't better or worse than cluster 1, they're just labels.

The actual result is the two numbers on the right of the slide. The Adjusted Rand Index between our clusters and Nutri-Score is 0.081. Between our clusters and NOVA it's 0.029. This index runs from 0, which is chance level, to 1, a perfect match, so both numbers are weak in absolute terms. But the gap between them is the point: clusters built purely from nutrients land noticeably closer to Nutri-Score than to NOVA. That confirms our hypothesis H4, though only weakly. It also makes sense, since the clusters are built entirely from nutrient values, and that's exactly what Nutri-Score is based on too. NOVA looks at processing and additives instead, none of which went into this clustering at all.

One more number on this slide worth mentioning: the underlying PCA visualization uses two components that together explain 54.7% of the variance in the data.

Neither label gets picked up well by nutrient clusters alone. And that's really the motivation for the next section. If we actually want to predict these labels well, we need supervised learning, using all the features we have available, not just nutrients.

---

## Slide: Divider "Machine Learning Models"

That brings us to the core of the project. Can we actually predict NOVA? We trained six different models, compared them against each other, and then tuned the two strongest ones.

---

## Slide: "Preprocessing of Data"

Before any model sees the data, the pipeline needs to be built correctly. Our feature matrix consists of the nine nutrient values, plus the additive count, the ingredient count, and the product category. The target is the NOVA group, one through four.

We split the data 80/20, stratified, which gives us 18,724 products for training and 4,682 for testing. Stratifying matters here because NOVA is quite imbalanced, and we want that same imbalance preserved in both the training and the test set.

Missing nutrient values are filled in using the median rather than the mean, since several of these distributions are skewed and the median is more robust to that. It also means we don't lose any rows by dropping them.

We apply a standard scaler to the numeric features. That matters most for the models that rely on distance or feature weighting, K-Nearest Neighbors, Logistic Regression, and the neural network. Without scaling, a feature like energy, which can run up to a thousand kilocalories, would completely dominate something small like salt.

The product category is one-hot encoded. If the test set contains a category the model never saw during training, that row simply gets zeros across all category columns instead of causing an error.

And the most important part of this slide: the entire pipeline, imputer, scaler, and encoder, is only ever fit on the training data. On the test set it's only applied, never refit. That's what keeps this leakage-safe, no information from the test set ever influences how the model is built.

---

## Slide: "Overview of all Models - The Winners"

We trained six models in total, and every single one clears 80% accuracy. Random Forest comes out on top.

We chose these six models to cover a real spread of approaches rather than six variations of the same idea: Logistic Regression as a linear baseline, K-Nearest Neighbors as an instance-based method, a single Decision Tree, two ensemble methods, Random Forest using bagging and Gradient Boosting using boosting, and a Multi-Layer Perceptron as our neural network. This also satisfies the project requirement of using at least five models, including one ensemble and one neural network.

Ranked by weighted F1 on the test set: Random Forest at 0.857, Gradient Boosting at 0.841, Decision Tree at 0.831, K-Nearest Neighbors at 0.816, the neural network at 0.815, and Logistic Regression at 0.808.

This ranking holds across all four metrics we report, accuracy, F1, precision, and recall. So Random Forest's lead isn't a fluke of one particular metric, it's consistently ahead.

We chose weighted F1 as our main metric rather than plain accuracy because NOVA is imbalanced, almost 65% of the data falls into group 4. Weighted F1 accounts for precision and recall together and isn't as easily flattered by a dominant majority class as accuracy can be.

---

## Slide: "Model comparison - NOVA main problem"

Here's why the ensemble methods win, not just that they do. Random Forest and Gradient Boosting handle correlated features gracefully. Earlier in our correlation analysis we saw fat and energy correlated at 0.82, and sugars and carbohydrates at 0.67. A tree-based model can simply split on whichever correlated feature works best at each step, it's not bothered by that redundancy. Logistic Regression, on the other hand, is the one model here that genuinely struggles with that kind of multicollinearity.

There's also the shape of the NOVA boundary itself. A product doesn't become "ultra-processed" because of one single threshold, it's several conditions crossed together, a lot of additives, a lot of ingredients, and a particular nutrient pattern all at once. That kind of interaction is exactly what tree-based and ensemble methods are built to capture, and it's something a linear model like Logistic Regression structurally cannot represent, no matter how it's tuned.

So our best model here is Random Forest, with an F1 score of 0.857 and an accuracy of 0.859. That's the model we look at more closely on the next two slides, and the one we tune later on.

---

## Slide: "Where does the best model fail?"

This confusion matrix shows exactly where Random Forest struggles, and it's almost entirely concentrated on the boundary between group 3 and group 4.

Most of the predictions sit on the diagonal, so overall the model performs well. Group 4, the ultra-processed foods, gets 2,827 out of 3,030 correct, a recall of 0.93. Given that it's the majority class you'd expect that, but it's good to see it actually holds up in practice. Group 2 is the more interesting result: 190 out of 224 correct, a recall of 0.85, despite being the smallest class at only 4.8% of the data. That's because group 2 has a very distinct nutrient profile, oils, butter, and sugar, which shows up as extreme fat and energy values, something we already saw in the nutrient profile heatmap earlier.

The real source of error is group 3 against group 4. 254 group-3 products get predicted as group 4, and 175 group-4 products get predicted as group 3. That single pair accounts for by far the largest share of all the misclassifications in this matrix. It makes sense why: moderately processed and ultra-processed foods overlap heavily in both nutrient content and ingredient counts, which our earlier EDA heatmap already hinted at. Conceptually these two groups are also the closest to each other, the line between them is gradual rather than sharp.

Errors between the extremes, group 1 against group 4, or group 1 against group 2, are rare. The model handles the clear-cut cases reliably, it's the middle ground where processing levels blur together that gives it trouble.

---

## Slide: "Feature importance - what drives NOVA prediction"

This slide shows what the Random Forest is actually relying on to make its predictions, and it's not the nutrients, it's the two count features. The ingredient count contributes 22.1% of the total importance, and the additive count contributes 19.4%. Together that's about 41.5%, far more than any single nutrient. Energy, the third-strongest feature, only accounts for 7.6%.

This directly confirms our hypothesis H1: more additives and more ingredients push a product toward NOVA group 4. It also lines up exactly with how NOVA is defined in the literature, the whole classification is built around the extent of processing and the complexity of additives and ingredients, not the nutrient content itself.

The category dummy variables barely register, each contributing under 2%, so product category adds very little on top of these two count features.

---

## Slide: "Hyperparameter tuning"

We ran GridSearchCV on our two strongest models, Random Forest and Gradient Boosting, using five-fold stratified cross-validation and optimizing for weighted F1.

For Random Forest, we searched over the number of trees, 200 or 400, the maximum depth, none, 10, or 20, and the minimum samples per leaf, 1, 3, or 5. The best combination during cross-validation was 400 trees, a depth of 20, and a leaf size of 1, giving a cross-validation F1 of 0.867. But on the actual test set, the F1 score moved from 0.857 to 0.856, essentially no change. That tells us the baseline configuration, 300 trees with unrestricted depth, was already close to optimal. Random Forest tends to be fairly insensitive to its hyperparameters once it has enough trees, and that's exactly what we see here.

Gradient Boosting responded differently. We searched over the number of estimators, 100 or 200, the learning rate, 0.05 or 0.1, and the maximum depth, 2, 3, or 4. The best configuration was 200 estimators, a learning rate of 0.1, and a depth of 4. On the test set, F1 improved from 0.841 to 0.849, a genuine gain of 0.8 points. Gradient Boosting is noticeably more sensitive to exactly these three settings than Random Forest is, so there was real room for tuning to help.

Which brings us to the key result of this slide: even after tuning, the model we keep as our final model is the untuned Random Forest, at F1 equals 0.857. It still outperforms both the tuned Gradient Boosting, at 0.849, and even the tuned Random Forest itself, at 0.856. The takeaway is that hyperparameter tuning doesn't automatically improve performance on unseen data, and the cross-validation score alone can't be fully trusted, the test set is what actually confirms whether an improvement holds.

So our final model is settled, the untuned Random Forest at F1 equals 0.857. Next, we'll look at how that compares to the Nutri-Score side problem, and what all of this means for our hypotheses overall.
